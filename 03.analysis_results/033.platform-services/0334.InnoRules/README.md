# README

## 1. 목적

이 폴더는 DevOn 자체가 아닌 외부 의료 Rule 솔루션인 InnoRules를 정리하는 기준 폴더다.

약어/용어는 [../../030.index/0303.약어-용어집/약어-용어집.md](../../030.index/0303.%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91/%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91.md)를 먼저 보면 빠르다.

## 2. 왜 033에 두는가

- InnoRules는 DevOn 프레임워크 자체가 아니다.
- `ruleEngine` 설정, `com.innoexpert.*` 패키지, `RulesInterface`, `RuleReq` 중심의 실행 표면, 그리고 `IRClient`/`IRSession` 관련 초기화·예시 흔적은 외부 솔루션/패키지 축에 속한다.
- 따라서 기준 문서는 `032.framework-core`보다 `033.platform-services`에 두는 것이 더 정확하다.

## 2A. 공개 자료를 어떻게 읽을 것인가

- 공개 웹 자료는 InnoRules 제품이 어떤 종류의 BRMS인지, RuleBuilder/DDM/시뮬레이션/저작 환경을 어떻게 설명하는지 파악하는 데 유용하다.
- 반면 NPH 실코드 분석의 주 근거는 여전히 로컬 소스/설정/JAR이다.
- 따라서 이 폴더 문서는 `제품 관점`과 `NPH 실사용 관점`을 분리해서 읽는 것을 원칙으로 한다.
- 현재 NPH에서 직접 확인되는 운영 표면은 `RulesInterface`, `RuleReq` 쪽이 강하다. 반면 `IRClient`, `IRSession`은 현재 JAR 스캔 범위와 업무 소스 기준 class 실체 및 직접 사용이 모두 약하거나 미확인이다.

## 3. 이 폴더의 문서

- [A.InnoRules-아키텍처-개요.md](./A.InnoRules-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EA%B0%9C%EC%9A%94.md)
  - `ruleEngine` 설정 파일, 저장소, JNDI, 초기화 모듈 구조
- [B.InnoRules-Java-연동.md](./B.InnoRules-Java-%EC%97%B0%EB%8F%99.md)
  - 실제 UC/코드에서 확인된 Java API 사용
- [D.InnoRules-Rule-정의-IRL.md](./D.InnoRules-Rule-%EC%A0%95%EC%9D%98-IRL.md)
  - Rule 코드 체계, 저장소, IRL 추정 구조
- [C.InnoRules-Batch-연동.md](./C.InnoRules-Batch-%EC%97%B0%EB%8F%99.md)
  - 배치/Job과 InnoRules의 결합 방식

## 4. 다른 폴더와의 경계

- DevOn Batch 컨테이너 자체는 [../../032.framework-core/0323.batch-rule](../../032.framework-core/0323.batch-rule)
- 의료업무/심사/청구의 도메인 의미는 별도 업무 문서가 준비되면 분리 정리

## 5. 읽는 순서

1. [A.InnoRules-아키텍처-개요.md](./A.InnoRules-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EA%B0%9C%EC%9A%94.md)
2. [B.InnoRules-Java-연동.md](./B.InnoRules-Java-%EC%97%B0%EB%8F%99.md)
3. [C.InnoRules-Batch-연동.md](./C.InnoRules-Batch-%EC%97%B0%EB%8F%99.md)
4. [D.InnoRules-Rule-정의-IRL.md](./D.InnoRules-Rule-%EC%A0%95%EC%9D%98-IRL.md)







