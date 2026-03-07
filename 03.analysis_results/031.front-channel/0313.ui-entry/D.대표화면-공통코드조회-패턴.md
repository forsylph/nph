# 대표화면-공통코드조회-패턴

약어/용어는 [030.index 용어집](../../030.index/0303.약어-용어집/약어-용어집.md)을 먼저 보면 빠르다.

이 문서는 `AZ_UTL01002P.xml`을 기준으로, 의료 특화가 아닌 일반 MiPlatform CRUD형에 가까운 화면이 `화면 XML -> script -> .mhi -> navigation -> command`로 어떻게 이어지는지 정리한 기준본이다.

## 1. 왜 이 화면을 대표 패턴으로 쓰는가

- 공통코드 조회 팝업이라 업무 의미가 단순하다.
- 화면 XML 안에 `Design + Data + Event`가 같이 들어 있다.
- 조회 버튼, 엔터키, callback, Grid binding, 부모창 return까지 기본 패턴이 짧게 닫힌다.
- `.mhi`, `navigation`, `command`, `xmlquery`까지 실제 코드 근거로 연결하기 쉽다.

## 2. 직접 확인된 파일

- 화면 XML
  - `NPH_HIS/webapp/ui/AZ/UTL/AZ_UTL01002P.xml`
- navigation
  - `NPH_HIS/devonhome/navigation/mhi/az/bizcom/comNavi.xml`
- command
  - `NPH_HIS/src/nph/his/az/bizcom/com/cmd/RetrieveComnCdCMD.java`
- xmlquery
  - `NPH_HIS/devonhome/xmlquery/az/bizcom/azcmmcmcd.xml`

## 3. 화면 XML 관점

### 3.1 Design
- 검색영역 `div_search`
- 검색 버튼 `btn_Search`
- 결과 Grid `grd_zip`
- 선택/취소 버튼

### 3.2 Data
- `ds_comncd`
  - 공통코드 결과 Dataset
- `ds_searchDiv`
  - 코드/코드명 검색 구분 Dataset
- `Grid BindDataset="ds_comncd"`

### 3.3 Event
- `form_OnLoadCompleted`
- `div_search_btn_Search_OnClick`
- `div_search_ed_search_OnKeyDown`
- `grd_zip_OnCellDblClick`
- `fTrCallBack`

## 4. 핵심 script 흐름

```mermaid
flowchart LR
    Load[form_OnLoadCompleted] --> Search[fRetrieveComncd]
    Btn[조회 버튼/엔터] --> Search
    Search --> MHI[/az/bizcom/comNavi/RetrieveComnCd.mhi]
    MHI --> Navi[comNavi.xml]
    Navi --> CMD[RetrieveComnCdCMD]
```

### 4.1 OnLoadCompleted
- `RET_FUNC`, `CLSF_CD`, `SEARCH_KEY`, `SEARCH_DIV`, `SORT_COLUMN`을 초기화한다.
- `ds_comncd.Copy(parent.ds_comncd_pub)`처럼 부모창 Dataset을 참조한다.
- 특정 분류코드(`AZ053`)에서는 로딩 직후 바로 조회를 수행한다.

### 4.2 조회 버튼 / 엔터키
- `div_search_btn_Search_OnClick()`
- `div_search_ed_search_OnKeyDown()`
- 둘 다 `fRetrieveComncd()`로 모인다.

### 4.3 조회 함수
- `sSvcID = "RetrieveComnCd"`
- `sSvcURL = "/az/bizcom/comNavi/RetrieveComnCd.mhi"`
- `sOutputDs = "ds_comncd=ds_comncd"`
- `sParam`
  - `clsfCd`
  - `searchKey`
  - `searchDiv`
  - `sortColumn`
  - `dsName`
- `cf_Transaction(...)` 호출

### 4.4 callback
- `fTrCallBack(sSvcID, ...)`
- `case "RetrieveComnCd"`에서 결과 rowcount를 보고 메시지를 표시한다.
- 결과는 이미 `ds_comncd`에 바인딩되어 Grid에 반영된다.

## 5. navigation -> command

### 5.1 navigation
- `comNavi.xml`
- 직접 확인된 action
  - `<action name="RetrieveComnCd">`
  - `<command>nph.his.az.bizcom.com.cmd.RetrieveComnCdCMD</command>`
- 같은 command를 `NotLoginRetrieveComnCd`에서도 재사용한다.

### 5.2 command
- `RetrieveComnCdCMD extends AbstractMiplatformCommand`
- 핵심 실행 흐름:
  - `TxServiceUtil.getNTxService("az.bizcom.CommonPC")`
  - `commonPc.retrieveComnCd(data)`
  - `platformResponse.addDataset(dsName, mResult)`

즉 이 화면은 MiPlatform 화면이 넘긴 입력값을 command가 그대로 `CommonPC`로 넘기고, 그 결과를 Dataset으로 다시 올리는 전형적인 조회형 패턴이다.

## 6. query path -> xmlquery

문서 범위상 `PC/EC` 상세는 `032.framework-core`에서 다루지만, 현재 직접 확인 가능한 최종 쿼리 끝점은 아래와 같다.

- xmlquery 파일
  - `azcmmcmcd.xml`
- statement
  - `retrieveComnCd`
- 핵심 SQL 구조
  - `FROM AZCMMCMCD`
  - `WHERE CLSF_CD = ${clsfCd}`
  - `searchDiv`, `searchKey`, `sortColumn`에 따라 `<append>` 분기

즉 화면에서 넘긴 `clsfCd`, `searchKey`, `searchDiv`, `sortColumn`은 결국 `retrieveComnCd` statement의 분기 조건으로 이어진다.

## 7. 해석

이 화면은 NPH의 일반 MiPlatform 패턴을 설명하기에 적합하다.

- 의료 특화 로직이 거의 없다.
- Dataset과 Grid binding이 단순하다.
- 이벤트와 transaction 연결이 짧다.
- `.mhi -> navigation -> command -> xmlquery`까지 실제 근거로 닫힌다.

그래서 로그인보다 한 단계 복잡하고, `MD_ORD01001P`보다 훨씬 단순한 **중간 난이도 기준 패턴**으로 쓰기 좋다.

## 8. 연결 문서

- [A.Miplatform.md](../0311.miplatform/A.Miplatform.md)
- [B.MiPlatform-Transaction-패턴.md](../0311.miplatform/B.MiPlatform-Transaction-%ED%8C%A8%ED%84%B4.md)
- [C.Dataset-입출력.md](../0311.miplatform/C.Dataset-%EC%9E%85%EC%B6%9C%EB%A0%A5.md)
- [B.화면XML-script-mhi-연결.md](./B.%ED%99%94%EB%A9%B4XML-script-mhi-%EC%97%B0%EA%B2%B0.md)
- [A.Command-Navigation-Dispatch.md](../0312.navigation-command/A.Command-Navigation-Dispatch.md)
- [B.LCommonDao-LQueryMaker.md](../../032.framework-core/0322.data-access/B.LCommonDao-LQueryMaker.md)
- [C.XML-Query-실행구조.md](../../032.framework-core/0322.data-access/C.XML-Query-%EC%8B%A4%ED%96%89%EA%B5%AC%EC%A1%B0.md)


## 연결 문서

- [D.공통코드조회-체인-기준패턴.md](../0312.navigation-command/D.%EA%B3%B5%ED%86%B5%EC%BD%94%EB%93%9C%EC%A1%B0%ED%9A%8C-%EC%B2%B4%EC%9D%B8-%EA%B8%B0%EC%A4%80%ED%8C%A8%ED%84%B4.md)




