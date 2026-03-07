# README

## 1. 목적

이 폴더는 NPH의 DevOn Batch 실행 틀을 현행 설정 기준으로 설명하는 기준 폴더다. 여기서 중심은 `배치를 자동/수동으로 어떻게 실행하는가`, `스케줄 정의가 어디에 있는가`, `JobGroup/Job이 어떤 컨테이너를 타는가`이다.

약어/용어는 [../../030.index/0303.약어-용어집/약어-용어집.md](../../030.index/0303.%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91/%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91.md)를 먼저 보면 빠르다.

## 2. 핵심 결론

- 현행 백업셋 기준 `devon-batch-scheduler.xml` 안에는 운영 Job의 개별 cron 식이 하드코딩돼 있지 않다.
- 대신 스케줄러 관리 화면과 DB 관리형 JobGroup/Job 구조가 존재한다.
- 실제 실행 진입점은 `BatchExecutor.cmd`이며, 온라인 실행은 `BatchOnlineExecuteCMD -> BatchInfoPC -> BatchInfoUC`를 통해 `-groupId`, `-override` 인자로 이 실행기를 호출한다.
- 즉 현재 사이트 베이스에서 봐야 할 핵심은 `배치 컨테이너`, `스케줄러 UI/DB`, `BatchExecutor.cmd`, `실행 로그`의 조합이다.

## 3. 이 폴더에 남기는 것

- DevOn Batch 컨테이너 설정
- DevOn Batch 스케줄러/파이프라인 설정
- JobGroup/Job 스케줄러 운영 방식
- 온라인 배치 실행 진입점
- Batch Job이 외부 Rule 엔진을 호출하는 접점

## 4. 이 폴더에서 빼는 것

- InnoRules 엔진 자체 아키텍처
- InnoRules Java API 상세
- InnoRules Rule 저장소/IRL 분석
- InnoRules 솔루션 기준 문서

위 항목은 [../../033.platform-services/0334.InnoRules/README.md](../../033.platform-services/0334.InnoRules/README.md)로 이동했다.

## 5. 읽는 순서

1. [A.DevOn-Batch-컨테이너-개요.md](./A.DevOn-Batch-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EA%B0%9C%EC%9A%94.md)
2. [B.DevOn-Batch-스케줄러-파이프라인.md](./B.DevOn-Batch-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%9F%AC-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8.md)
3. [D.현행-스케줄-운영방식.md](./D.%ED%98%84%ED%96%89-%EC%8A%A4%EC%BC%80%EC%A4%84-%EC%9A%B4%EC%98%81%EB%B0%A9%EC%8B%9D.md)
4. [C.Rule-Engine-연동지점.md](./C.Rule-Engine-%EC%97%B0%EB%8F%99%EC%A7%80%EC%A0%90.md)
