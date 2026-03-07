# iText XML Worker (PDF 처리)

> 최종 수정: 2026-03-08

---

## 1. 개요

NPH 시스템에서 iText XML Worker의 **직접 확인 경로**는 `eView/emrtopdf.jsp` 이다. `TPR`과 같은 의료 출력 파일군 주변에도 `xmlworker-1.2.0.jar`가 함께 존재하지만, 현재 코드베이스 기준으로 TPR JSP 내부에서 iText를 직접 사용하는 근거는 아직 확인되지 않았다.

---

## 2. JAR 파일

### 2.1 라이브러리 목록

| 라이브러리 | 파일명 | 버전 | 위치 |
|-----------|--------|------|------|
| **XML Worker** | xmlworker-1.2.0.jar | 1.2.0 | `webapp/EMR_DATA/applet/` |

### 2.2 버전 호환

| XML Worker | iText 버전 |
|------------|------------|
| 1.2.0 | iText 5.0.0 ~ 5.2.0 |

**참고**: iText 코어 라이브러리 JAR은 현재 WEB-INF/lib에서 직접 확인되지 않았지만, `emrtopdf.jsp`의 import는 `com.itextpdf.*` 계열이다.

---

## 3. Import 문

### 3.1 emrtopdf.jsp

```java
<%@ page import = "com.itextpdf.text.Document"%>
<%@ page import = "com.itextpdf.text.DocumentException"%>
<%@ page import = "com.itextpdf.text.pdf.PdfWriter"%>
<%@ page import = "com.itextpdf.tool.xml.XMLWorkerHelper"%>
```

### 3.2 직접 확인 코드

```java
Document document = new Document();
PdfWriter writer = PdfWriter.getInstance(document, outputStream);
document.open();
XMLWorkerHelper.getInstance().parseXHtml(writer, document, htmlInputStream);
document.close();
```

---

## 4. 사용 위치

### 4.1 Applet/출력 파일군

```
webapp/EMR_DATA/applet/
├── TPRreport.jar
├── xmlworker-1.2.0.jar
├── painter.jar
├── signedpainter.jar
├── SignPad.jar
├── ChartController.jar
├── PedigreeChart.jar
├── QuickPDFDLL0816.dll
├── novapdf-full.exe
└── EDViewer_Ocx.ocx
```

### 4.2 PDF 관련 JSP

| 파일 | 위치 | 용도 |
|------|------|------|
| **emrtopdf.jsp** | `webapp/eView/` | EMR 문서 PDF 변환 (직접 근거) |
| **TprReport.jsp** | `webapp/EMR_DATA/`, `webapp/eView/` | TPR 리포트 JSP 파일군 |
| **TPRReportBohum_2.jsp** | `webapp/EMR_DATA/` | TPR 보고서 HTML 생성 |

---

## 5. 기술 스택

### 5.1 PDF 처리 방식

```mermaid
flowchart LR
    HTML["HTML"] --> JSP["emrtopdf.jsp"]
    JSP --> XW["XMLWorkerHelper"]
    XW --> PDF["PDF"]
```

### 5.2 주요 기능

| 기능 | 설명 |
|------|------|
| **HTML 파싱** | HTML 태그를 PDF 요소로 변환 |
| **스타일 적용** | CSS 스타일 PDF 반영 |
| **이미지 처리** | HTML 내 이미지 PDF 포함 |
| **한글 지원** | UTF-8 인코딩 처리 |

---

## 6. 인접 파일군 예시

```java
// BkmakeTPRreport.java
public static String base64Encode(String str) throws IOException {
    sun.misc.BASE64Encoder encoder = new sun.misc.BASE64Encoder();
    byte[] b1 = str.getBytes("utf-8");
    return encoder.encode(b1);
}
```

이 코드는 TPR 주변 파일군에 존재하지만, 이것만으로 XML Worker 직접 사용을 뜻하지는 않는다.

---

## 7. 연동 구조

### 7.1 TPR Report와의 관계

| 단계 | 설명 |
|------|------|
| 1. 직접 확인 | `emrtopdf.jsp`에서 `Document`, `PdfWriter`, `XMLWorkerHelper` import/use 확인 |
| 2. 인접 파일군 | `TPRreport.jar`, `TprReport.jsp`, `TPRReportBohum_2.jsp`, `BK_PDF_COMMMON.js`와 함께 배치 |
| 3. 미확인 부분 | TPR JSP 내부의 iText 직접 호출은 현재 미확인 |

### 7.2 데이터 흐름

```
HTML → emrtopdf.jsp → XML Worker → PDF
```

---

## 8. PDF 관련 컴포넌트

| 파일 | 용도 |
|------|------|
| **EDViewer_Ocx.ocx** | EMR 문서 뷰어 |
| **QuickPDFDLL0816.dll** | QuickPDF 라이브러리 |
| **novapdf-full.exe** | novaPDF PDF 프린터 설치 파일 |
| **BK_PDF_COMMMON.js** | PDF 관련 공통 JavaScript |

---

## 9. 주요 업무

| 업무 | 용도 |
|------|------|
| **EMR 문서 PDF** | EMR 뷰어에서 문서를 PDF로 변환 |
| **리포트 PDF 인접 파일군** | TPR 주변 파일군과 같이 배치되어 있으나 직접 호출은 추가 확인 필요 |
| **HTML 변환** | HTML 포맷 데이터를 PDF로 변환 |

---

## 10. 요약

| 구분 | 내용 |
|------|------|
| **라이브러리** | iText XML Worker 1.2.0 |
| **버전** | iText 5.0.0 ~ 5.2.0 호환 |
| **위치** | `webapp/EMR_DATA/applet/` |
| **용도** | HTML to PDF 변환 |
| **연동** | 직접 확인은 `emrtopdf.jsp`, TPR은 인접 파일군 수준 |
| **주요 파일** | `emrtopdf.jsp` |

---

## 11. 관련 문서

- [README.md](./README.md)
- [D.TPR-Report.md](./D.TPR-Report.md)
- [B.Rexpert-리포트엔진.md](./B.Rexpert-리포트엔진.md)
