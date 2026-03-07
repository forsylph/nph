# DevOn Batch 스케줄러 파이프라인

## 1. 목적

이 문서는 `devon-batch-scheduler.xml`을 기준으로 NPH 배치 스케줄러가 어떤 실행 위치, Reader, Parser, 파라미터 처리 구성을 갖는지 정리한다. 여기서 중요한 점은 이 XML이 `운영 cron 목록`을 담는 파일이 아니라 `스케줄러가 관리할 수 있는 실행 틀`을 담는 파일이라는 점이다.

약어/용어는 [../../030.index/0303.약어-용어집/약어-용어집.md](../../030.index/0303.%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91/%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91.md)를 먼저 보면 빠르다.

## 2. 핵심 결론

- `devon-batch-scheduler.xml`은 배치 스케줄러의 실행 위치와 파라미터 처리 파이프라인을 정의한다.
- 현재 확인된 기본 실행 위치는 `embedded` executor 하나다.
- Reader/Parser 구성이 꽤 풍부해서 파일·리소스·JDBC·JSON·CSV·고정길이 등 다양한 입력 형식을 수용한다.
- 반면 운영 Job별 cron 식이나 스케줄 목록은 이 XML에 직접 박혀 있지 않다.
- 즉 현행 스케줄은 XML 하드코딩보다 스케줄러 UI/DB 관리형 구조로 보는 것이 맞다.

## 3. 스케줄러 설정의 중심축

### 3.1 실행 위치

확인된 설정:
- `scheduler-enable=false`
- `executor/location-list/embedded/id=SCHEDULER-EMBEDDED`
- `enabled=true`

해석:
- 상주 자동 스케줄러 기능은 꺼져 있을 가능성이 높다.
- 하지만 executor 목록에는 embedded 실행자가 남아 있다.
- 즉 자동 스케줄러와 개별 JobGroup 실행은 분리해서 봐야 한다.

### 3.2 스케줄러 DB spec

확인된 설정:
- `database/devon-spec-name=default`
- `timestamp-format=yyyy-MM-dd HH:mm:ss.SSSSSS`
- `schedule-date-format=yyyy-MM-dd hh:mm:ss`

해석:
- 스케줄러가 관리하는 JobGroup/Job/실행로그 메타데이터는 DevOn spec `default`를 사용한다.
- 이 점이 XML 하드코딩이 아니라 DB 관리형이라는 해석과 잘 맞는다.

## 4. 파라미터 파이프라인

```mermaid
flowchart LR
    input --> reader
    reader --> parser
    parser --> job
```

### 4.1 Reader 계층

실제 설정에서 보이는 대표 Reader:
- `DirectStringReader`
- `FileParameterReader`
- `ResourceFileParameterReader`
- `ResourceDirFileParameterReader`
- `JdbcCursorReader`
- `RelativeFileParameterReader`
- `EmptyReader`

해석:
- 배치 입력은 단일 형태가 아니다.
- 파일, 리소스, DB 커서, 직접 입력까지 흡수한다.
- 특히 `JdbcCursorReader`가 들어 있다는 점은 배치가 DB를 직접 입력 소스로 삼는 공식 경로를 갖고 있다는 의미다.

### 4.2 Parser 계층

실제 설정에서 보이는 대표 Parser:
- `JSON2LDataParser`
- `CSV2LDataParser`
- `LData2CSVParser`
- `FixedLength2LDataParser`
- `LData2FixedLengthParser`

해석:
- DevOn Batch는 단순 스케줄러가 아니라 데이터 형태 변환 파이프라인까지 제공한다.
- 따라서 Job은 `입력 형식`과 `실행 로직`을 분리해서 설계된다.

### 4.3 Job class registry

`devon-batch-scheduler.xml`에 등록된 `<job class="...">` 목록은 존재하지만, 현재 확인된 대부분은:
- `nph.bat.sample.*`
- 일부 `nph.bat.hp.job.*`

성격이다.

해석:
- 이 XML의 job 목록은 현행 운영 스케줄 전체 목록이라기보다, 스케줄러 UI가 선택 가능한 job class registry에 가깝다.
- 실제 사이트 운영 JobGroup/Job은 DB와 UI 관리 화면을 통해 묶이는 구조로 읽는 것이 더 안전하다.

## 5. 운영 스케줄을 XML에서 직접 못 찾는 이유

현행 백업 기준으로는 아래가 동시에 보인다.
- `scheduler-enable=false`
- `navigation/batch/navigation.xml`에 JobGroup/Job 생성/조회/수정/삭제 UI 액션 다수
- `BatchInfoUC`가 `BatchExecutor.cmd -groupId ... -override ...` 실행
- 실제 로그에 `HP_BAT01206B`, `HP_BAT01206B01` 같은 그룹/잡 식별자 남음

해석:
- 운영 스케줄 정의는 `devon-batch-scheduler.xml`의 `<job class>` 목록과 별개로, DB에 저장된 JobGroup/Job 메타데이터를 통해 관리되는 것으로 보는 것이 맞다.
- 즉 XML은 틀이고, 운영 스케줄은 DB/관리화면이 실체다.

## 6. 이 문서와 InnoRules 문서의 경계

### 6.1 여기서 설명하는 것

- 배치 스케줄러 설정
- 입력 소스
- 파싱 변환
- embedded executor
- batch spec / batch logging
- JobGroup/Job의 관리형 운영 방식

### 6.2 여기서 설명하지 않는 것

- InnoRules 저장소 `rule`
- IRClient / IRSession API 세부
- Rule 코드/Rule 정의 방식
- Rule 엔진 초기화 순서 자체

위 내용은 [../../033.platform-services/0334.InnoRules](../../033.platform-services/0334.InnoRules)에서 다룬다.

## 7. 리뷰

이 문서의 핵심은 `cron 식이 어디 있나`보다 `왜 XML에서 안 보이는가`를 설명하는 데 있다.

현재 설정 기준으로 보면:
- XML은 스케줄러 엔진 설정
- DB는 JobGroup/Job 메타데이터 저장소
- UI는 관리 화면
- `BatchExecutor.cmd`는 실제 실행기

로 역할이 분리되어 있다.

## 8. 다음에 읽을 문서

- [D.현행-스케줄-운영방식.md](./D.%ED%98%84%ED%96%89-%EC%8A%A4%EC%BC%80%EC%A4%84-%EC%9A%B4%EC%98%81%EB%B0%A9%EC%8B%9D.md)
- [C.Rule-Engine-연동지점.md](./C.Rule-Engine-%EC%97%B0%EB%8F%99%EC%A7%80%EC%A0%90.md)
- [../../033.platform-services/0334.InnoRules/C.InnoRules-Batch-연동.md](../../033.platform-services/0334.InnoRules/C.InnoRules-Batch-%EC%97%B0%EB%8F%99.md)
