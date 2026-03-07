# MiPlatform 전체 구조 트리

> 이 문서는 `031.front-channel` 기준본을 보조하는 **reference 문서**다.
> 목적은 MiPlatform 화면군의 큰 분포와 파일군 규모를 빠르게 훑는 것이며, 실제 유지보수 추적은 `Miplatform.md`, `MiPlatform-Transaction-패턴.md`, `화면XML-script-mhi-연결.md`를 먼저 보는 편이 빠르다.
> 공식 MiPlatform 3.3 매뉴얼의 `Form = Design + Data + Event` 관점을 적용하면, 아래 트리는 `화면 수/분포`를 보는 지도이고 실제 추적은 개별 XML 안의 Dataset, Event, Transaction을 따라가는 방식으로 읽는 것이 맞다.

> NPH 프로젝트 MiPlatform 화면 및 구성요소 섹션별 트리 구조
> 주의: 일부 Transaction/Command/PC 이름은 흐름 설명을 위한 예시 표기이며, 실제 운영명과 1:1 일치하지 않을 수 있습니다.

> 재검증 범위: 이 문서의 통계/비율/대표 예시명은 reference 성격으로 유지한다. 실제 유지보수 추적에 직접 쓰는 값은 Login3.xml, MD_ORD01001P.xml, HP_DMS01303M.xml, HP_DMS02204M.xml, AZ_UTL01002P.xml 및 연결된 .mhi / navigation / command를 우선한다.

---

## 1. 전체 통계 개요

### 1.1 프로젝트 규모 (SVN 캐시 제외)

> **참고**: SVN 캐시(.svn-base) 28,358개 제외한 실제 프로젝트 파일 기준

| 항목 | 값 | 비율 |
|------|-----|------|
| **전체 파일 수** | 56,779개 | 100% |
| **전체 코드 라인** | 약 682만 라인 | - |
| **전체 크기** | 약 1.93GB | - |
| **Java 소스** | 10,374개 (100만 라인) | 18.3% |
| **XML 파일** | 8,127개 (538만 라인) | 14.3% |
| **JavaScript 파일** | 6,433개 (30만 라인) | 11.3% |
| **HTML 파일** | 5,041개 | 8.9% |
| **이미지 파일** | 10,245개 | 18.0% |
| **컴파일 클래스** | 16,972개 | 29.9% |
| **JSP 파일** | 520개 | 0.9% |
| **기타** | 8,024개 | 14.1% |

### 1.2 프로젝트별 분포 (SVN 캐시 제외)

```
파일 수 기준
NPH_HIS  ████████████████████████████████████████  99.3%  (48,580 파일 / 1.72GB)
NPH_ECS  █                                          0.5%  (302 파일 / 193MB)
COMMON   ░                                          0.2%  (520 파일 / 12MB)
NPH_BAT  ░                                          0.0%  (180 파일 / 64KB)
NPH_BUILD░░                                          0.0%  (197 파일 / 8MB)

Note: SVN 캐시(.svn-base) 28,358개는 통계에서 제외됨
```

---

## 2. MiPlatform 파일 구조 트리

### 2.1 전체 디렉토리 구조

### 2.1A 실제 유지보수에서 먼저 보는 대표 `.mhi / navigation` 기준 화면

| 대표 화면 | 화면 XML | 대표 `.mhi` | navigation | command |
|------|------|------|------|------|
| 로그인 | `Login3.xml` | `/az/bizcom/authNavi/CheckLoginUser-new1.mhi` | `authNavi.xml` | `CheckLoginMiCMD` |
| 공통 코드 조회 | `AZ_UTL01002P.xml` | `/az/bizcom/comNavi/RetrieveComnCd.mhi` | `comNavi.xml` | `RetrieveComnCdCMD` |
| 처방 대형 화면 | `MD_ORD01001P.xml` | `/md/ord/ptmdcrNavi/RetrievePtOrder.mhi` | `ptmdcrNavi.xml` | `RetrievePtOrderCMD` |
| EDI 수신 | `HP_DMS01303M.xml` | `/hp/dms/clamNavi/RetrieveEdiRecvRcpn.mhi` | `clamNavi.xml` | `RetrieveEdiRecvRcpnCMD` |
| DRG 심사 | `HP_DMS02204M.xml` | `/hp/dms/drgNavi/RetrieveDrgRevwPtList.mhi` | `drgNavi.xml` | `RetrieveDrgRevwPtListCMD` |

이 표는 현재 백업셋에서 직접 확인된 대표 체인만 모은 것이다. 이 문서의 나머지 트리/통계 예시는 분포를 보는 reference 용도로 읽고, 실제 유지보수 추적은 위 5개 화면 기준으로 내려가는 편이 빠르다.


```
NPH_HIS/
└── webapp/
    ├── index.jsp                    # 진입점 (MiPlatform 런처)
    ├── index330.jsp                 # MiPlatform 3.3 진입점 (97.2%)
    ├── index320.jsp                 # MiPlatform 3.2 진입점 (1.8%)
    │
    ├── NPH_start.xml                # AppGroup 정의 (핵심 설정)
    │
    ├── LIBs/                        # 공통 JavaScript 라이브러리
    │   ├── common.js                # 15,234 라인
    │   ├── transaction.js           # 8,901 라인
    │   ├── grid.js                  # 12,456 라인
    │   └── ...
    │
    ├── com/                         # 공통 화면 (Common)
    │   ├── com_main.xml             # 메인 화면
    │   ├── com_login.xml            # 로그인
    │   └── ...
    │
    ├── AZ/                          # 원무/공통업무 (7.2%)
    │   ├── COM/                     # 공통업무
    │   │   ├── AZ_COM01001M.xml     # 접수등록
    │   │   ├── AZ_COM01002M.xml     # 예약관리
    │   │   └── ... (69개)
    │   ├── UTL/                     # 유틸리티
    │   └── BIZ/                     # 업무지원
    │
    ├── MD/                          # 진료업무 (35.4%)
    │   ├── OPN/                     # 외래진료
    │   │   ├── MD_OPN01001M.xml     # 진료대기
    │   │   ├── MD_OPN01002M.xml     # 진료작성
    │   │   └── ... (248개)
    │   ├── INP/                     # 입원진료
    │   └── ER/                      # 응급진료
    │
    ├── MR/                          # 원무/수납 (6.5%)
    │   ├── COM/                     # 원무공통
    │   │   ├── MR_COM01001M.xml     # 수납
    │   │   └── ... (60개)
    │   └── REC/                     # 수납관리
    │
    ├── SP/                          # 검사/방사선 (27.5%)
    │   ├── CEL/                     # 검체
    │   ├── PHA/                     # 약제
    │   ├── IMG/                     # 영상
    │   └── ... (220개)
    │
    ├── ER/                          # 응급 (27.0%)
    │   ├── ACC/                     # 응급접수
    │   ├── MNG/                     # 응급관리
    │   └── ... (222개)
    │
    └── eView/                       # EMR 뷰어
        └── common/
            └── eViewCommon.js        # EDViewer 래퍼
```

### 2.2 파일 유형별 크기 비중 (SVN 캐시 제외)

| 유형 | 파일 수 | 라인 수 | 크기 | 비중(파일수) | 설명 |
|------|---------|---------|------|--------------|------|
| Java (*.java) | 10,374 | 1,004,401 | 36MB | ████████░░░░░░░░░░░░ 18.3% | 소스 코드 |
| XML (*.xml) | 8,127 | 5,384,052 | 1.6GB | ██████░░░░░░░░░░░░░░ 14.3% | 화면/설정 |
| JavaScript (*.js) | 6,433 | 307,013 | 10MB | █████░░░░░░░░░░░░░░░ 11.3% | 클라이언트 스크립트 |
| HTML (*.html) | 5,041 | - | - | ████░░░░░░░░░░░░░░░░ 8.9% | 템플릿/정적 |
| 이미지 (*.png/*.gif/*.jpg) | 10,245 | - | - | ████████░░░░░░░░░░░░ 18.0% | UI 리소스 |
| 클래스 (*.class) | 16,972 | - | - | █████████████░░░░░░░ 29.9% | 빌드 산출물 |
| JSP (*.jsp) | 520 | 128,016 | 5.5MB | █░░░░░░░░░░░░░░░░░░░ 0.9% | 진입점 |
| 기타 | 8,024 | - | - | ████░░░░░░░░░░░░░░░░ 14.1% | 설정, 라이브러리 등 |

**코드 파일 vs 리소스/바이너리 비중**:
- 코드 파일 (Java/XML/JS/HTML/JSP): 30,495개 (53.7%)
- 리소스/바이너리 (이미지/클래스/기타): 26,284개 (46.3%)

---

## 3. 업무별(AZ/MD/MR/SP/ER) 대표 추적 기준

이 섹션은 업무군별 화면 분포를 세밀하게 열거하기보다, 현재 직접 검증된 대표 화면을 어디서 먼저 잡아야 하는지 보여주는 기준 섹션이다.

| 업무군 | 직접 검증한 대표 화면 | 먼저 볼 문서 |
|------|------|------|
| AZ 공통/유틸 | `AZ_UTL01002P.xml` | `031.front-channel/0313.ui-entry/대표화면-공통코드조회-패턴.md` |
| AZ 로그인 | `Login3.xml` | `031.front-channel/0312.navigation-command/로그인-체인-기준패턴.md` |
| MD 처방 | `MD_ORD01001P.xml` | `037.runtime-trace/MD_ORD01001P-실행체인.md` |
| HP DMS EDI | `HP_DMS01303M.xml` | `031.front-channel/0313.ui-entry/대표화면-EDI-수신-패턴.md` |
| HP DMS DRG | `HP_DMS02204M.xml` | `037.runtime-trace/HP_DMS02204M-실행체인.md` |

기존 업무별 예시 화면명, Command명, PC명은 분포를 설명하는 보조 예시에 가깝다. 실제 유지보수에서 바로 따라갈 값은 위 표의 화면/XML/.mhi/navigation/command 체인을 우선한다.

---

## 4. MiPlatform AppGroup 구조

```
NPH_start.xml (ConnectGroup)
│
├── LIBs (Type: js)
│   ├── common.js
│   ├── transaction.js
│   └── grid.js
│
├── com (Type: form)
│   └── com_main.xml
│
├── AZ_COM (Type: form)
│   └── AZ/COM/*.xml
│
├── MD_OPN (Type: form)
│   └── MD/OPN/*.xml
│
├── MD_IPN (Type: form)
│   └── MD/IPN/*.xml
│
├── MR_COM (Type: form)
│   └── MR/COM/*.xml
│
├── SP_CEL (Type: form)
│   └── SP/CEL/*.xml
│
└── ER_ACC (Type: form)
    └── ER/ACC/*.xml
```

---

## 5. 핵심 화면 목록 (파일 크기 기준 Top 20)

| 순위 | 화면ID | 경로 | 크기 | 설명 |
|------|--------|------|------|------|
| 1 | MD_OPN01002M | MD/OPN/ | 512KB | 진료작성 (메인) |
| 2 | SP_IMG01001M | SP/IMG/ | 485KB | 영상조회 |
| 3 | ER_MNG01001M | ER/MNG/ | 462KB | 응급진료 |
| 4 | MD_OPN01003M | MD/OPN/ | 438KB | 처방입력 |
| 5 | SP_PHA01001M | SP/PHA/ | 425KB | 처방조회 |
| 6 | AZ_COM01001M | AZ/COM/ | 398KB | 접수등록 |
| 7 | ER_ACC01001M | ER/ACC/ | 385KB | 응급접수 |
| 8 | MD_IPN01002M | MD/IPN/ | 372KB | 입원진료 |
| 9 | SP_CEL01001M | SP/CEL/ | 365KB | 검체채취 |
| 10 | MR_COM01001M | MR/COM/ | 358KB | 수납 |
| ... | ... | ... | ... | ... |

---

## 6. Patient Journey 구간은 별도 문서로 본다

이 문서는 MiPlatform 화면군의 분포와 reference 성격의 트리를 보는 문서다. 환자 여정, 부서 여정, 보험심사 흐름처럼 업무 단위로 읽는 내용은 아래 문서를 우선한다.

- `030.index/0305.process journey/*`
- `035.Biz-medical-Domain/*`
- `037.runtime-trace/*`

즉 이 문서 안에서는 화면 분포와 대표 진입점만 잡고, 실제 업무 여정은 별도 축에서 읽는 편이 맞다.

---

*참고: 본 문서는 실제 NPH 프로젝트 소스코드를 기반으로 한 통계 및 reference 정리다.*
