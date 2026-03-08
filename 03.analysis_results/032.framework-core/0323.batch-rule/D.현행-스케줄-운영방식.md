# 현행 스케줄 운영방식

## 1. 목적

이 문서는 현행 사이트 베이스 설정 기준으로 NPH 배치 스케줄이 실제로 어떻게 관리되고 실행되는지 정리한다. 핵심은 `cron 식이 어디에 있느냐`보다 `운영자가 어디서 JobGroup/Job을 관리하고, 시스템이 무엇을 실행하는가`를 확인하는 것이다.

약어/용어는 [../../030.index/0303.약어-용어집/약어-용어집.md](../../030.index/0303.%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91/%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91.md)를 먼저 보면 빠르다.

## 2. 핵심 결론

- `devon-batch-scheduler.xml`은 운영 JobGroup/Job의 개별 스케줄을 직접 나열하지 않는다.
- 현행 백업셋 기준 스케줄 운영은 `batchMgr` 관리 화면 + DB 메타데이터 + `BatchExecutor.cmd` 실행 조합으로 보는 것이 맞다.
- 자동 상주 스케줄러는 `scheduler-enable=false`라서 꺼져 있을 가능성이 높다.
- 그러나 온라인/수동 실행 경로는 살아 있다. 실제 로그에도 `BatchExecutor.cmd -groupId ... -override ...` 호출 흔적이 남아 있다.

## 3. 운영 화면 기준 관리 구조

확인 파일:
- `NPH_HIS/devonhome/navigation/batch/navigation.xml`

확인된 액션 범위:
- Scheduler 설정 화면
- JobGroup 목록/단건 조회/생성/수정/삭제
- Job 목록/단건 조회/생성/수정/삭제
- Job 실행 로그 조회/삭제
- JobGroup 실행 지시 / 중지 지시
- JobTarget / JobSupportClass 관리

해석:
- 이 시스템은 단순 cron 설정 파일이 아니라, 배치 관리자 화면에서 JobGroup/Job 메타데이터를 다루는 구조다.
- 따라서 운영 스케줄 정의의 실체는 XML보다는 DB와 관리자 UI에 더 가깝다.

## 4. 실행기 기준 운영 구조

### 4.1 설정

확인 파일:
- `his.xml`
- `BatchExecutor.cmd`

확인된 값:
- `online-batch-executor/command = C:\AADEV_NPH\workspace\NPH_HIS\cmd\BatchExecutor.cmd`
- `BatchExecutor.cmd` main class = `devon.batch.core.shell.JobGroupExecutor`

### 4.2 실행 요청 코드

확인 파일:
- `BatchOnlineExecuteCMD.java`
- `BatchInfoPC.java`
- `BatchInfoUC.java`

확인된 동작:
- `groupId` 확인
- `override` 문자열 조립
- `BatchExecutor.cmd -groupId ... -override ...` 실행

```mermaid
flowchart LR
    UIorCMD --> BatchOnlineExecuteCMD
    BatchOnlineExecuteCMD --> BatchInfoPC
    BatchInfoPC --> BatchInfoUC
    BatchInfoUC --> BatchExecutorCmd
    BatchExecutorCmd --> JobGroupExecutor
```

해석:
- 현재 사이트 운영에서 배치 실행은 UI/Command가 JobGroup ID를 넘기고, 외부 실행기로 JobGroupExecutor를 호출하는 형태다.

## 5. 실제 운영 흔적

확인 로그:
- `devonhome/logs/20260202AMdebug.log`
- `devonhome/logs/batch-file-log/devon-batch-syslog`
- `devonhome/logs/batch-file-log/batchLog`
- `devonhome/logs/batch-file-log/2026-02-02/HP_BAT01206B-86.log`
- `devonhome/logs/batch-file-log/2026-02-02/HP_BAT01206B-86-HP_BAT01206B01.log`

직접 확인된 예:
- `작업그룹 HP_BAT01206B를 [동기]로 실행합니다.`
- `작업 [HP_BAT01206B01] 설정이 override될 예정입니다.`
- `batchLog`에 `초기화 -> 처리 중 -> 리더 생성 -> 파서 생성 -> 리더-파서 검증 -> 작업 생성 -> 작업 실행 -> 작업 종료 -> 작업 정리` 상태 전이
- `devon.batch.core.exception.exec.JobGroupExecutionException`
- `작업 [HP_BAT01206B01] devon.batch.core.exception.exec.BatchStackedException`
- 개별 로그 파일명 `HP_BAT01206B-86`, `HP_BAT01206B-86-HP_BAT01206B01`

해석:
- 로그 기준으로는 JobGroup/Job 구조가 실제 운영에서 사용되고 있다.
- 그리고 실행 흐름은 `JobGroup -> Job -> Reader -> Parser -> Job class` 순서로 남는다.
- 다만 현재 로그만으로는 `HP_BAT01206B01`이 어떤 Java Job 클래스로 연결되는지는 닫히지 않는다.

## 6. 자동 스케줄러와 수동/온라인 실행의 분리

현재 동시에 보이는 것:
- `scheduler-enable=false`
- embedded executor enabled
- JobGroup 관리 화면 존재
- BatchExecutor 기반 수동/온라인 실행 로그 존재

가장 안전한 해석:
- 상주 자동 스케줄러는 비활성일 가능성이 높다.
- 그러나 JobGroup/Job 메타데이터와 실행기는 살아 있고, 운영자는 UI 또는 서비스 호출을 통해 개별 JobGroup을 실행할 수 있다.
- 즉 `자동 예약 실행`과 `수동/온라인 실행`은 분리해서 이해해야 한다.

## 7. 확인된 한계

현재 백업셋만으로는 아래를 확정할 수 없다.
- 실제 DB에 저장된 전체 JobGroup 목록
- 개별 Job의 cron 식 또는 예약 주기
- 현재 운영 중인 활성 스케줄 전체
- `HP_BAT01206B01`의 최종 Java Job 클래스 매핑

즉 현행 설정의 틀과 실행 방식은 확인됐지만, `전체 예약표` 자체는 DB 없이는 닫히지 않는다.

## 8. 리뷰

이 문서가 필요한 이유는, 기존 `batch-rule` 문서만 보면 `배치 = Rule 엔진`처럼 읽히기 쉽기 때문이다.

실제로는:
- DevOn Batch 컨테이너
- 스케줄러 관리자 UI
- DB 메타데이터
- BatchExecutor.cmd
- JobGroup/Job 실행 로그

이 먼저 있고,
Rule 엔진은 그 뒤에 일부 Job/UC에서 붙는다.

## 9. 다음에 읽을 문서

- [A.DevOn-Batch-컨테이너-개요.md](./A.DevOn-Batch-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EA%B0%9C%EC%9A%94.md)
- [B.DevOn-Batch-스케줄러-파이프라인.md](./B.DevOn-Batch-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%9F%AC-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8.md)
- [C.Rule-Engine-연동지점.md](./C.Rule-Engine-%EC%97%B0%EB%8F%99%EC%A7%80%EC%A0%90.md)
