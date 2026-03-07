# Dataset 입출력

약어/용어는 [030.index 용어집](../../030.index/0303.약어-용어집/약어-용어집.md)을 먼저 보면 빠르다.

이 문서는 NPH MiPlatform 화면에서 Dataset이 입력과 출력에 어떻게 쓰이는지, 그리고 서버로 넘어갈 때 어떤 계층을 거치는지 현재 확인된 범위로 정리한 문서다.

## 1. 한 줄 요약

```mermaid
flowchart LR
    DSIn[입력 Dataset] --> Tx[Transaction]
    Tx --> MHI[.mhi]
    MHI --> Conv[MiplatformConverter]
    Conv --> LData[LData/LMultiData]
    LData --> Cmd[command]
```

반대로 응답은 이 순서를 거꾸로 탄다.

## 2A. 공식 매뉴얼 기준 Dataset 해석

MiPlatform 3.3 PID Developer 가이드의 `화면(Form) 개발` 문서는 Dataset을 `Table 형태의 기억장소`로 설명한다. 또한 다음 성격을 같이 강조한다.
- Server Side Service Object와 연결 가능
- Client Side에서 Table 구조를 생성해 사용할 수도 있음
- Component와 Dataset이 bind되면 데이터 변경이 자동으로 동기화될 수 있음

이 설명을 NPH에 그대로 대입하면 다음처럼 읽는 것이 맞다.
- 화면 XML 안의 Dataset은 단순 변수 묶음이 아니라, 서버 통신 단위이자 화면 bind 단위다.
- `sInputDs`, `sOutputDs`는 단순 문자열이 아니라, 어떤 Dataset을 보내고 어떤 Dataset을 다시 그리드/폼에 반영할지 결정하는 경계다.
- Grid, Edit, Combo 같은 화면 컴포넌트가 깨졌을 때는 화면 컴포넌트 자체보다 Dataset 이름과 binding 관계를 먼저 보는 편이 빠르다.

매뉴얼에는 `Const Column`으로 통신량을 줄일 수 있다는 설명도 나온다. NPH 문서화에서는 아직 Dataset 세부 컬럼 최적화까지는 닫지 못했지만, Dataset을 `화면용 구조 + 통신용 구조`로 함께 보는 관점은 유지할 필요가 있다.

## 2. 화면에서 보이는 Dataset 사용

### 2.1 로그인 화면
- `Login3.xml`에서 `Transaction("CheckUserInfo", ..., inds, outds, params, ...)` 형태가 직접 보인다.
- 즉 화면 스크립트가 입력 Dataset 이름과 출력 Dataset 이름을 함께 넘긴다.

### 2.2 대형 업무 화면
- `MD_ORD01001P.xml`, `HP_DMS02204M.xml`에서는 대체로 다음 패턴이 반복된다.
  - `sInputDs`
  - `sOutputDs`
  - `sParam`
  - `cf_Transaction(sSvcID, sSvcURL, sInputDs, sOutputDs, sParam, sCallBack)`

실무적으로는 이 네 가지를 같이 봐야 한다.
- 어떤 Dataset을 보냈는가
- 어떤 Dataset을 받을 것으로 기대하는가
- 추가 파라미터는 무엇인가
- callback에서 어떤 `srvID`로 분기하는가

## 3. 서버 진입 뒤의 변환층

현재 코드 기준 직접 확인된 클래스:
- `MiplatformServlet`
- `MiplatformRequest`
- `MiplatformConverter`
- `AbstractMiplatformCommand`

현재 확인된 핵심 메서드:
- `convertToLMultiDataWithJobType(Dataset ds, Dataset sessionDs)`
- `convertToDataset(...)`

해석:
- 화면에서 보낸 Dataset은 그대로 business 계층으로 가지 않는다.
- `MiplatformConverter`가 `LData / LMultiData`로 변환한다.
- command는 변환된 내부 데이터 구조를 기준으로 동작한다.

## 4. 실무에서 먼저 보는 포인트

### 4.1 조회가 안 될 때
- 화면의 `sInputDs` 구성
- callback에서 기대하는 `sOutputDs`
- command 쪽에서 기대하는 Dataset 이름
을 먼저 맞춘다.

### 4.2 저장이 안 될 때
- row 상태
- `_CUD` 성격의 입력 Dataset
- 저장 후 callback에서 다시 조회를 타는지
를 본다.

### 4.3 데이터는 오는데 화면이 안 바뀔 때
- callback 함수의 `srvID` 분기
- 출력 Dataset 이름 오탈자
- grid/bind 대상 Dataset 이름
을 먼저 본다.

## 5. 현재 문서가 말하는 범위

이 문서는 Dataset의 `형식`과 `변환층`을 설명한다.
- 개별 화면의 실제 trace는 [037.runtime-trace](../../037.runtime-trace/%ED%8A%B8%EB%A0%88%EC%9D%B4%EC%8A%A4-%EC%9D%BD%EB%8A%94%EC%88%9C%EC%84%9C.md)에서 본다.
- command 이후 구조는 [032.framework-core](../../032.framework-core/0321.overview/03.Architecture-overview.md)에서 본다.
