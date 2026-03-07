# FV_CommonCall API 상세 분석

> EDViewer OCX의 핵심 인터페이스 메서드 분석
> 분석 일자: 2026-03-07

---

## 1. 개요

### 1.1 FV_CommonCall이란?

**FV_CommonCall은 JavaScript 함수가 아닌 EDViewer OCX/ActiveX 컨트롤의 메서드**입니다. 웹 페이지에 임베드된 ActiveX 객체를 통해 호출됩니다.

| 항목 | 내용 |
|------|------|
| **타입** | ActiveX/OCX 메서드 |
| **소유자** | EDViewer_Ocx.ocx |
| **목적** | 의료 문서(PDF/Word/XML) 로드 및 표시 |
| **ClassID** | `clsid:879DF37E-E2E1-4C52-979D-60E0806C6E97` |

### 1.2 파일 위치

```
OCX 파일:
├── NPH_HIS/webapp/EMR_DATA/applet/EDViewer_Ocx.ocx (8,006,144 bytes)
└── NPH_ECS/webapp/EMR_DATA/eMobile/EDViewer_Ocx.ocx (8,017,384 bytes)

호출 JSP 파일:
├── NPH_HIS/webapp/eView/RecordViewer.jsp
├── NPH_HIS/webapp/eView/EdViewer.jsp
├── NPH_HIS/webapp/eView/popup/pdfViewerModule.jsp
├── NPH_HIS/webapp/eView/popup/HidepdfViewerModule.jsp
└── NPH_HIS/webapp/eView/popup/copyRecdView.jsp
```

---

## 2. 메서드 시그니처

### 2.1 기본 형식

```javascript
ED_OBJ.FV_CommonCall(parameterString);
```

### 2.2 파라미터 형식

```
[Document_URL]^?[Metadata_URL]^?[Key|Value#Key|Value#...^]
```

**구분자 설명:**
| 구분자 | 용도 |
|--------|------|
| `^` | 주요 섹션 구분 |
| `?` | URL 섹션 분리 (선택적) |
| `#` | 키-값 쌍 구분 |
| `\|` | 키와 값 분리 |

### 2.3 파라미터 구성요소

| 컴포넌트 | 설명 | 예시 |
|----------|------|------|
| `Document_URL` | 문서 파일 URL 경로 | `http://192.168.3.172:9300/EMR_DATA/document/PDF_2293449_1.xml` |
| `Metadata_URL` | XML 메타데이터 파일 URL | `http://192.168.3.172:9300/EMR_DATA/xml/2014/11_11/2293449.XML` |
| `Key\|Value` | 메타데이터 키-값 쌍 | `DOC_CODE\|2293449` |

### 2.4 메타데이터 필드

| 키 | 설명 | 필수여부 |
|-----|------|----------|
| `DOC_CODE` | 문서 코드 | 필수 |
| `REC_CODE` | 레코드 코드 | 필수 |
| `CHOSNO` | 차트번호 | 선택 (빈 값 가능) |
| `IP` | 서버 IP 주소 | 필수 |
| `PORT` | 서버 포트 (기본: 9300) | 필수 |

---

## 3. 호출 패턴 분석

### 3.1 기본 문서 로드 (RecordViewer.jsp)

```javascript
// ActiveX 객체 참조
var ED_OBJ = document.getElementById("edvA");

// 파라미터 문자열 구성
var a = "http://192.168.3.172:9300/EMR_DATA/document/PDF_2293449_1.xml"  // 문서 URL
      + "^"
      + "http://192.168.3.172:9300//EMR_DATA/xml/2014/11_11/2293449.XML"  // 메타데이터 URL
      + "^"
      + "DOC_CODE|2293449#REC_CODE|1370770#CHOSNO|#IP|192.168.3.172#PORT|9300#^";  // 메타데이터

// 문서 로드
ED_OBJ.FV_CommonCall(a);

// 표시 모드 설정
ED_OBJ.SetDrawMode(3);  // 3: 일반 보기/편집 모드
```

### 3.2 EMR 문서 뷰어 (EdViewer.jsp)

```javascript
var ED_OBJ = document.getElementById("edvA");
var EDIp = '<%=EDIp%>';     // 서버 IP (JSP 변수)
var EDPort = '<%=EDPort%>'; // 포트 (기본: 9300)

// 파라미터 동적 구성
var str = EDIp + ":" + EDPort + "/EMR_DATA/document/" + p1  // 문서 파일명
        + "^"
        + EDIp + ":" + EDPort + "/" + p2 + "$tempSave=2"     // 메타데이터 + 임시저장 플래그
        + "^"
        + "DOC_CODE|" + p3
        + "#REC_CODE|" + p4
        + "#CHOSNO|"
        + "#IP|" + EDIp
        + "#PORT|" + EDPort
        + "#^";

ED_OBJ.FV_CommonCall(str);
ED_OBJ.SetDrawMode(3);
```

### 3.3 PDF 뷰어 모듈 (pdfViewerModule.jsp)

```javascript
var sPath = "http://" + serverAddrPdf + "/EMR_DATA/document/" + DocFileName;

// PDF 전용 간소화 호출
ED_OBJ.FV_CommonCall(sPath + "^^");  // 메타데이터 없이 문서만 로드

ED_OBJ.SetDrawMode(100);  // 100: 보기 전용 모드
```

### 3.4 복사/인쇄 뷰 (copyRecdView.jsp)

```javascript
var ED_OBJ = document.getElementById("edvA");
var EDIp = '<%=EDIp%>';
var EDPort = '<%=EDPort%>';

var str = "http://" + EDIp + ":" + EDPort + "/EMR_DATA/document/" + emrwordfilename[0]
        + "^"
        + "http://" + EDIp + ":" + EDPort + "/" + emrwordfilename[2]
        + "^"
        + "DOC_CODE|" + emrwordfilename[1]
        + "#REC_CODE|" + edrecordId[1]
        + "#CHOSNO|"
        + "#IP|" + EDIp
        + "#PORT|" + EDPort
        + "#^";

ED_OBJ.FV_CommonCall(str);
```

---

## 4. 관련 메서드 및 속성

### 4.1 EDViewer OCX 메서드

| 메서드 | 파라미터 | 용도 |
|--------|----------|------|
| `FV_CommonCall(str)` | String | 문서 로드 및 표시 |
| `SetDrawMode(mode)` | Integer | 표시 모드 설정 |
| `PrintDocument(type, p1, p2, copies)` | (2, "", "", 1) | 문서 인쇄 |
| `PageMove(direction)` | Integer (-1, 1) | 페이지 이동 |
| `SetData2Form(paramStr)` | String | 폼 필드 데이터 채우기 |
| `SaveDocument()` | - | 문서 저장 |

### 4.2 SetDrawMode 값

| 값 | 의미 |
|-----|------|
| `3` | 일반 보기/편집 모드 |
| `100` | 보기 전용 모드 (PDF Viewer) |

### 4.3 isCreate 속성

| 값 | 의미 |
|-----|------|
| `0` | 컨트롤 초기화 중 / 준비 안됨 |
| `1` | 컨트롤 준비 완료 |

### 4.4 초기화 패턴

```javascript
function OpenFromServer() {
    ED_OBJ = document.getElementById("edvA");
    if (ED_OBJ == null) return;

    // 컨트롤 초기화 대기
    if (ED_OBJ.isCreate == 0) {
        setTimeout("OpenFromServer()", 500);
        return;
    }

    // 문서 로드
    var sPath = "http://" + serverAddrPdf + "/EMR_DATA/document/" + DocFileName;
    ED_OBJ.FV_CommonCall(sPath + "^^");
    ED_OBJ.SetDrawMode(100);
}
```

---

## 5. 지원 함수

### 5.1 XSS 필터 (eViewCommon.js)

```javascript
function GetParamXSSFilter(name) {
    if (name != null && name != "") {
        name = name.replace("&", "&amp;");
        name = name.replace("\"", "&quot;");
        name = name.replace("\'", "&apos;");
        name = name.replace("/", "&#x2F;");
        name = name.replace("<", "&lt;");
        name = name.replace(">", "&gt;");
        name = name.replace("%", "&#x25;");
    }
    return name;
}
```

### 5.2 서버 주소 설정 (common_ENV.js)

```javascript
var serverAddr = document.domain;
var serverAddrPdf = document.domain + ":9300";
```

### 5.3 JSP 서버 설정 (makeSessionParam.jsp)

```jsp
String EDIp = "http://" + inetA.getHostAddress();
String EDPort = "9300";
```

### 5.4 ActiveX 컨트롤 감지

```javascript
function DetectActiveXControl() {
    ED_OBJ = document.getElementById("edvA");
    if (ED_OBJ == null) {
        alert("ActiveX Control이 정상적으로 설치되지 않았습니다. 재설치하거나 수동설치 하시기 바랍니다.");
    }
}
```

---

## 6. 데이터 흐름

### 6.1 시퀀스 다이어그램

```
┌─────────┐     ┌─────────┐     ┌──────────┐     ┌──────────┐
│ Browser │     │ EDViewer│     │ NPH_ECS  │     │ Database │
│   JSP   │     │   OCX   │     │ Server   │     │          │
└────┬────┘     └────┬────┘     └────┬─────┘     └────┬─────┘
     │               │               │                │
     │ 페이지 로드   │               │                │
     │──────────────>│               │                │
     │               │               │                │
     │ ActiveX 초기화│               │                │
     │ (isCreate=1)  │               │                │
     │               │               │                │
     │ FV_CommonCall │               │                │
     │ (URL 파라미터)│               │                │
     │──────────────>│               │                │
     │               │               │                │
     │               │ HTTP GET      │                │
     │               │ Document URL  │                │
     │               │──────────────>│                │
     │               │               │                │
     │               │               │ DB Query       │
     │               │               │───────────────>│
     │               │               │                │
     │               │               │ Document Data  │
     │               │               │<───────────────│
     │               │               │                │
     │               │ PDF/XML/Word  │                │
     │               │<──────────────│                │
     │               │               │                │
     │ SetDrawMode   │               │                │
     │──────────────>│               │                │
     │               │               │                │
     │               │ 문서 렌더링   │                │
     │               │──────────────>│ (화면 표시)    │
     │               │               │                │
```

### 6.2 URL 경로 구조

```
EMR_DATA/
├── document/              # 문서 파일 저장소
│   ├── PDF_2293449_1.xml  # EMR 문서 (XML 형식)
│   ├── PDF_2293449_1.pdf  # EMR 문서 (PDF 형식)
│   └── ...
│
└── xml/                   # 메타데이터 저장소
    ├── 2014/              # 연도별 디렉터리
    │   └── 11_11/         # 월별 디렉터리
    │       └── 2293449_204314_11423758_1415669202540.XML
    └── ...
```

---

## 7. 보안 고려사항

### 7.1 XSS 방지

`GetParamXSSFilter()` 함수로 파라미터 필터링 수행:
- `&` → `&amp;`
- `"` → `&quot;`
- `'` → `&apos;`
- `<` → `&lt;`
- `>` → `&gt;`
- `%` → `&#x25;`

### 7.2 ActiveX 설치 확인

```javascript
// 컨트롤 감지 및 설치 안내
function DetectActiveXControl() {
    ED_OBJ = document.getElementById("edvA");
    if (ED_OBJ == null) {
        alert("ActiveX Control이 정상적으로 설치되지 않았습니다...");
        return false;
    }
    return true;
}
```

### 7.3 보안 이슈

| 항목 | 내용 | 대응방안 |
|------|------|----------|
| **ActiveX 의존성** | IE 전용, 보안 위험 | HTML5 마이그레이션 권장 |
| **평문 HTTP** | URL 파라미터 평문 전송 | HTTPS 사용 권장 |
| **파라미터 노출** | URL에 문서 ID 노출 | 세션 기반 인증 강화 |

---

## 8. 마이그레이션 가이드

### 8.1 HTML5 대체 방안

```javascript
// 기존: EDViewer OCX 사용
var ED_OBJ = document.getElementById("edvA");
ED_OBJ.FV_CommonCall(documentUrl + "^" + metadataUrl + "^" + params);
ED_OBJ.SetDrawMode(3);

// 대안: PDF.js + HTML5 Canvas
const pdfjsLib = require('pdfjs-dist');

async function loadDocument(documentUrl, metadata) {
    const pdf = await pdfjsLib.getDocument(documentUrl).promise;
    const page = await pdf.getPage(1);
    const canvas = document.getElementById('pdf-canvas');
    const context = canvas.getContext('2d');

    const viewport = page.getViewport({ scale: 1.5 });
    canvas.width = viewport.width;
    canvas.height = viewport.height;

    await page.render({
        canvasContext: context,
        viewport: viewport
    }).promise;

    // 메타데이터 처리
    displayMetadata(metadata);
}
```

### 8.2 API 래핑

```javascript
// 기존 API 호환성 유지를 위한 래퍼
class EMRDocumentViewer {
    constructor(containerId) {
        this.container = document.getElementById(containerId);
        this.mode = 'view';  // 'view' or 'edit'
    }

    // FV_CommonCall 대체 메서드
    loadDocument(documentUrl, metadataUrl, params) {
        // 파라미터 파싱
        const docCode = this.parseParam(params, 'DOC_CODE');
        const recCode = this.parseParam(params, 'REC_CODE');

        // 문서 로드
        return this.fetchDocument(documentUrl)
            .then(doc => this.renderDocument(doc))
            .then(() => this.loadMetadata(metadataUrl));
    }

    // SetDrawMode 대체 메서드
    setMode(mode) {
        this.mode = (mode === 3) ? 'edit' : 'view';
        this.updateUI();
    }

    parseParam(paramStr, key) {
        const regex = new RegExp(key + '\\|([^#]*)');
        const match = paramStr.match(regex);
        return match ? match[1] : null;
    }
}
```

---

## 9. 결론

### FV_CommonCall 핵심 요약

| 항목 | 내용 |
|------|------|
| **타입** | EDViewer OCX 메서드 |
| **파라미터** | 복합 문자열 (URL^URL^Key\|Value#...) |
| **용도** | 의료 문서(PDF/Word/XML) 로드 |
| **의존성** | ActiveX 설치 필수 (IE 전용) |
| **대안** | PDF.js + HTML5 Canvas 마이그레이션 |

### 권장사항

1. **단기**: XSS 필터링 강화, HTTPS 적용
2. **중기**: JavaScript 래퍼 개발로 추상화
3. **장기**: PDF.js 기반 HTML5 뷰어로 전면 교체

---

*FV_CommonCall은 EDViewer의 핵심 인터페이스로, 문서 로드부터 표시까지의 전체 흐름을 제어합니다.*

