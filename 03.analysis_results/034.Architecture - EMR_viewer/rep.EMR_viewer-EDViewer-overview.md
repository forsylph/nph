# EMR 뷰어 EDViewer(eView) 분석

> 국립의료원 병원정보시스템(NPH)의 전자의무기록(EMR) 뷰어 분석
> 사실 기반 요약본: `rep.EMR_viewer-EDViewer-팩트체크.md`

---

## 코드베이스 재검증 메모 (N:\99.SourceCode Backup\NPH 기준)

아래 항목은 실제 코드/파일 검색으로 재확인한 결과입니다.

### 1) 질문 항목 추적 결과

- `retrieveEmrData`, `saveEmrSignature`
  - 현재 백업 코드셋(`AADEV_NPH/workspace`)에서는 **직접 일치 메서드 미확인**.
  - 유사/대체 후보:
    - `NPH_ECS/src/BKSNP/EMR/ExternalDBPusher.java` -> `CallExternalDB(...)`
    - `NPH_ECS/src/BKSNP/EMR/ExternalDBPusher1.java` -> `CallExternalDB1(...)`
- `NPH_ECS/webapp/jsp/emr/*`
  - 현재 백업 코드셋에서 **해당 디렉터리 미확인**.
- `EMRDataServlet.java`
  - 현재 백업 코드셋에서 **파일 미확인**.
  - 대신 `NPH_ECS/src` 하위에 EMR 저장/삭제 관련 Java 진입점(`SaveRecord`, `DeleteRecord`, `save_xml` 등) 다수 확인.
- `emrViewer.jsp`
  - 현재 백업 코드셋에서 **정확한 파일명 미확인**.
  - 대체 후보로 `EdViewer.jsp` 확인:
    - `NPH_HIS/webapp/eView/EdViewer.jsp`
    - `NPH_HIS/webapp/jsp/md_mobile/emr/EdViewer.jsp`

### 2) 경로/호출 관련 보정 포인트

- `NPH_HIS/webapp/jsp/md/opn/emrView.jsp`로 표기된 경로는 본 백업셋에서 확인되지 않음.
- 확인된 EMR 조회 JSP:
  - `NPH_HIS/webapp/jsp/md_mobile/emr/emrView.jsp`
- EDViewer ActiveX 연결은 문서 예시의 `new ActiveXObject("EDViewer.Control")` 패턴보다,
  `OBJECT classid=...` + `FV_CommonCall(...)` 사용 흔적이 명확함 (`EdViewer.jsp` 기준).
- 실제 배포 바이너리 위치:
  - `NPH_HIS/webapp/EMR_DATA/applet/EDViewer_Ocx.ocx`
  - `NPH_ECS/webapp/EMR_DATA/eMobile/EDViewer_Ocx.ocx`
  - `NPH_ECS/webapp/EMR_DATA/eMobile/BKEDViewer.exe`

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

> 주의: 본 문서의 일부 메서드명/호출 예시는 기능 설명용이며, 현재 백업 코드셋에서 직접 확인된 구현은 `EdViewer.jsp`, `emrView.jsp`, `ExternalDBPusher*`, EMR_DATA 바이너리 배포본 기준입니다.

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
│    ├── BK_ajax.java                                             │
│    └── @WebServlet 계열 EMR 처리 클래스                         │
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
// 확인된 경로:
// - NPH_HIS/webapp/eView/EdViewer.jsp
// - NPH_HIS/webapp/jsp/md_mobile/emr/EdViewer.jsp
//
// 실제 호출 패턴(요약): OBJECT classid + 메서드 호출

<OBJECT ID="edvA"
        classid="clsid:879DF37E-E2E1-4C52-979D-60E0806C6E97"
        width="100%"
        height="100%">
</OBJECT>

<script>
  var ED_OBJ = document.getElementById("edvA");
  // EDViewer 데이터 전달 (실소스: FV_CommonCall 사용)
  ED_OBJ.FV_CommonCall(payloadString);
</script>
```

### 4.2 NPH_ECS와의 연동

```java
// NPH_ECS - 확인된 클래스
// src/BKSNP/EMR/ExternalDBPusher.java

public class ExternalDBPusher {
    public void CallExternalDB(ArrayList Al1) throws IOException { ... }
    public void MakeLog(ArrayList Al1) throws IOException { ... }
}

// 참고:
// retrieveEmrData / saveEmrSignature 메서드는
// 현재 백업 코드셋에서 직접 확인되지 않음.
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
│       ├── AjaxXmlDirectConnection 요청 (NPH_HIS)
│       └── NPH_ECS @WebServlet 계열/EMR_DATA 리소스 처리
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
│       └── (참고) jsp/emr 경로는 본 백업셋에서 미확인
│
└── NPH_HIS/
    └── webapp/
        ├── eView/EdViewer.jsp
        └── jsp/md_mobile/emr/
            ├── EdViewer.jsp
            └── emrView.jsp
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
│    ├── AjaxXmlDirectConnection/BK_ajax 처리                    │
│    └── EMR XML/문서 데이터 생성                                 │
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

## 9. 추가 발견 컴포넌트

> 소스코드 분석 과정에서 추가로 발견된 컴포넌트들입니다.
>
> **범례:**
> - ✅ **수정됨**: 2차 작업에서 정확한 크기/정보로 업데이트된 항목
> - 📋 **추가발견 - 분석필요**: 새로 발견되어 추가 분석이 필요한 항목

### 9.1 미문서화된 Java 클래스 📋

**경로:** `NPH_ECS/src/BKSNP/EMR/`

| 파일 | 용도 (추정) | 비고 |
|------|------------|------|
| `TMSG.java` 📋 | 메시지 처리 유틸리티 | EMR 메시지 전송 관련 - 분석필요 |
| `BK_logMsg.java` 📋 | 로깅 메시지 클래스 | EMR 로그 기록 - 분석필요 |
| `BK_util.java` 📋 | 공통 유틸리티 함수 | 문자열/데이터 변환 - 분석필요 |
| `DB_CONN.java` 📋 | 데이터베이스 연결 클래스 | EMR DB 접근 - 분석필요 |
| `BK_dbConnection.java` 📋 | 대체 DB 연결 클래스 | 연결 풀 관리 - 분석필요 |

**분석 필요 사항:**
- 각 클래스의 정확한 역할 파악
- `ExternalDBPusher.java`와의 연관성 분석
- `BK_ajax.java`와의 호출 관계 매핑

### 9.2 미문서화된 바이너리 컴포넌트

**경로:** `NPH_HIS/webapp/EMR_DATA/applet/` 및 `NPH_ECS/webapp/EMR_DATA/eMobile/`

| 파일 | 크기 | 용도 | 상태 |
|------|------|------|------|
| `BkSdePrintModule.ocx` 📋 | 6.8 MB | 인쇄 모듈 | 분석필요 |
| `BkAgentOcx.ocx` 📋 | 6.7 MB | 에이전트 컨트롤 | 분석필요 |
| `MagnifyGlass.ocx` 📋 | 6.4 MB | 확대/돋보기 도구 | 분석필요 |
| `bk_capture.ocx` 📋 | 6.5 MB | 이미지 캡처 | 분석필요 |
| `TPR.dll` 📋 | - | TPR 관련 컴포넌트 | 분석필요 |
| `TPR_.ocx` 📋 | - | TPR OCX | 분석필요 |
| `SignPad.jar` 📋 | - | 서명 패드 Java 애플릿 | 분석필요 |
| `painter.jar` 📋 | - | 화면 그리기 도구 | 분석필요 |
| `signedpainter.jar` 📋 | - | 서명된 그리기 도구 | 분석필요 |

**정확한 파일 크기 (수정됨):** ✅

| 파일 | 위치 | 크기 (Bytes) | 크기 (MB) |
|------|------|-------------|----------|
| `EDViewer_Ocx.ocx` ✅ | NPH_HIS/webapp/EMR_DATA/applet/ | 8,006,144 | 7.64 MB |
| `EDViewer_Ocx.ocx` ✅ | NPH_ECS/webapp/EMR_DATA/eMobile/ | 8,017,384 | 7.65 MB |
| `BKEDViewer.exe` ✅ | NPH_HIS/webapp/EMR_DATA/applet/ | 6,775,296 | 6.45 MB |
| `BKEDViewer.exe` ✅ | NPH_ECS/webapp/EMR_DATA/eMobile/ | 6,858,728 | 6.53 MB |

**분석 필요 사항:**
- 각 OCX/DLL의 정확한 기능 검증
- `BkSdePrintModule.ocx`와 EDViewer의 연동 방식 분석
- `SignPad.jar`와 SignGate의 관계 파악

### 9.3 모바일 EMR 아키텍처

**경로:** `NPH_HIS/webapp/jsp/md_mobile/emr/`

| 파일 | 용도 |
|------|------|
| `EdViewer.jsp` | 모바일용 EDViewer 진입점 |
| `emrView.jsp` | 모바일 EMR 뷰어 |

**Android APK 파일 (크기 수정됨):** ✅

| 파일 | 크기 | 용도 |
|------|------|------|
| `ISEDViewer(20).apk` ✅ | 3,116,017 bytes (2.97 MB) | Android API 20용 |
| `ISEDViewer(21).apk` ✅ | 3,116,015 bytes (2.97 MB) | Android API 21용 |
| `ISEDViewer(NHIS).apk` ✅ | 3,116,062 bytes (2.97 MB) | 건강보험공단 전용 |

**모바일 EMR 데이터 흐름:**
```
모바일 앱 (ISEDViewer)
    │
    ▼
NPH_ECS/webapp/EMR_DATA/eMobile/
    │
    ▼
EdViewer.jsp (모바일)
    │
    ▼
AjaxXmlDirectConnection
    │
    ▼
NPH_ECS Backend
    │
    ▼
Database
```

**분석 필요 사항:** 📋
- 모바일 APK 기능 분석 (APK 디컴파일 필요)
- `md_mobile` 모듈 전체 구조 파악
- 모바일-데스크톱 EMR 데이터 동기화 방식 분석

### 9.4 SignGate 통합 상세

**경로:** `NPH_ECS/webapp/signgate/` 📋

**디렉터리 구조:**
```
signgate/
├── cert/           # 인증서 저장소
├── js/             # JavaScript 모듈
├── css/            # 스타일시트
├── images/         # UI 이미지
├── sample.jsp      # 연동 샘플
└── serverCert.jsp  # 서버 인증서 처리
```

**SignGate 컴포넌트:**
| 항목 | 설명 |
|------|------|
| `cert/` 📋 | 공인인증서 저장/관리 - 분석필요 |
| `js/` 📋 | 클라이언트 JavaScript 라이브러리 - 분석필요 |
| `sample.jsp` 📋 | 연동 샘플 코드 (참고용) |
| `serverCert.jsp` 📋 | 서버 인증서 검증 - 분석필요 |

**SignGate 데이터 흐름:**
```
EDViewer (서명 요청)
    │
    ▼
signgate/ (JavaScript 호출)
    │
    ▼
cert/ (인증서 선택)
    │
    ▼
공인인증서 검증 (KISA/인증기관)
    │
    ▼
서명값 생성
    │
    ▼
NPH_ECS DB 저장
```

**분석 필요 사항:** 📋
- `signgate/js/` JavaScript 모듈 분석
- EDViewer와 SignGate 간 호출 인터페이스 파악
- 인증서 저장 방식 및 보안 검토

### 9.5 EMR_DATA JSP 파일 (레코드 운영) 📋

**경로:** `NPH_HIS/webapp/EMR_DATA/` 및 `NPH_ECS/webapp/EMR_DATA/`

| 파일 | 용도 |
|------|------|
| `SaveRecord.jsp` 📋 | 개별 레코드 저장 - 분석필요 |
| `SaveRecord_All.jsp` 📋 | 전체 레코드 일괄 저장 - 분석필요 |
| `Save_XML_File.jsp` 📋 | XML 형식 파일 저장 - 분석필요 |
| `Save_File.jsp` 📋 | 일반 파일 저장 - 분석필요 |
| `GetSignImg.jsp` 📋 | 서명 이미지 조회 - 분석필요 |
| `RunPrint.jsp` 📋 | 인쇄 실행 - 분석필요 |

**분석 필요 사항:**
- 각 JSP 파일의 정확한 기능 및 파라미터 분석
- 저장 프로세스 데이터 흐름 검증
- 보안 취약점 점검 (XSS, 파일 업로드 등)

### 9.6 추가 JSP 파일 (FV_CommonCall 사용) 📋

**경로:** `NPH_HIS/webapp/eView/popup/`

| 파일 | 용도 |
|------|------|
| `copyRecdView.jsp` 📋 | 레코드 복사 보기 - 분석필요 |
| `directRecdPrinter.jsp` 📋 | 직접 인쇄 - 분석필요 |
| `HidepdfViewerModule.jsp` 📋 | PDF 뷰어 (숨김) - 분석필요 |
| `pdfViewerModule.jsp` 📋 | PDF 뷰어 모듈 - 분석필요 |
| `RecordViewer.jsp` 📋 | 레코드 뷰어 - 분석필요 |

**분석 필요 사항:**
- `FV_CommonCall` 호출 패턴 상세 분석
- 팝업 JSP와 EDViewer 간 데이터 전달 방식

---

## 10. 결론

### EDViewer 현황

| 구분 | 내용 |
|------|------|
| **기술** | ActiveX/OCX (레거시) |
| **플랫폼** | Windows + Internet Explorer |
| **의존성** | SignGate (전자서명) |
| **현재 상태** | 기능 동작 중이나 현대화 필요 |
| **미래** | HTML5 기반 마이그레이션 권장 |

### 2차 작업 수정 내용 요약 ✅

| 항목 | 수정 내용 |
|------|----------|
| **바이너리 파일 크기** | 정확한 바이트 단위 크기로 업데이트 (EDViewer_Ocx.ocx: 8,006,144 / 8,017,384 bytes) |
| **APK 파일 크기** | 정확한 크기 추가 (ISEDViewer: ~3.1MB) |
| **Java 클래스** | 5개 미문서화 클래스 추가 발견 (TMSG.java 등) |
| **바이너리 컴포넌트** | 9개 미문서화 OCX/DLL/JAR 추가 발견 |
| **SignGate 경로** | `NPH_ECS/webapp/signgate/` 디렉터리 구조 추가 |
| **EMR_DATA JSP** | 6개 JSP 파일 추가 발견 (SaveRecord.jsp 등) |
| **eView 팝업 JSP** | 5개 팝업 JSP 파일 추가 발견 |

### 추가 분석 필요 항목 📋

| 구분 | 항목 | 우선순위 |
|------|------|----------|
| **Java 클래스** | TMSG.java, BK_logMsg.java, BK_util.java, DB_CONN.java, BK_dbConnection.java 역할 분석 | 중간 |
| **바이너리** | BkSdePrintModule.ocx, MagnifyGlass.ocx, bk_capture.ocx 기능 검증 | 높음 |
| **모바일** | ISEDViewer APK 디컴파일 및 기능 분석 | 높음 |
| **SignGate** | JavaScript 모듈 및 인증서 저장 방식 분석 | 중간 |
| **EMR_DATA JSP** | 저장 프로세스 및 보안 취약점 점검 | 높음 |
| **FV_CommonCall** | 호출 패턴 및 데이터 전달 방식 분석 | 중간 |

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
