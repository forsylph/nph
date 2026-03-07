# HP_DMS02204M 실행 체인 복원

## 1. 문서 목적

이 문서는 `HP_DMS02204M` 화면의 실제 조회 체인을 `화면 -> navigation -> CMD -> PC -> EC -> query path -> xmlquery` 기준으로 닫기 위한 문서다.

## 2. 대표 진입 경로

- 화면 URL: `/hp/dms/drgNavi/RetrieveDrgRevwPtList.mhi`
- navigation: `devonhome/navigation/mhi/hp/dms/drgNavi.xml`
- action: `RetrieveDrgRevwPtList`
- command: `nph.his.hp.dms.drg.cmd.RetrieveDrgRevwPtListCMD`
- service 진입: `TxServiceUtil.getNTxService("hp.dms.DrgPostRevwMngmPC")`

## 3. PC -> EC

### 3.1 PC

`DrgPostRevwMngmPC`

- `retrieveDrgRevwPtList(data)`는 `PostRevwEC.retrieveDrgRevwPtList(data)`로 위임한다
- 같은 PC 안에 `ROW_STATUS_CREATE`, `ROW_STATUS_UPDATE`, `ROW_STATUS_DELETE` 분기가 다수 존재한다
- 즉 조회 전용 PC라기보다, 심사 후처리 업무를 함께 품은 PC다

### 3.2 EC

`PostRevwEC`

- `retrieveDrgRevwPtList(data)`
  - query path: `/hp/dms/hpdmhdmbs/retrieveDrgRevwPtList`
- 같은 EC 안에 아래 계열도 함께 존재한다
  - `saveDrgPostRevw`
  - 다수 `update*`
  - 다수 `delete*`

즉 이 체인은 단순 조회 전용으로만 분리된 것이 아니라, 같은 파일군에 저장/후처리 책임까지 묶여 있다.

## 4. query path -> xmlquery

- xmlquery 파일: `devonhome/xmlquery/hp/dms/hpdmhdmbs.xml`
- 확인된 statement:
  - `retrieveDrgRevwPtList`
  - `saveDrgPostRevw`
  - 다수 `update*`
  - 다수 `delete*`

해석:

- `HP_DMS02204M`는 화면만 보면 조회 중심이다
- 하지만 실제로는 `hpdmhdmbs.xml`이라는 큰 도메인 파일군의 일부를 사용한다
- 그래서 당시 구조가 `command -> PC -> EC -> xmlquery`로 짜인 이유는, 조회/저장/후처리를 하나의 심사 도메인 파일군으로 통제하려는 의도였다고 보는 편이 맞다

## 5. 결론

`HP_DMS02204M`는 `EdiMngmPC`처럼 분기 폭이 큰 화면은 아니다. 대신 다음을 보여준다.

- 조회 화면처럼 보여도 내부 도메인 파일군은 두껍다
- `DrgPostRevwMngmPC`는 단순 조회 wrapper가 아니다
- `PostRevwEC`와 `hpdmhdmbs.xml`은 심사 후처리까지 같이 품고 있다

즉 이 화면은 `프레임워크가 괜히 과한가`라는 질문에 대해, 적어도 일부 도메인에서는 그렇게 단정하기 어렵다는 반례가 된다.
