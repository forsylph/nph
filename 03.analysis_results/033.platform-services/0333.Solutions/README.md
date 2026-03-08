# Solutions 개요

> 최종 수정: 2026-03-08

---

## 1A. 상위 연결

- 이 폴더의 기준 설명은 [../README.md](../README.md) 를 먼저 본다.
- DevOn 코어는 [../../032.framework-core/0321.overview/A.Framework-개요.md](../../032.framework-core/0321.overview/A.Framework-개요.md) 와 같이 본다.

이 문서는 DevOn 바깥에서 동작하는 공통 솔루션과 플랫폼 공통 패키지를 정리하는 기준본이다.

---

## 1B. 직접 확인 근거 파일

| 기술 | 직접 확인 근거 |
|------|----------------|
| Rexpert | `rexservice.jsp`, `devonhome/conf/repConf/conf/DataSource.properties`, 다수 화면 XML의 `cf_PreviewReport()` |
| Quartz | `quartz-1.6.1.jar`, 코드베이스 전역 `org.quartz` 직접 사용 미확인 |
| DevOn Batch | `devon-batch-core-1.1.0.jar`, `devon-batch-scheduler.jar`, `BatchExecutor.cmd`, `devon-batch-scheduler.xml` |
| TPR | `TprServlet.java`, `TprServletMethod.java`, `TprReport.jsp`, `sql.xml`, `sqlParameter.xml`, `TPRsetup.XML`, `TPRreport.jar` |
| Excel | `poi-3.2-FINAL-20081019.jar`, `jxl.jar`, `utilLib.js`의 Grid Excel 관련 함수 |
| PDF | `eView/emrtopdf.jsp`, `itext-2.1.7.jar`, `xmlworker-1.2.0.jar` |

---

## 2. 분석 현황

| 솔루션 | 상태 | 문서 |
|--------|------|------|
| **Rexpert** | ✅ 분석 완료 | [B.Rexpert-리포트엔진.md](./B.Rexpert-리포트엔진.md) |
| **Quartz** | ✅ 분석 완료 | [C.Quartz-스케줄러.md](./C.Quartz-스케줄러.md) |
| **TPR Report** | ✅ 분석 완료 | [D.TPR-Report.md](./D.TPR-Report.md) |
| **Excel 라이브러리** | ✅ 분석 완료 | [E.Excel-라이브러리.md](./E.Excel-라이브러리.md) |
| **iText XML Worker** | ✅ 분석 완료 | [F.iText-PDF.md](./F.iText-PDF.md) |

---

## 3. 기술 스택 요약

### 3.1 리포트 엔진

| 기술 | 버전 | 공급사 | 비고 |
|------|------|--------|------|
| **Rexpert** | 3.x 계열 | - | `rexservice.jsp`, `rexpert.js`, `cf_PreviewReport()` 기준 사용 확인 |
| **Rexpert Viewer** | 1.0.0.57 | - | ActiveX/Plugin 계열 흔적 |
| **TPR Report** | Version 76 흔적 | - | `TprServlet`, `TprReport.jsp`, `sql.xml`, `TPRsetup.XML` 기준 확인 |

### 3.2 리포트 파일 형식

| 형식 | 설명 |
|------|------|
| `.reb` | Rexpert 리포트 템플릿 (REX3 바이너리) |
| `.oof` | OOF (Object Oriented Format) 데이터 |

### 3.3 스케줄러

| 기술 | 버전 | 비고 |
|------|------|------|
| **Quartz** | 1.6.1 | JAR 포함, 현재 코드베이스에서 `org.quartz` 직접 사용 미확인 |
| **DevOn Batch** | 1.1.0 | 실제 배치 실행 코어, 상세는 `032.framework-core/0323.batch-rule` 참조 |

### 3.4 Excel/PDF 처리

| 기술 | 버전 | 비고 |
|------|------|------|
| **Apache POI** | 3.2 (2008) | 외부 Excel 라이브러리, DevOn Excel API 문서와 함께 존재 |
| **JExcelApi** | - | 외부 Excel 라이브러리, DevOn Excel API 문서와 함께 존재 |
| **iText XML Worker** | 1.2.0 | `emrtopdf.jsp` 기준 HTML to PDF 변환 확인 |

---

## 4. 문서별 한 줄 요약

| 문서 | 한 줄 요약 |
|------|------------|
| [B.Rexpert-리포트엔진.md](./B.Rexpert-리포트엔진.md) | NPH에서 실제 강하게 확인되는 보고서 출력 스택과 제품 일반론을 분리한 문서 |
| [C.Quartz-스케줄러.md](./C.Quartz-스케줄러.md) | Quartz JAR 존재와 현행 DevOn Batch 운영 구조를 분리해 정리한 문서 |
| [D.TPR-Report.md](./D.TPR-Report.md) | TPR 보고서 엔진의 서블릿/JSP/설정 파일 기반 사용 흔적을 정리한 문서 |
| [E.Excel-라이브러리.md](./E.Excel-라이브러리.md) | Apache POI/JExcelApi 존재와 실제 Excel 기능 확인 범위를 정리한 문서 |
| [F.iText-PDF.md](./F.iText-PDF.md) | iText/XML Worker 기반 PDF 생성의 직접 사용 근거를 정리한 문서 |

---

## 5. 아키텍처 위치

```mermaid
flowchart LR
    UI["화면 XML"] --> JSP["JSP/JS"]
    JSP --> Rex["Rexpert"]
    JSP --> TPR["TPR"]
    JSP --> Excel["Excel util"]
    JSP --> PDF["iText/XMLWorker"]
```

---

## 6. 솔루션 사용 현황

### 5.1 Rexpert

- `rexservice.jsp`, `rexpert.js`, `rexpert_properties.js`가 존재한다.
- 다수 화면 XML에서 `cf_PreviewReport()`, `cf_ViewReport()`, `cf_printReport()` 호출이 확인된다.
- 따라서 Rexpert는 NPH에서 현재도 강하게 확인되는 보고서 출력 스택이다.

### 5.2 Quartz / DevOn Batch

- `quartz-1.6.1.jar`는 포함되어 있다.
- 현재 코드베이스에서는 `org.quartz`, `SchedulerFactory`, `JobDetail`, `Trigger` 직접 사용을 확인하지 못했다.
- 실제 운영 표면은 `BatchExecutor.cmd`, `batchMgr`, `devon-batch-scheduler.xml`, `BatchInfoUC` 쪽이 더 강하다.

### 5.3 Excel / PDF

- `poi-3.2-FINAL-20081019.jar`, `jxl.jar`, `xmlworker-1.2.0.jar`은 포함되어 있다.
- Excel은 현재 `utilLib.js`와 MiPlatform Grid 메서드 기반의 화면 기능이 더 강하게 확인된다.
- PDF는 `eView/emrtopdf.jsp`의 `com.itextpdf.*`, `XMLWorkerHelper` import가 직접 근거다.

---

## 7. 파일 구조

```
0333.Solutions/
├── README.md
├── B.Rexpert-리포트엔진.md
├── C.Quartz-스케줄러.md
├── D.TPR-Report.md
├── E.Excel-라이브러리.md
└── F.iText-PDF.md
```

---

## 8. 분류 기준

- DevOn 자체 실행 구조가 아닌 외부 솔루션/패키지 설명은 여기서 관리한다.
- 단, 배치/룰의 DevOn 실행 구조 설명은 `032.framework-core`에 둔다.
- 의료 특화 솔루션의 기준본은 `035.Biz-medical-Domain`에 둔다.

---

## 9. 빠른 판단

- `Rexpert`, `TPR`, `iText/XML Worker`는 현재 코드/설정/JSP 기준 직접 근거가 있다.
- `Quartz`는 JAR 존재는 명확하지만, 현재 코드베이스에서 `org.quartz` 직접 사용은 확인되지 않았다.
- `Excel`은 외부 라이브러리 실체와 화면 기능 흔적은 있으나, 서버 코드 직접 import는 추가 확인이 필요하다.

---

## 10. 다음 단계

1. Rexpert 제품 일반론과 NPH 실증 구간 분리 유지
2. TPR의 실제 사용 화면과 호출 체인 추가 확인
3. Excel/iText의 서버측 직접 사용 경로 추가 확인

---

## 11. 관련 문서

- [B.Rexpert-리포트엔진.md](./B.Rexpert-리포트엔진.md)
- [Tech-Stack-개요.md](../../030.index/0307.Tech%20Stack/Tech-Stack-개요.md)
