# EDViewer(eView) 바이너리 분석

> EDViewer는 완전히 바이너리(OCX/DLL/EXE)로만 제공되며 소스 코드가 없습니다.
> 사실 기반 요약본: `rep.EMR_viewer-EDViewer-팩트체크.md`

---

## 코드베이스 재검증 메모 (N:\99.SourceCode Backup\NPH 기준)

문서 내 의심 지점에 대해 실제 소스/리소스 경로를 재확인한 결과:

- `retrieveEmrData`, `saveEmrSignature` 메서드:
  - 현재 백업 코드셋(`AADEV_NPH/workspace`)에서 **직접 일치 메서드 미확인**.
  - 대체로 확인된 클래스/메서드:
    - `NPH_ECS/src/BKSNP/EMR/ExternalDBPusher.java` -> `CallExternalDB(...)`
    - `NPH_ECS/src/BKSNP/EMR/ExternalDBPusher1.java` -> `CallExternalDB1(...)`
- `NPH_ECS/webapp/jsp/emr/*`, `EMRDataServlet.java`, `emrViewer.jsp`:
  - 현재 백업 코드셋에서 **직접 일치 경로/파일 미확인**.
  - 대체 후보:
    - `NPH_HIS/webapp/eView/EdViewer.jsp`
    - `NPH_HIS/webapp/jsp/md_mobile/emr/EdViewer.jsp`
    - `NPH_HIS/webapp/jsp/md_mobile/emr/emrView.jsp`
- EDViewer 호출 패턴:
  - 문서 예시의 `new ActiveXObject("EDViewer.Control")`는 본 백업셋에서 직접 확인되지 않음.
  - `EdViewer.jsp` 기준으로는 `OBJECT classid=...` + `FV_CommonCall(...)` 패턴이 확인됨.

--- 

## 핵심 결론

| 항목 | 내용 |
|------|------|
| **EDViewer 소스** | ❌ **없음** (C++ 바이너리만 존재) |
| **수정 가능성** | ❌ **불가능** (역공학 없이는) |
| **대안** | 래퍼 개발 또는 완전 교체 필요 |

---

## 1. EDViewer 실제 구성

### 1.1 발견된 바이너리 파일

```
NPH_ECS/webapp/EMR_DATA/eMobile/
├── EDViewer_Ocx.ocx              # 메인 OCX ✅ 정확한 크기로 수정 (8,017,384 bytes)
├── EDViewer_Ocx.cab              # 설치 패키지 ✅ 크기 추가 (2,296,303 bytes)
├── EDViewer_Ocx_test.OCX         # 테스트 버전
├── BKEDViewer.exe                # 실행 파일 ✅ 정확한 크기로 수정 (6,858,728 bytes)
├── BkAgentOcx.ocx                # 에이전트 OCX
├── HookDLL.dll                   # 후킹 DLL ✅ 크기 추가 (94,208 bytes)
├── ISEDViewer(20).apk            # Android 모바일 ✅ 크기 추가 (3,116,017 bytes)
├── ISEDViewer(21).apk            # Android 모바일 ✅ 크기 추가 (3,116,015 bytes)
└── ISEDViewer(NHIS).apk          # 건강보험공단용 ✅ 크기 추가 (3,116,062 bytes)

NPH_HIS/webapp/EMR_DATA/applet/
├── EDViewer_Ocx.ocx              # 복사본 ✅ 정확한 크기로 수정 (8,006,144 bytes)
├── BKEDViewer.exe                # 실행 파일 ✅ 정확한 크기로 수정 (6,775,296 bytes)
├── BkSdePrintModule.ocx          # 출력 모듈 📋 추가발견 (6.8 MB) - 분석필요
├── BkAgentOcx.ocx                # 에이전트 📋 추가발견 (6.7 MB) - 분석필요
├── MagnifyGlass.ocx              # 확대 도구 📋 추가발견 (6.4 MB) - 분석필요
├── bk_capture.ocx                # 캡처 모듈 📋 추가발견 (6.5 MB) - 분석필요
├── TPR.dll / TPR_.ocx            # TPR 관련 📋 추가발견 - 분석필요
├── SignPad.jar                   # 서명 애플릿 📋 추가발견 - 분석필요
├── painter.jar                   # 그리기 도구 📋 추가발견 - 분석필요
└── signedpainter.jar             # 서명된 그리기 도구 📋 추가발견 - 분석필요
```

**범례:**
- ✅ **수정됨**: 2차 작업에서 정확한 크기/정보로 업데이트된 항목
- 📋 **추가발견 - 분석필요**: 소스 분석 중 새로 발견되어 추가 분석이 필요한 항목

### 1.2 파일 상세 정보

| 파일 | 크기 | 타입 | 설명 |
|------|------|------|------|
| `EDViewer_Ocx.ocx` | **7.7 MB** | PE32 DLL (GUI) | 메인 뷰어 OCX |
| `BKEDViewer.exe` | - | PE32 EXE | 독립 실행 뷰어 |
| `EDViewer_Ocx.cab` | - | CAB | 설치 패키지 |

```bash
# EDViewer OCX 파일 정보
$ file EDViewer_Ocx.ocx
PE32 executable (DLL) (GUI) Intel 80386, for MS Windows, 5 sections

# 파일 크기
$ ls -lh EDViewer_Ocx.ocx
-rwxrwxrwx 1 root root 7.7M Mar 28  2023 EDViewer_Ocx.ocx
```

---

## 2. 소스 코드 존재 여부

### 2.1 실제 존재하는 코드

```
✅ 존재하는 코드 (Java)
├── NPH_ECS/src/BKSNP/EMR/
│   ├── ExternalDBPusher.java       # EMR 데이터 연동
│   └── TBL_PROC_ocsapabh.java      # 입원 데이터 처리
│
└── NPH_HIS/webapp/eView/common/
    └── eViewCommon.js              # EDViewer 호출 인터페이스
```

### 2.2 존재하지 않는 코드

```
❌ 존재하지 않음
├── EDViewer_Ocx.ocx 소스 (C++)
├── BKEDViewer.exe 소스
├── BkSdePrintModule.ocx 소스
└── HookDLL.dll 소스
```

**결론**: EDViewer는 완전한 **클로즈드 소스(Closed Source)** 제품입니다.

---

## 3. eViewCommon.js 분석

### 3.1 JavaScript 인터페이스

> 아래 코드는 기능 설명을 위한 대표 예시입니다.
> 현재 백업셋에서 직접 확인된 호출 흔적은 `EdViewer.jsp`의 `OBJECT classid + FV_CommonCall(...)` 패턴이며, `EDViewer.Control` ProgID 문자열은 확인되지 않았습니다.

```javascript
// NPH_HIS/webapp/eView/common/eViewCommon.js
// EDViewer를 호출하는 JavaScript 래퍼

var wardCd='';
var patDate='';
var pid='';
var chosType='';

// XSS 방지 필터
function GetParamXSSFilter(name) {
    if (name != null && name != "") {
        name = name.replace("&", "&amp;");
        name = name.replace("\"", "&quot;");
        name = name.replace("\'", "&apos;");
        // ...
    }
    return name;
}

// 파라미터 설정
function setParameter() {
    pid = document.one.pid.value;
    chosType = document.one.chosType.value;
    doctorId = document.one.doctorId.value;
    // ...
}

// EDViewer 호출
function openEDViewer() {
    setParameter();

    // ActiveX 객체 생성
    var edviewer = new ActiveXObject("EDViewer.Control");

    // 설정
    edviewer.PID = pid;
    edviewer.ChosType = chosType;
    edviewer.DoctorId = doctorId;

    // 문서 로드
    edviewer.LoadDocument(emrDataUrl);
    edviewer.Show();
}
```

### 3.2 JavaScript로 할 수 있는 것

| 가능한 작업 | 설명 |
|-------------|------|
| **파라미터 전달** | EDViewer에 표시할 데이터 지정 |
| **창 크기/위치** | 뷰어 창 설정 |
| **권한 체크** | 쓰기/읽기 권한 전달 |
| **콜백 처리** | 저장/닫기 이벤트 처리 |

| 불가능한 작업 | 설명 |
|-------------|------|
| **렌더링 로직 변경** | 소스 없음 |
| **UI 수정** | OCX 내부 변경 불가 |
| **버그 수정** | 바이너리 패치 필요 |
| **포맷 지원 추가** | OCX 재컴파일 필요 |

---

## 4. 변경 가능성 분석

### 4.1 직접 수정: ❌ 불가능

```
EDViewer_Ocx.ocx (7.7MB 바이너리)
    ↓
[역공학/디컴파일]
    ↓
C++ 소스 (추정) - 복잡하고 어려움
    ↓
[수정]
    ↓
재컴파일 - 거의 불가능 (의존성, 라이선스)
```

**문제점:**
- C++로 개발된 네이티브 코드
- OCX 내부 구역 복잡함 (7.7MB)
- 디컴파일해도 가독성 낮음
- 저작권 문제

### 4.2 간접 수정: ⚠️ 제한적 가능

```
JavaScript 래퍼 (eViewCommon.js)
    ↓ 수정 가능
EDViewer_Ocx.ocx
    ↓ 호출만 가능
내부 로직
```

**가능한 작업:**
1. **JavaScript 래퍼 수정** (eViewCommon.js)
2. **파라미터 전처리** 추가
3. **권한 체크 로직** 강화
4. **후처리** 추가

### 4.3 패치 파일 적용

```
제작사에서 제공하는 패치
├── EDViewer_Ocx.ocx (새 버전)
└── CAB 파일 (자동 설치)

적용 방법:
1. 서버에 새 OCX 파일 배포
2. 클라이언트 CAB 설치
3. 레지스트리 등록
```

---

## 5. 대안 방안

### 5.1 방안 1: JavaScript 래퍼 강화

```javascript
// eViewCommon.js 확장
// 기존 EDViewer 활용하되 주변 로직 강화

function EnhancedEDViewer() {
    // 1. 사전 검증
    if (!validateBeforeOpen()) return;

    // 2. EDViewer 호출
    var edviewer = new ActiveXObject("EDViewer.Control");

    // 3. 추가 보안 설정
    edviewer.EnableSecurity = true;

    // 4. 후처리
    edviewer.OnClose = function() {
        logEmrAccess();
        updateViewCount();
    };

    edviewer.Show();
}
```

**장점:**
- ✅ 기존 EDViewer 재사용
- ✅ 빠른 구현

**단점:**
- ❌ EDViewer 자체 한계는 극복 불가

### 5.2 방안 2: HTML5 EMR 뷰어 개발

```
새로운 EMR 뷰어 (HTML5)
├── Frontend
│   ├── PDF.js (PDF 렌더링)
│   ├── Fabric.js (서명 캔버스)
│   └── React/Vue.js (UI)
│
├── Backend
│   ├── Spring Boot (NPH 연동)
│   └── EMR 데이터 변환 서비스
│
└── 전자서명
    └── REST API 기반 (SignGate 대체)
```

**장점:**
- ✅ 현대 브라우저 지원
- ✅ 유지보수 용이
- ✅ 모바일 지원

**단점:**
- ❌ 높은 개발 비용
- ❌ 기존 데이터 마이그레이션 필요

### 5.3 방안 3: 상용 제품 도입

| 제품 | 특징 | 비용 |
|------|------|------|
| **Oracle Healthcare** | 종합 EMR | 고비용 |
| **InterSystems HealthShare** | 통합 플랫폼 | 고비용 |
| **Cerner PowerChart** | 대형 병원용 | 고비용 |
| **Samsung SDS EMR** | 국내 지원 | 중간 |

---

## 6. 현재 소스로 가능한 작업

### 6.1 가능한 변경

```
✅ 가능한 작업
├── NPH_ECS/src/BKSNP/EMR/*.java
│   └── EMR 데이터 연동 로직 수정
│
├── NPH_HIS/webapp/eView/common/eViewCommon.js
│   └── EDViewer 호출 파라미터 수정
│
└── NPH_HIS/src/nph/his/*/cmd/*.java
    └── EMR 관련 Command 수정
```

### 6.2 불가능한 작업

```
❌ 불가능한 작업
├── EDViewer_Ocx.ocx 수정
├── EDViewer UI 변경
├── EDViewer 버그 수정
└── EDViewer 새 기능 추가
```

---

## 7. 결론

### EDViewer 현황 요약

| 구분 | 상태 |
|------|------|
| **소스 코드** | ❌ 없음 (C++ 바이너리) |
| **수정 가능성** | ❌ 없음 (역공engineering 어려움) |
| **JavaScript 인터페이스** | ✅ 있음 (eViewCommon.js) |
| **패치** | ⚠️ 제작사 제공 패치만 적용 가능 |

### 권장 대안

```
단기: JavaScript 래퍼 개선
    └── eViewCommon.js 확장
        ├── 추가 검증 로직
        └── 로깅/감사 기능

중기: HTML5 EMR 뷰어 개발
    └── PDF.js + Fabric.js 기반
        ├── 점진적 마이그레이션
        └── 기존 EDViewer 병행 사용

장기: 상용 EMR 뷰어 도입
    └── 종합적인 시스템 전환
```

### 핵심 메시지

> **EDViewer는 완전한 바이너리 제품으로 소스 코드가 없어 직접 수정이 불가능합니다.**

**가능한 것:**
- JavaScript 인터페이스(eViewCommon.js) 수정
- EMR 데이터 연동 로직 수정
- EDViewer 호출 전/후 처리 추가

**불가능한 것:**
- EDViewer 자체의 렌더링/UI 수정
- 버그 수정
- 새 기능 추가

**결론:**
EDViewer의 한계를 극복하려면 **JavaScript 래퍼로 우회**하거나 **완전히 새로운 뷰어 개발**이 필요합니다.

---

*EDViewer는 C++로 개발된 3rd-party 바이너리 컴포넌트로 소스 코드 제공이 없습니다.*


