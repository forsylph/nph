# EMR 뷰어 EDViewer(eView) 분석

> 국립의료원 병원정보시스템(NPH)의 전자의무기록(EMR) 뷰어 분석

---

## 개요

### EDViewer란?

**EDViewer** (또는 **eView**)는 국립의료원 병원정보시스템(NPH)에서 사용하는 **EMR(Electronic Medical Record, 전자의무기록) 뷰어**입니다.

| 항목 | 내용 |
|------|------|
| **제품명** | EDViewer / eView |
| **타입** | ActiveX / OCX 컴포넌트 |
| **사용 위치** | NPH_ECS (전자동의시스템), NPH_HIS |
| **기술** | Windows 전용 ActiveX, Internet Explorer 필수 |
| **목적** | EMR 문서 조회, 서명, 출력 |

---

## 1. 아키텍처

### 1.1 EDViewer 시스템 구성

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client (Windows)                        │
├─────────────────────────────────────────────────────────────────┤
│  Internet Explorer                                              │
│    │                                                           │
│    ├── ActiveX: EDViewer.ocx                                   │
│    │   ├── EMR 문서 렌더링                                     │
│    │   ├── 인쇄 기능                                           │
│    │   └── 서명 캡처                                           │
│    │                                                           │
│    └── ActiveX: SignGate (전자서명)                            │
│        └── 공인인증서 기반 전자서명                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ HTTP / HTTPS
┌─────────────────────────────────────────────────────────────────┐
│                      Server (NPH_ECS)                          │
├─────────────────────────────────────────────────────────────────┤
│  Servlet Layer                                                  │
│    ├── EMRDataServlet.java                                      │
│    └── BK_ajax.java                                             │
│                                                                 │
│  EMR 데이터 연동                                                │
│    ├── ExternalDBPusher.java                                    │
│    ├── TBL_PROC_ocsapabh.java                                   │
│    └── BKSNP.EMR.* 패키지                                       │
│                                                                 │
│  데이터 저장                                                    │
│    └── webapp/EMR_DATA/                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 NPH에서의 EMR 시스템 분리

```
NPH 시스템 구조
├── NPH_HIS (병원정보시스템)
│   ├── DevOn + MiPlatform (메인)
│   ├── EMR 조회 기능 (EDViewer 호출)
│   └── webapp/jsp/md_mobile/emr/ (모바일 EMR)
│
└── NPH_ECS (전자동의시스템)
    ├── MiPlatform 미사용
    ├── EDViewer (ActiveX) 사용
    ├── SignGate (전자서명) 연동
    └── EMR 데이터 연동 및 저장
```

---

## 2. EDViewer 기술적 특징

### 2.1 ActiveX/OCX 기반

```
┌────────────────────────────────────────────────────────────────┐
│                   EDViewer ActiveX 구조                        │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  EDViewer.ocx                                                   │
│  ├── 렌더링 엔진                                                │
│  │   ├── XML 기반 EMR 데이터 파싱                              │
│  │   ├── HTML/CSS 변환                                         │
│  │   └── 화면 렌더링                                           │
│  │                                                              │
│  ├── 기능 모듈                                                  │
│  │   ├── 페이지 탐색 (이전/다음)                                │
│  │   ├── 확대/축소                                              │
│  │   ├── 인쇄                                                   │
│  │   └── 저장 (PDF/이미지)                                      │
│  │                                                              │
│  └── 서명 모듈                                                  │
│      └── 서명 영역 캡처                                         │
│          └── SignGate 연계                                      │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### 2.2 기술적 특징

| 특징 | 설명 |
|------|------|
| **Windows 전용** | ActiveX이므로 Windows + IE 필수 |
| **플러그인 설치** | OCX 파일 설치 필요 |
| **보안 설정** | IE 보안 설정에서 ActiveX 허용 필요 |
| **브라우저 제한** | Internet Explorer 전용 (Chrome/Firefox 불가) |
| **64비트 제한** | 32비트 IE에서만 동작 가능한 경우 다수 |

---

## 3. EDViewer 기능

### 3.1 주요 기능

```
EDViewer 기능 목록
├── 문서 조회
│   ├── EMR 문서 로드
│   ├── 페이지 단위 탐색
│   ├── 섹션 이동 (처방, 검사, 경과 등)
│   └── 히스토리 조회 (이전 기록)
│
├── 문서 출력
│   ├── 인쇄 (프린터)
│   ├── PDF 저장
│   └── 이미지 저장 (PNG/JPG)
│
├── 서명 기능
│   ├── 서명 영역 표시
│   ├── 서명 캡처 (터치/마우스)
│   └── SignGate 연동 (법적 효력)
│
└── 보안 기능
    ├── 문서 암호화 (저장 시)
    ├── 접근 권한 체크
    └── 로깅/감사추적
```

### 3.2 EMR 문서 형식

```
EMR 데이터 구조
├── Header (머리글)
│   ├── 환자 기본 정보
│   │   ├── 이름, 생년월일, 성별
│   │   ├── 환자번호 (PID)
│   │   └── 차트번호
│   │
│   └── 진료 정보
│       ├── 진료일자
│       ├── 진료과
│       ├── 주치의
│       └── 입/외래 구분
│
├── Body (본문)
│   ├── 진료기록지
│   │   ├── 주호소 (CC)
│   │   ├── 현병력 (HPI)
│   │   ├── 과거력 (PMH)
│   │   ├── 검사 소견
│   │   └── 진단 및 처방
│   │
│   ├── 검사 결과
│   │   ├── 혈액검사
│   │   ├── 영상검사 (X-ray, CT, MRI)
│   │   └── 기타 검사
│   │
│   └── 처방 전문
│       ├── 약물 처방
│       ├── 주사 처방
│       └── 처방 실행
│
└── Footer (꼬리글)
    ├── 작성자 정보
    ├── 전자서명 영역
    └── 문서 ID
```

---

## 4. 시스템 연동

### 4.1 NPH_HIS와의 연동

```java
// NPH_HIS에서 EDViewer 호출 (JSP)
// webapp/jsp/md/opn/emrView.jsp

<%@ page language="java" contentType="text/html; charset=EUC-KR" %>
<html>
<head>
    <script>
        function openEmrViewer(pid, visitDate) {
            // EDViewer ActiveX 객체 생성
            var edviewer = new ActiveXObject("EDViewer.Control");

            // EMR 데이터 로드
            var emrDataUrl = "/ecs/emrNavi/RetrieveEmrData.mhi" +
                            "?pid=" + pid +
                            "&visitDate=" + visitDate;

            edviewer.LoadDocument(emrDataUrl);
            edviewer.Show();
        }
    </script>
</head>
<body>
    <button onclick="openEmrViewer('P12345', '20240305')">
        EMR 조회
    </button>

    <!-- EDViewer 컨테이너 -->
    <div id="edviewer-container" style="width:100%; height:600px;">
    </div>
</body>
</html>
```

### 4.2 NPH_ECS와의 연동

```java
// NPH_ECS - EMR 데이터 제공
// src/BKSNP/EMR/ExternalDBPusher.java

public class ExternalDBPusher {

    /**
     * 외부 EMR 데이터 조회
     */
    public String retrieveEmrData(String pid, String visitDate) {
        // HIS DB 연결
        Connection conn = getHISConnection();

        // EMR 데이터 조회
        String emrXml = queryEmrData(conn, pid, visitDate);

        // EDViewer 형식으로 변환
        String edviewerFormat = convertToEdviewerFormat(emrXml);

        return edviewerFormat;
    }

    /**
     * EMR 서명 데이터 저장
     */
    public boolean saveEmrSignature(String pid, String docId,
                                     byte[] signatureData) {
        // 전자서명 검증
        if (!verifySignature(signatureData)) {
            return false;
        }

        // 서명 데이터 저장
        return saveSignatureToDB(pid, docId, signatureData);
    }
}
```

### 4.3 데이터 흐름

```
EMR 조회 흐름
├── 1. 의사가 HIS에서 EMR 조회 클릭
│   └── NPH_HIS (MiPlatform 화면)
│
├── 2. EDViewer 팝업 또는 페이지 이동
│   └── IE + EDViewer ActiveX 로드
│
├── 3. EMR 데이터 요청
│   └── NPH_ECS로 데이터 요청
│       ├── ExternalDBPusher.retrieveEmrData()
│       └── HIS DB에서 데이터 조회
│
├── 4. 데이터 변환
│   └── EDViewer 형식(XML/HTML)으로 변환
│
├── 5. EDViewer 렌더링
│   └── 문서 화면에 표시
│
└── 6. 사용자 상호작용
    ├── 스크롤/페이지 이동
    ├── 확대/축소
    ├── 인쇄
    └── 서명 (필요시)
```

---

## 5. 전자서명(SignGate) 연동

### 5.1 SignGate 개요

| 항목 | 내용 |
|------|------|
| **제품** | SignGate (한국정보인증 또는 유사 제품) |
| **기술** | ActiveX 기반 공인인증서 |
| **목적** | EMR 문서 전자서명 (법적 효력) |
| **규격** | 전자서명법, 의료법 준수 |

### 5.2 서명 프로세스

```
┌────────────────────────────────────────────────────────────────┐
│                   전자서명 프로세스                            │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. EMR 작성 완료                                               │
│     └── 의사가 진료기록 작성                                    │
│                                                                 │
│  2. 서명 요청                                                   │
│     └── EDViewer에서 "서명" 버튼 클릭                           │
│                                                                 │
│  3. SignGate ActiveX 실행                                       │
│     └── 공인인증서 선택창 표시                                  │
│     └── 의사가 본인 인증서 선택                                 │
│                                                                 │
│  4. 인증서 검증                                                 │
│     └── 공인인증서 유효성 검증                                  │
│     └── 비밀번호 입력                                           │
│                                                                 │
│  5. 문서 해싱 및 서명                                           │
│     └── EMR 문서 해시값 생성 (SHA-256)                          │
│     └── 개인키로 서명 생성                                      │
│                                                                 │
│  6. 서명 데이터 저장                                            │
│     └── 서명값 + 인증서 + 타임스탬프                            │
│     └── NPH_ECS DB 저장                                         │
│                                                                 │
│  7. EMR 문서 완료 상태 변경                                     │
│     └── "서명 완료" 상태로 변경                                 │
│     └── 수정 불가 (법적 보존)                                   │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### 5.3 서명 데이터 구조

```java
// 전자서명 데이터 모델
public class EmrSignature {
    private String pid;                    // 환자번호
    private String documentId;             // 문서ID
    private String doctorId;               // 의사ID
    private byte[] signedData;             // 서명값
    private X509Certificate certificate;   // 공인인증서
    private Date signedTime;               // 서명 시간
    private String timestamp;              // 타임스탬프
    private String hashAlgorithm;          // 해시 알고리즘 (SHA-256)
}
```

---

## 6. 문제점과 한계

### 6.1 기술적 문제

| 문제 | 설명 |
|------|------|
| **ActiveX 의존성** | Windows + IE 전용, Chrome/Edge 불가 |
| **보안 취약점** | ActiveX는 보안 위험으로 차단되는 경우 많음 |
| **설치 복잡성** | OCX 파일 수동 설치 필요 |
| **64비트 문제** | 32비트 IE에서만 동작 |
| **업데이트 어려움** | 클라이언트 전체 업데이트 필요 |

### 6.2 현대 브라우저 호환성 문제

```
브라우저 지원 현황
├── Internet Explorer 11
│   └── ✅ 지원 (Windows 10)
│
├── Microsoft Edge (레거시)
│   └── ⚠️ ActiveX 모드 필요
│
├── Microsoft Edge (Chromium)
│   └── ❌ ActiveX 미지원
│
├── Google Chrome
│   └── ❌ ActiveX 미지원
│
└── Mozilla Firefox
    └── ❌ ActiveX 미지원
```

### 6.3 보안 정책 충돌

```
문제 상황
├── 기업 보안 정책
│   └── ActiveX 차단
│       └── 그룹 정책(GPO)으로 비활성화
│
├── 브라우저 보안
│   └── IE 보안 설정
│       └── 인터넷/인트라넷 영역 설정 필요
│
└── 백신/보안 소프트웨어
    └── ActiveX 설치 차단
        └── OCX 파일 삭제/차단
```

---

## 7. 마이그레이션 필요성

### 7.1 현대화 방안

```
EDViewer → 현대적 EMR 뷰어
├── 방안 1: HTML5 기반 뷰어
│   ├── PDF.js (PDF 렌더링)
│   ├── Fabric.js (서명 캔버스)
│   └── HTML5 Canvas (이미지 처리)
│
├── 방안 2: 웹 기반 뷰어
│   ├── Spring Boot + Thymeleaf
│   ├── React/Vue.js 프론트엔드
│   └── REST API 연동
│
└── 방안 3: 표준 EMR 포맷
    ├── HL7 FHIR (의료 데이터 표준)
    ├── CDA (Clinical Document Architecture)
    └── PDF/A (장기 보존 형식)
```

### 7.2 전자서명 현대화

```
SignGate ActiveX → 현대적 전자서명
├── 공인전자서명 (KISA)
│   └── REST API 기반 서명
│
├── 전자문서유통 표준 (ODF)
│   └── XMLDsig, XAdES
│
└── 블록체인 기반
    └── 서명 이력 불변성 보장
```

---

## 8. NPH 프로젝트에서의 EDViewer 위치

### 8.1 파일 위치

```
NPH 프로젝트
├── NPH_ECS/
│   ├── src/BKSNP/EMR/
│   │   ├── ExternalDBPusher.java         # EMR 데이터 연동
│   │   ├── TBL_PROC_ocsapabh.java      # 입원 환자 데이터 처리
│   │   └── BK_ajax.java                # EMR AJAX 처리
│   │
│   └── webapp/
│       ├── EMR_DATA/                   # EMR 데이터 저장
│       └── jsp/
│           └── emr/
│               ├── edviewer.jsp        # EDViewer 호출 페이지
│               └── emrViewer.jsp       # EMR 조회
│
└── NPH_HIS/
    └── webapp/
        └── jsp/
            ├── md_mobile/emr/          # 모바일 EMR
            └── md/opn/
                └── emrView.jsp         # EMR 뷰어 연동
```

### 8.2 데이터 흐름

```
┌─────────────────────────────────────────────────────────────────┐
│                      EDViewer 데이터 흐름                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  OCS (Order Communication System)                               │
│    │                                                           │
│    ▼                                                           │
│  HIS DB (Oracle/Tibero)                                        │
│    │                                                           │
│    ▼                                                           │
│  NPH_ECS                                                        │
│    ├── ExternalDBPusher.retrieveEmrData()                     │
│    └── EMR XML 생성                                             │
│         │                                                      │
│         ▼                                                      │
│    EDViewer ActiveX                                             │
│         │                                                      │
│         ▼                                                      │
│    의사 화면 (조회/서명)                                        │
│         │                                                      │
│         ▼                                                      │
│    SignGate (서명 시)                                           │
│         │                                                      │
│         ▼                                                      │
│    NPH_ECS DB (서명 저장)                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. 결론

### EDViewer 현황

| 구분 | 내용 |
|------|------|
| **기술** | ActiveX/OCX (레거시) |
| **플랫폼** | Windows + Internet Explorer |
| **의존성** | SignGate (전자서명) |
| **현재 상태** | 기능 동작 중이나 현대화 필요 |
| **미래** | HTML5 기반 마이그레이션 권장 |

### 핵심 메시지

> **EDViewer는 ActiveX 기반의 레거시 EMR 뷰어로, 현대 브라우저와 호환되지 않아 마이그레이션이 필요합니다.**

**NPH 프로젝트에서 EDViewer는:**
1. NPH_ECS(전자동의시스템)에서 주로 사용
2. NPH_HIS에서도 EMR 조회 시 호출
3. SignGate와 연계하여 법적 효력 있는 전자서명 제공
4. ActiveX 의존성으로 인해 현대화가 시급

**권장 방향:**
- HTML5 기반 EMR 뷰어로 마이그레이션
- 전자서명도 REST API 기반으로 현대화
- HL7 FHIR 등 표준 기반 데이터 교환

---

*EDViewer는 과거 기술(ActiveX)에 기반한 EMR 뷰어로, 현재 의료 시스템의 모던화 추세에 맞추어 개선이 필요합니다.*
