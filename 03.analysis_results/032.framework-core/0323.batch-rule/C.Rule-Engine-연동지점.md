# Rule-Engine 연동지점

## 1. 목적

이 문서는 DevOn Batch 컨테이너가 외부 Rule 엔진과 만나는 접점을 정리한다. 여기서 중심은 InnoRules 엔진 자체가 아니라, 배치/UC 흐름이 외부 Rule 호출을 어디에서 붙이는가이다.

약어/용어는 [../../030.index/0303.약어-용어집/약어-용어집.md](../../030.index/0303.%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91/%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91.md)를 먼저 보면 빠르다.

## 2. 핵심 결론

- 현재 확인된 InnoRules 연동은 `배치 컨테이너 자체`가 아니라 `배치 Job 또는 UC`에서 외부 Rule 호출을 붙이는 방식이다.
- `OtptClamCrtnJob.java`에는 `ModuleInitializer.initialize("innorules")`와 `destroy()` 흔적이 있지만 주석 처리돼 있다.
- 실제 Rule 실행은 `ClamCrtnRuleUC -> ClamChkRuleUC`처럼 UC 계층을 통해 내려가는 경로가 더 직접적으로 확인된다.
- 따라서 DevOn Batch는 컨테이너를 제공하고, InnoRules는 그 컨테이너 또는 UC 흐름에 결합되는 외부 솔루션으로 보는 것이 맞다.

## 3. 실제 접점

### 3.1 배치 Job 레벨 접점

확인 파일:
- `nph/bat/hp/job/OtptClamCrtnJob.java`

확인된 흔적:
- `ModuleInitializer.initialize("innorules")` 주석 처리
- `ModuleInitializer.destroy()` 주석 처리
- `ClamCrtnRuleUC.ruleDms(data)` 호출 흔적
- 과거 Rule 코드(`#S00000311`, `#S00000607`) 주석 흔적

해석:
- 한때 배치 Job이 직접 InnoRules 모듈 초기화를 다뤘거나 적어도 그 흔적이 남아 있다.
- 하지만 현재 백업 기준으로는 활성 호출이 아니라 주석 처리 상태다.

### 3.2 UC 레벨 접점

확인 파일:
- `nph/his/hp/com/uc/ClamCrtnRuleUC.java`
- `nph/his/hp/com/uc/ClamChkRuleUC.java`

확인된 흐름:
- `ClamCrtnRuleUC.ruleDms()`
- 공통코드 `MD994`로 Rule 적용 여부 확인
- `ClamChkRuleUC.chkClamRule()` 위임
- `ClamChkRuleUC` 안에서 `RuleReq`, `RulesInterface`, `ConnectionProperties.getConnection()`, `intf.execute(req, iRuleCodeType)` 호출

```mermaid
flowchart LR
    Job --> RuleUC
    RuleUC --> CheckUC
    CheckUC --> RuleAPI
    RuleAPI --> InnoRules
```

해석:
- Rule 실행은 배치 컨테이너 primitive가 아니라, 애플리케이션 UC가 외부 Rule API를 호출하는 형태다.
- 이 점이 중요하다. `0323`에서 엔진 자체 분석을 빼야 하는 이유이기도 하다.

## 4. 온라인 실행 경로와의 관계

온라인 실행 경로:
- `BatchOnlineExecuteCMD`
- `BatchInfoPC.executeOnlineBatch(...)`
- `BatchInfoUC.executeOnlineBatch(...)`
- `BatchExecutor.cmd -groupId ... -override ...`

이 경로는 JobGroup 실행 자체를 담당한다.

Rule 엔진 연동은 이 다음 단계에서,
- 실제 Job class
- 또는 Job이 호출하는 UC
에서 발생한다.

즉 온라인 배치 실행과 Rule 호출은 같은 층이 아니라:
- 실행기 호출
- Job 실행
- UC 위임
- Rule API 호출

순서로 분리된다.

## 5. 무엇이 032에 남고, 무엇이 033으로 가는가

### 5.1 032에 남는 것

- DevOn Batch 컨테이너 설정
- Scheduler / Reader / Parser / Executor
- Job이 외부 Rule 엔진을 호출하는 접점
- Job/UC 체인에서 Rule이 끼어드는 위치

### 5.2 033으로 가는 것

- InnoRules 엔진 아키텍처
- InnoRules Java API 구조
- InnoRules Rule 저장소/IRL
- InnoRules 초기화 모듈 세부

## 6. 리뷰

현재 `0323.batch-rule`을 그대로 두면 `배치 실행 틀`과 `InnoRules 솔루션 분석`이 섞여 읽히게 된다.

이 문서 기준으로 경계를 다시 잡으면:
- DevOn 쪽 질문: `배치가 외부 Rule을 어디서 호출하나?`
- InnoRules 쪽 질문: `Rule 엔진 자체는 어떻게 동작하나?`

로 나뉜다.

이 구분이 실제 유지보수에서도 더 유용하다.

## 7. 다음에 읽을 문서

- [D.현행-스케줄-운영방식.md](./D.%ED%98%84%ED%96%89-%EC%8A%A4%EC%BC%80%EC%A4%84-%EC%9A%B4%EC%98%81%EB%B0%A9%EC%8B%9D.md)
- [../../033.platform-services/0334.InnoRules/README.md](../../033.platform-services/0334.InnoRules/README.md)
- [../../033.platform-services/0334.InnoRules/C.InnoRules-Batch-연동.md](../../033.platform-services/0334.InnoRules/C.InnoRules-Batch-%EC%97%B0%EB%8F%99.md)
- [../../033.platform-services/0334.InnoRules/B.InnoRules-Java-연동.md](../../033.platform-services/0334.InnoRules/B.InnoRules-Java-%EC%97%B0%EB%8F%99.md)
