# Excel 라이브러리 (Apache POI / JExcelApi)

> 최종 수정: 2026-03-08

---

## 1. 개요

NPH 시스템은 Apache POI와 JExcelApi 두 가지 엑셀 처리 라이브러리를 사용하며, DEVON Framework가 이를 추상화하여 제공한다.

---

## 2. JAR 파일

### 2.1 라이브러리 목록

| 라이브러리 | 파일명 | 버전 | 크기 |
|-----------|--------|------|------|
| **Apache POI** | poi-3.2-FINAL-20081019.jar | 3.2 Final | 1.4 MB |
| **JExcelApi** | jxl.jar | - | 743 KB |

### 2.2 위치

```
NPH_HIS/webapp/WEB-INF/lib/
├── poi-3.2-FINAL-20081019.jar    # Apache POI 3.2 (2008)
└── jxl.jar                        # JExcelApi
```

---

## 3. DEVON Framework Excel API

### 3.1 패키지 구조

```
devonframework/service/excel/
├── poi/                           # Apache POI 래퍼
│   ├── LPOIConstants.java          # POI 상수
│   ├── LPOIDao.java                # POI DAO
│   ├── LPOIFactory.java            # POI 팩토리 (deprecated)
│   ├── LPOIReaderIF.java           # POI 리더 인터페이스 (deprecated)
│   ├── LPOIWriterIF.java           # POI 라이터 인터페이스 (deprecated)
│   ├── LPOIException.java          # POI 예외
│   ├── reader/                     # 리더 서브패키지
│   └── writer/                     # 라이터 서브패키지
├── jxl/                           # JExcelApi 래퍼
│   ├── process/
│   │   └── LExcelBatchConsole.java  # 배치 콘솔
│   └── util/
│       ├── LExcelBatchDAO.java      # 배치 DAO
│       ├── LExcelBatchResult.java   # 배치 결과
│       ├── LExcelConverter.java     # 변환기
│       ├── LSQLQuery.java           # SQL 쿼리
│       └── LSheetBatchResult.java   # 시트 결과
└── common/                         # 공통 기능
```

### 3.2 주요 클래스

| 클래스 | 패키지 | 설명 |
|--------|--------|------|
| `LPOIDao` | devonframework.service.excel.poi | POI 기반 DAO |
| `LPOIConstants` | devonframework.service.excel.poi | POI 상수 정의 |
| `LPOIException` | devonframework.service.excel.poi | 예외 처리 |
| `LExcelBatchProcessor` | devonframework.service.excel.jxl | JXL 배치 처리 |
| `LExcelConverter` | devonframework.service.excel.jxl | Excel 변환기 |

---

## 4. 사용 패턴

### 4.1 엑셀 다운로드

```mermaid
flowchart LR
    A[화면 Grid] --> B[JavaScript 함수]
    B --> C[MiPlatform SaveExcel]
    C --> D[엑셀 파일 생성]
    D --> E[다운로드]
```

### 4.2 JavaScript 함수 (utilLib.js)

| 함수명 | 용도 |
|--------|------|
| `cf_SaveExcel(objGrid, sDocName, bMerge)` | Grid 내용을 Excel로 저장 |
| `cf_Excel_Export(strTitle, objGrid, objFileDialog)` | Grid를 Excel로 내보내기 |
| `cf_Excel_Export_Log(strTitle, objGrid, objFileDialog)` | 개인정보 로깅 포함 내보내기 |
| `cf_Excel2Dataset(objDs, sFileName, nSheet, bColInfo)` | Excel에서 Dataset으로 가져오기 |
| `cf_Excel2Grid(sGridID, sFileName, nSheet)` | Excel을 Grid로 가져오기 |

### 4.3 MiPlatform Grid 메서드

| 메서드 | 설명 |
|--------|------|
| `objGrid.SaveExcel()` | Grid를 Excel로 저장 |
| `objGrid.SaveExcelEx()` | Grid를 Excel로 저장 (확장) |
| `objGrid.ExportExcelEx()` | Grid를 Excel로 내보내기 |
| `ext_ExcelImportByIndex()` | Excel 가져오기 (인덱스 기반) |

---

## 5. 엑셀 관련 화면

### 5.1 발견된 화면 파일

| 업무 영역 | 화면 ID | 용도 |
|-----------|---------|------|
| **AZ/STA** | AZ_STA01202M ~ AZ_STA04906M | 통계 엑셀 다운로드 |
| **MR/COM** | MR_COM99002M ~ MR_COM99008M | 공통 엑셀 기능 |
| **MR/RCH** | MR_RCH01053M, MR_RCH01086M, ... | 접수 관련 엑셀 |
| **MR/RDI** | MR_RDI01002M, MR_RDI01016M | 예약/접수 엑셀 |
| **HP/DMS** | HP_DMS04201M, HP_DMS05120M | 병원 관리 엑셀 |
| **ER/INS** | ER_INS09001M | 응급 보험 엑셀 |
| **SP/LAB** | SP_LAB01082M | 검사 엑셀 |

### 5.2 엑셀 템플릿

```
webapp/EMR_DATA/CCM_upload.xlsx    # CCM(임상경로관리) 업로드용 템플릿
```

---

## 6. 기술 비교

### 6.1 POI vs JExcelApi

| 항목 | Apache POI | JExcelApi |
|------|------------|-----------|
| **버전** | 3.2 (2008) | - |
| **지원 형식** | .xls, .xlsx | .xls only |
| **기능** | 읽기/쓰기 | 읽기/쓰기 |
| **성능** | 상대적 느림 | 상대적 빠름 |
| **특징** | 기능 풍부 | 가벼움 |

### 6.2 DEVON 추상화

```
             ┌─────────────────────┐
             │   Application Code  │
             └─────────┬───────────┘
                       │
             ┌─────────▼───────────┐
             │  DEVON Excel API    │
             │  (추상화 계층)       │
             └─────────┬───────────┘
           ┌───────────┴───────────┐
           │                       │
    ┌──────▼──────┐        ┌───────▼──────┐
    │ Apache POI  │        │  JExcelApi   │
    └─────────────┘        └──────────────┘
```

---

## 7. 코드 예시

### 7.1 엑셀 다운로드 CMD

```java
// QC결과리스트 엑셀 다운로드용 CMD 예시
public class RetrieveQcRsltListRptCMD extends AbstractMiplatformCommand {
    public void execute() throws Exception {
        QcIFPC qcPC = (QcIFPC)TxServiceUtil.getNTxService("sp.lab.QcPC");
        LMultiData mData = qcPC.retrieveQcRsltListRpt(data);
        platformResponse.addDataset("ds_out", mData);
    }
}
```

### 7.2 JavaScript 엑셀 함수 호출

```javascript
// Grid를 엑셀로 저장
function btn_Excel_OnClick(obj) {
    cf_SaveExcel(Grid_Main, "QC결과리스트", true);
}

// 개인정보 포함 엑셀 내보내기 (로깅)
function btn_Export_OnClick(obj) {
    cf_Excel_Export_Log("환자목록", Grid_Patient, FileDialog);
}
```

---

## 8. 파일 구조

```
webapp/
├── WEB-INF/lib/
│   ├── poi-3.2-FINAL-20081019.jar
│   └── jxl.jar
├── api/devon-framework_api/
│   └── devonframework/service/excel/
│       ├── poi/
│       │   ├── LPOIConstants.java
│       │   ├── LPOIDao.java
│       │   ├── LPOIException.java
│       │   ├── reader/
│       │   └── writer/
│       └── jxl/
│           ├── process/
│           └── util/
├── ui/LIBs/
│   └── utilLib.js              # 엑셀 관련 JavaScript 함수
└── EMR_DATA/
    └── CCM_upload.xlsx         # 엑셀 템플릿
```

---

## 9. 요약

| 구분 | 내용 |
|------|------|
| **주요 라이브러리** | Apache POI 3.2, JExcelApi |
| **추상화 계층** | DEVON Framework Excel Service |
| **사용 방식** | DEVON API 통해 POI/JXL 선택 사용 |
| **업무** | 통계, 접수, 관리자 엑셀 다운로드 |
| **버전** | POI 3.2 (2008년 버전) |

---

## 10. 관련 문서

- [A.Solutions-개요.md](./A.Solutions-개요.md)
- [D.TPR-Report.md](./D.TPR-Report.md)