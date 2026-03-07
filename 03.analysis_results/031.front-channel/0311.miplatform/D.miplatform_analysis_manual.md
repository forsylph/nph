# NPH MiPlatform 분석 매뉴얼

이 문서는 `031.front-channel` 기준본을 보조하는 **reference 문서**다. 목표는 Tobesoft MiPlatform 3.3 공식 문서와 NPH 실제 소스를 함께 놓고, 화면 구성과 추적 방법을 차분히 설명하는 것이다.

- 기준본 문서:
  - [A.Miplatform.md](./A.Miplatform.md)
  - [B.MiPlatform-Transaction-패턴.md](./B.MiPlatform-Transaction-%ED%8C%A8%ED%84%B4.md)
  - [C.Dataset-입출력.md](./C.Dataset-%EC%9E%85%EC%B6%9C%EB%A0%A5.md)
- 이 문서의 역할:
  - 개념 정리
  - 공식 매뉴얼 관점 보조
  - 화면 XML 읽는 법 정리
  - NPH 실제 파일 위치와 연결

## 1. MiPlatform을 NPH에서 어떻게 볼 것인가

MiPlatform은 NPH에서 단순 화면 툴이 아니다. 실제로는 아래 네 가지를 같이 보는 것이 맞다.

1. 화면 XML
2. 화면 스크립트
3. Dataset
4. `.mhi` transaction 호출

즉 NPH에서 MiPlatform은 `화면 구성 기술`이면서 동시에 `서버 진입 채널`이다.

## 2. 공식 매뉴얼 기준 핵심 개념

Tobesoft MiPlatform 3.3 매뉴얼 기준으로 화면(Form)은 크게 세 층으로 읽을 수 있다.

### 2.1 Design
- Form 속성
- Div, Tab, Grid, Button, Edit 같은 Component 배치
- 화면 레이아웃 자체

### 2.2 Data
- Dataset 정의
- Dataset 컬럼 구조
- Component와 Dataset binding
- 입력/출력 데이터 경계

### 2.3 Event
- `OnLoadCompleted`, `OnClick`, `OnChange`
- Script 함수
- `Transaction()` 호출

NPH에 적용하면, 화면 XML 하나를 열었을 때 이 세 층이 종종 한 파일 안에 같이 들어 있다. 그래서 대형 화면도 다음 순서로 보면 읽기가 빨라진다.

1. Form 구조
2. Dataset 구조
3. Event 함수
4. `sSvcID`, `sSvcURL`, `Transaction()`

## 3. 공식 매뉴얼에서 NPH 분석에 실제로 도움 되는 항목

### 3.1 Include Event Script
공식 매뉴얼은 Form 생성 시 다음 두 패턴을 구분한다.
- `Include Event Script`
- `Component & Dataset, Image Only`

NPH 대표 화면은 첫 번째 패턴에 가깝다.
- `Login3.xml`
- `MD_ORD01001P.xml`
- `HP_DMS01303M.xml`
- `HP_DMS02204M.xml`

즉 화면 XML 하나에서 `구성 + Dataset + 이벤트 + 스크립트 + .mhi 호출`을 같이 읽을 수 있다.

### 3.2 Dataset은 Table 형태의 기억장소
공식 매뉴얼은 Dataset을 table 형태의 데이터 저장 구조로 설명한다. 또한 bind와 service object 연결을 강조한다.

NPH에선 이 관점이 그대로 적용된다.
- `sInputDs`
  - 화면에서 서버로 보내는 Dataset 묶음
- `sOutputDs`
  - 서버에서 화면으로 다시 받는 Dataset 묶음
- Grid/Combo/Edit
  - Dataset과 bind되어 바로 갱신되는 경우가 많다.

### 3.3 Service Object / Transaction
공식 매뉴얼에서는 Dataset과 서비스 호출을 결합해 설명한다. NPH에서는 이 부분이 실제로 `.mhi` 호출로 구현된다.

즉 실무에서는 아래가 핵심이다.
- 어떤 이벤트가 호출되나
- 어떤 함수가 실행되나
- 어떤 `sSvcURL`이 잡히나
- 어떤 `.mhi`로 내려가나

## 4. NPH 실제 시작 구조

### 4.1 index330.jsp
- 브라우저 진입점
- 로그인 상태에 따라 런처 URL 생성

### 4.2 NPH_start.xml
- `SessionURL="com::Login3.xml"`
- `protocol Compress="True"`, `XmlFormat="False"`
- AppGroup 다수 선언
- MiPlatform 세션 및 업무군 매핑의 출발점

### 4.3 Login3.xml
- 가장 단순한 MiPlatform transaction 예시
- `Transaction("CheckUserInfo", "NPHSE::/az/bizcom/authNavi/CheckLoginUser-new1.mhi", ...)`
- `RetirevePrivCodeList.mhi` 호출도 직접 보임

## 5. NPH 화면 XML을 읽는 실제 절차

### 5.1 첫 단계: 화면 정체 파악
- Form ID
- Title
- 주요 Grid/Tab/Div
- 어떤 화면군인지(`MD_ORD`, `HP_DMS`, `ER_PAY` 등)

### 5.2 둘째 단계: 이벤트 찾기
- `OnLoadCompleted`
- `OnClick`
- `OnChange`
- callback 함수

### 5.3 셋째 단계: Transaction 찾기
- `Transaction(`
- `cf_Transaction(`
- `sSvcID`
- `sSvcURL`

### 5.4 넷째 단계: 서버 추적
- `.mhi`
- navigation XML
- action
- command

## 6. 대표 화면 유형

### 6.1 로그인형
- 파일: `webapp/ui/com/Login3.xml`
- 특징:
  - 짧은 화면
  - 단순 transaction
  - authNavi와 바로 연결
- 용도:
  - 기본 패턴 학습용

### 6.2 대형 처방형
- 파일: `webapp/ui/MD/ORD/MD_ORD01001P.xml`
- 특징:
  - 조회, 저장, 보조조회, 탭 전환, 전처방, CPS가 한 화면에 몰림
  - `.mhi` 종류가 많음
- 용도:
  - 실제 유지보수 난도가 높은 MiPlatform 대형 화면 사례

### 6.3 DMS/심사형
- 파일: `webapp/ui/HP/DMS/HP_DMS02204M.xml`
- 특징:
  - `RetrieveDrgRevwPtList.mhi`
  - `UpdateDrgRcpnNo.mhi`
- 용도:
  - 조회형 심사 화면 패턴

### 6.4 EDI 수신형
- 파일: `webapp/ui/HP/DMS/HP_DMS01303M.xml`
- 특징:
  - `RetrieveEdiRecvRcpn.mhi`
  - `SaveEdiRecvRcpn.mhi`
  - `clamNavi.xml`로 연결
- 용도:
  - 입력/조회가 짧고 명확한 front-channel trace 템플릿

## 7. 파일 구조 참고

### 7.1 NPH_HIS/webapp
- `index330.jsp`
- `ui/`
- `eView/`
- `jsp/`
- `WEB-INF/`

### 7.2 ui 하위
- `NPH_start.xml`
- `com/`
- `MD/`
- `HP/`
- `ER/`
- `SP/`
- `AZ/`

### 7.3 devonhome 하위
- `navigation/`
- `xmlquery/`
- `conf/`

## 8. 이 문서를 읽을 때 주의할 점

- 이 문서는 reference 문서다.
- 제품 일반론과 NPH 실구현을 같이 설명하지만, **최종 판단 근거는 항상 로컬 소스**다.
- 개념 설명은 공식 매뉴얼을 참고했고, 경로/패턴 예시는 NPH 코드 확인값만 남겼다.

## 9. 연결 문서

- [A.Miplatform.md](./A.Miplatform.md)
- [B.MiPlatform-Transaction-패턴.md](./B.MiPlatform-Transaction-%ED%8C%A8%ED%84%B4.md)
- [C.Dataset-입출력.md](./C.Dataset-%EC%9E%85%EC%B6%9C%EB%A0%A5.md)
- [A.Front-Channel-개요.md](../0313.ui-entry/A.Front-Channel-%EA%B0%9C%EC%9A%94.md)
- [B.화면XML-script-mhi-연결.md](../0313.ui-entry/B.%ED%99%94%EB%A9%B4XML-script-mhi-%EC%97%B0%EA%B2%B0.md)
- [E.대표화면-EDI-수신-패턴.md](../0313.ui-entry/E.%EB%8C%80%ED%91%9C%ED%99%94%EB%A9%B4-EDI-%EC%88%98%EC%8B%A0-%ED%8C%A8%ED%84%B4.md)

## 8. 일반 CRUD형에 가까운 대표 패턴

### 8.1 AZ_UTL01002P
- 파일: `NPH_HIS/webapp/ui/AZ/UTL/AZ_UTL01002P.xml`
- 성격: 공통코드 조회 팝업
- 직접 확인된 흐름:
  - `form_OnLoadCompleted()`
  - `div_search_btn_Search_OnClick()` / `div_search_ed_search_OnKeyDown()`
  - `fRetrieveComncd()`
  - `cf_Transaction("RetrieveComnCd", "/az/bizcom/comNavi/RetrieveComnCd.mhi", ...)`
  - `comNavi.xml`의 `RetrieveComnCd`
  - `RetrieveComnCdCMD`
- 의미:
  - 의료 특화 로직이 거의 없는 공통 팝업형 화면이라, MiPlatform의 기본 추적 패턴을 연습하기에 좋다.
  - `Design + Data + Event`가 한 XML 안에 모여 있고, `Dataset -> Grid binding -> callback`까지 짧게 닫힌다.



