# 032.front-miplatform 구조안

## 1. 목적

이 문서는 `N:\99.SourceCode Backup\NPH`를 실제 유지보수 관점으로 분석할 때, `032.front-miplatform` 폴더를 어떻게 구성하면 좋은지 제안하는 문서다.

핵심 전제는 아래와 같다.

- 실제 유지보수는 Front/MiPlatform에서 시작하는 경우가 많다.
- 따라서 `032`는 단순 아키텍처 설명이 아니라, 유지보수 착수용 폴더여야 한다.

## 2. 추천 하위 구조

```text
032.front-miplatform/
├─ 0320.index/
├─ 0321.screen-structure/
├─ 0322.transaction-mhi/
├─ 0323.navigation-command/
├─ 0324.dataset-converter/
├─ 0325.ui-event-pattern/
├─ 0326.jsp-browser-bridge/
├─ 0327.fact-check/
├─ 0328.todo/
└─ 0329.archive/
```

## 3. 하위 폴더 역할

### 0320.index
- README
- 문서맵
- 용어집
- 읽는 순서

### 0321.screen-structure
- MiPlatform XML 화면 구조
- dataset 배치
- 탭/그리드/영역 구획
- 대형 화면 구조 요약

### 0322.transaction-mhi
- 화면에서 호출하는 Transaction 목록
- `.mhi` URL 정리
- 입력/출력 Dataset 패턴
- 화면별 API 표

### 0323.navigation-command
- `.mhi -> navigation -> action -> command`
- 대표 command 매핑
- command 진입점 기준 추적

### 0324.dataset-converter
- MiplatformRequest
- MiplatformConverter
- Dataset / LData / LMultiData
- `_CUD`, row status, converter 관점 정리

### 0325.ui-event-pattern
- OnLoad
- 버튼 클릭
- 탭 전환
- 그리드 클릭
- 자동조회/자동복사 같은 UI 이벤트 패턴

### 0326.jsp-browser-bridge
- JSP 진입
- 브라우저/스크립트 연계
- ActiveX/OCX/외부 뷰어 브리지
- MiPlatform 외부 화면 연결

### 0327.fact-check
- 화면/XML/transaction 관련 확정/미확인 사실 분리

### 0328.todo
- 아직 닫히지 않은 front 관련 후속 조사

### 0329.archive
- 초기 조사본, 재구성 전 문서 보존

## 4. 추천 문서 유형

### 유지보수 시작 문서
- `대형 화면별 Transaction 맵`
- `버튼/탭 -> 함수 -> mhi -> command 표`
- `화면 OnLoad 흐름`

### 구조 이해 문서
- `Dataset 구조 설명`
- `화면 XML 공통 패턴`
- `Navigation 연결 규칙`

### 검증 문서
- `fact-check`
- `todo`

## 5. 우선 작성 대상

1. `0320.index/01.README.md`
2. `0322.transaction-mhi/01.화면별-mhi-맵.md`
3. `0323.navigation-command/01.mhi-navigation-command-규칙.md`
4. `0325.ui-event-pattern/01.대형화면-이벤트-패턴.md`
5. `0324.dataset-converter/01.Dataset-LData-LMultiData.md`

## 6. 대표 수용 문서 예시

- `MD_ORD01001P`의 UI 이벤트/Transaction 관점 문서
  - `0325.ui-event-pattern`
  - `0322.transaction-mhi`
- `ptmdcrNavi`, `clamNavi`, `drgNavi` 같은 navigation 설명
  - `0323.navigation-command`
- `MiplatformConverter`, Dataset 변환 설명
  - `0324.dataset-converter`

## 7. 판단

`032.front-miplatform`은 단순 보조 폴더가 아니라, 실제 유지보수자가 가장 먼저 보는 폴더가 되어야 한다.

이유는 간단하다.

- 장애/변경 요청은 화면에서 시작된다.
- `.mhi`, Dataset, command 매핑이 틀리면 DB까지 내려가기 전에 이미 문제가 생긴다.
- 따라서 `032`를 앞세우는 것이 실제 작업 흐름과 가장 잘 맞는다.
