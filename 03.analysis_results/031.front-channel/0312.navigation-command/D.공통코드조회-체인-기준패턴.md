# 공통코드조회-체인-기준패턴

약어/용어는 [030.index 용어집](../../030.index/0303.약어-용어집/약어-용어집.md)을 먼저 보면 빠르다.

이 문서는 `AZ_UTL01002P -> RetrieveComnCd.mhi` 흐름을 기준으로, `031.front-channel`에서 `032.framework-core`로 내려가기 직전의 가장 짧은 command 체인을 고정하기 위한 기준본이다.

## 1. 직접 확인된 파일

- 화면 XML
  - `NPH_HIS/webapp/ui/AZ/UTL/AZ_UTL01002P.xml`
- navigation
  - `NPH_HIS/devonhome/navigation/mhi/az/bizcom/comNavi.xml`
- command
  - `NPH_HIS/src/nph/his/az/bizcom/com/cmd/RetrieveComnCdCMD.java`
- PC interface
  - `NPH_HIS/src/nph/his/az/bizcom/com/pc/CommonIFPC.java`
- PC implementation
  - `NPH_HIS/src/nph/his/az/bizcom/com/pc/CommonPC.java`
- EC
  - `NPH_HIS/src/nph/his/az/zzaz/bizcom/com/CommonEC.java`
- xmlquery
  - `NPH_HIS/devonhome/xmlquery/az/bizcom/azcmmcmcd.xml`

## 2. 전체 체인

```mermaid
flowchart LR
    UI[AZ_UTL01002P] --> MHI[RetrieveComnCd.mhi]
    MHI --> Navi[comNavi.xml]
    Navi --> CMD[RetrieveComnCdCMD]
    CMD --> PC[CommonIFPC/CommonPC]
    PC --> EC[CommonEC]
    EC --> XQ[azcmmcmcd.xml]
```

## 3. navigation -> command

### 3.1 navigation
- `comNavi.xml`
- 직접 확인된 action
  - `<action name="RetrieveComnCd">`
  - `<command>nph.his.az.bizcom.com.cmd.RetrieveComnCdCMD</command>`
- 같은 command를 `NotLoginRetrieveComnCd`에서도 재사용한다.

### 3.2 command
- `RetrieveComnCdCMD extends AbstractMiplatformCommand`
- 핵심 흐름:
  - `TxServiceUtil.getNTxService("az.bizcom.CommonPC")`
  - `String dsName = data.getString("dsName")`
  - `commonPc.retrieveComnCd(data)`
  - `platformResponse.addDataset(dsName, mResult)`

해석:
- command는 business logic을 많이 담지 않는다.
- 입력 `data`를 `CommonPC`로 넘기고, 결과 `LMultiData`를 다시 Dataset으로 올리는 얇은 adapter에 가깝다.

## 4. PC interface / implementation

### 4.1 CommonIFPC
- `retrieveComnCd(LData data)`
- `CommonPC`의 표면 계약 역할

### 4.2 CommonPC
- `retrieveComnCd(LData data)`
- 직접 확인된 핵심 라인:
  - `LMultiData lResult = commonEc.retrieveComnCd(data);`

해석:
- 이 경로에서는 `PC`도 복잡한 오케스트레이션보다 `EC` 위임 역할이 강하다.
- 그래서 이 패턴은 `CMD -> PC -> EC`의 가장 얇은 예시로 쓰기 좋다.

## 5. EC -> xmlquery

### 5.1 CommonEC
- `retrieveComnCd(LData data)`
- `retrieveComnCdNew(LData data)`도 존재

### 5.2 xmlquery
- 파일: `azcmmcmcd.xml`
- statement: `retrieveComnCd`
- 직접 확인된 SQL 구조:
  - `FROM AZCMMCMCD`
  - `WHERE CLSF_CD = ${clsfCd}`
  - `searchKey`, `searchDiv`, `sortColumn`, `displayYn`에 따라 `<append>` 분기

해석:
- 화면에서 넘긴 조건이 거의 그대로 xmlquery 분기로 내려간다.
- 즉 이 체인은 `MiPlatform 입력값 -> command -> PC -> EC -> xmlquery`의 가장 깨끗한 예시다.

## 6. 왜 이 문서가 필요한가

`D.대표화면-공통코드조회-패턴.md`는 `화면 XML -> .mhi` 중심 문서다.
이 문서는 그 다음 단계인 `navigation -> command -> PC -> EC -> xmlquery`를 짧게 이어 붙여 보여준다.

그래서 두 문서를 같이 보면 아래 체인이 한 번에 닫힌다.

1. 화면 XML과 script
2. `.mhi`
3. navigation
4. command
5. PC/EC
6. xmlquery

## 7. 연결 문서

- [D.대표화면-공통코드조회-패턴.md](../0313.ui-entry/D.%EB%8C%80%ED%91%9C%ED%99%94%EB%A9%B4-%EA%B3%B5%ED%86%B5%EC%BD%94%EB%93%9C%EC%A1%B0%ED%9A%8C-%ED%8C%A8%ED%84%B4.md)
- [A.Command-Navigation-Dispatch.md](./A.Command-Navigation-Dispatch.md)
- [B.LCommonDao-LQueryMaker.md](../../032.framework-core/0322.data-access/B.LCommonDao-LQueryMaker.md)
- [C.XML-Query-실행구조.md](../../032.framework-core/0322.data-access/C.XML-Query-%EC%8B%A4%ED%96%89%EA%B5%AC%EC%A1%B0.md)




