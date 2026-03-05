# MiPlatform 전체 구조 트리

> NPH 프로젝트 MiPlatform 화면 및 구성요소 섹션별 트리 구조
> 주의: 일부 Transaction/Command/PC 이름은 흐름 설명을 위한 예시 표기이며, 실제 운영명과 1:1 일치하지 않을 수 있습니다.

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

## 3. 업무별(AZ/MD/MR/SP/ER) 상세 트리

### 3.1 AZ (원무/공통) - 7.2%

```
AZ/ (원무/공통업무) - 69개 화면, 692개 Java 파일
├── COM/ (공통)
│   ├── AZ_COM01001M.xml    # 접수등록
│   ├── AZ_COM01002M.xml    # 예약관리
│   ├── AZ_COM01003M.xml    # 예약조회
│   ├── AZ_COM01004M.xml    # 접수변경
│   ├── AZ_COM01005M.xml    # 진료대기현황
│   └── ...
│
├── UTL/ (유틸리티)
│   ├── AZ_UTL01001M.xml    # 공통코드조회
│   ├── AZ_UTL01002M.xml    # 사용자관리
│   └── ...
│
└── BIZ/ (업무지원)
    ├── AZ_BIZ01001M.xml    # 휴일관리
    └── ...

Java 소스:
├── cmd/ (Command)
│   ├── RetrievePatientListCMD.java
│   ├── SavePatientCMD.java
│   └── ... (150개)
└── pc/ (Process Component)
    ├── PatientPC.java
    └── ... (542개)

통계:
├── 화면(XML): 69개 (3.2MB)
├── Java: 692개 (49,849 라인)
└── 비중: 7.2%
```

### 3.2 MD (진료) - 35.4%

```
MD/ (진료업무) - 248개 화면, 2,480개 Java 파일
├── OPN/ (외래진료)
│   ├── MD_OPN01001M.xml    # 진료대기
│   ├── MD_OPN01002M.xml    # 진료작성
│   ├── MD_OPN01003M.xml    # 처방입력
│   ├── MD_OPN01004M.xml    # 검사처방
│   ├── MD_OPN01005M.xml    # 약처방
│   ├── MD_OPN01006M.xml    # 주사처방
│   ├── MD_OPN01007M.xml    # 진료경과
│   ├── MD_OPN01008M.xml    # 상병관리
│   ├── MD_OPN01009M.xml    # 진료비계산
│   └── ... (180개)
│
├── INP/ (입원진료)
│   ├── MD_INP01001M.xml    # 입원대기
│   ├── MD_INP01002M.xml    # 입원진료
│   ├── MD_INP01003M.xml    # 회진관리
│   └── ... (50개)
│
└── ER/ (응급진료 연계)
    └── ...

Java 소스:
├── cmd/ (Command)
│   ├── RetrieveOpnPatientCMD.java
│   ├── SaveOpnOrderCMD.java
│   ├── RetrieveOpnChartCMD.java
│   └── ... (580개)
└── pc/ (Process Component)
    ├── OpnPC.java
    ├── InpPC.java
    └── ... (1,900개)

통계:
├── 화면(XML): 248개 (12.5MB)
├── Java: 2,480개 (246,301 라인)
└── 비중: 35.4% (가장 큰 업무)
```

### 3.3 MR (원무/수납) - 6.5%

```
MR/ (원무/수납) - 60개 화면, 602개 Java 파일
├── COM/ (원무공통)
│   ├── MR_COM01001M.xml    # 수납
│   ├── MR_COM01002M.xml    # 수납취소
│   ├── MR_COM01003M.xml    # 미수관리
│   ├── MR_COM01004M.xml    # 입원수납
│   ├── MR_COM01005M.xml    # 퇴원수납
│   └── ... (40개)
│
└── REC/ (수납관리)
    ├── MR_REC01001M.xml    # 일별수납현황
    └── ... (20개)

Java 소스:
├── cmd/ (Command)
│   ├── CalculateFeeCMD.java
│   ├── ReceiptCMD.java
│   └── ... (130개)
└── pc/ (Process Component)
    ├── ReceiptPC.java
    └── ... (472개)

통계:
├── 화면(XML): 60개 (2.8MB)
├── Java: 602개 (44,904 라인)
└── 비중: 6.5%
```

### 3.4 SP (검사) - 27.5%

```
SP/ (검사/방사선) - 220개 화면, 2,203개 Java 파일
├── CEL/ (검체)
│   ├── SP_CEL01001M.xml    # 검체채취
│   ├── SP_CEL01002M.xml    # 검체인수
│   ├── SP_CEL01003M.xml    # 검체검사현황
│   └── ... (60개)
│
├── PHA/ (약제)
│   ├── SP_PHA01001M.xml    # 처방조회
│   ├── SP_PHA01002M.xml    # 약품관리
│   ├── SP_PHA01003M.xml    # 조제관리
│   └── ... (70개)
│
├── IMG/ (영상)
│   ├── SP_IMG01001M.xml    # 영상조회
│   ├── SP_IMG01002M.xml    # PACS연동
│   └── ... (50개)
│
└── LIS/ (검사실)
    └── ... (40개)

Java 소스:
├── cmd/ (Command)
│   ├── RetrieveLabResultCMD.java
│   ├── SaveLabOrderCMD.java
│   └── ... (500개)
└── pc/ (Process Component)
    ├── LabPC.java
    ├── PacsPC.java
    └── ... (1,703개)

통계:
├── 화면(XML): 220개 (11.2MB)
├── Java: 2,203개 (191,036 라인)
└── 비중: 27.5%
```

### 3.5 ER (응급) - 27.0%

```
ER/ (응급) - 222개 화면, 2,225개 Java 파일
├── ACC/ (응급접수)
│   ├── ER_ACC01001M.xml    # 응급접수
│   ├── ER_ACC01002M.xml    # 응급분류
│   └── ... (50개)
│
├── MNG/ (응급관리)
│   ├── ER_MNG01001M.xml    # 응급진료
│   ├── ER_MNG01002M.xml    # 응급처치
│   ├── ER_MNG01003M.xml    # 중환자실
│   └── ... (120개)
│
└── DIS/ (퇴원)
    └── ... (52개)

Java 소스:
├── cmd/ (Command)
│   ├── RetrieveErPatientCMD.java
│   ├── SaveErChartCMD.java
│   └── ... (520개)
└── pc/ (Process Component)
    ├── ErPC.java
    └── ... (1,705개)

통계:
├── 화면(XML): 222개 (11.0MB)
├── Java: 2,225개 (187,782 라인)
└── 비중: 27.0%
```

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
├── MD_INP (Type: form)
│   └── MD/INP/*.xml
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
| 8 | MD_INP01002M | MD/INP/ | 372KB | 입원진료 |
| 9 | SP_CEL01001M | SP/CEL/ | 365KB | 검체채취 |
| 10 | MR_COM01001M | MR/COM/ | 358KB | 수납 |
| ... | ... | ... | ... | ... |

---

## 6. Patient Journey 시뮬레이션 (개념 예시)

아래는 환자 입원부터 퇴원까지의 **개념적 흐름 예시**입니다.
실제 운영 트랜잭션명(`*.mhi`) 및 Command/PC 매핑은 업무별 Navigation/XML Query에서 별도 검증이 필요합니다.

```
[Patient Journey - 환자 여정 시뮬레이션]

1단계: 접수/예약 (AZ)
├── AZ_COM01001M.xml (접수등록)
│   └── Transaction: (예시) SavePatient.mhi
│   └── Command: (예시) SavePatientCMD.java
│   └── PC: (예시) PatientPC.java
│   └── 다음: 진료대기
│
2단계: 원무/수납 (MR)
├── MR_COM01001M.xml (수납)
│   └── Transaction: (예시) CalculateFee.mhi
│   └── Command: (예시) CalculateFeeCMD.java
│   └── PC: (예시) ReceiptPC.java
│   └── 다음: 진료
│
3단계: 진료 (MD)
├── MD_OPN01001M.xml (진료대기) → MD_OPN01002M.xml (진료작성)
│   └── Transaction: (예시) SaveOpnChart.mhi
│   └── Command: (예시) SaveOpnChartCMD.java
│   └── PC: (예시) OpnPC.java
│   └── 분기: 검사/처방 → 약처방 → 수납
│
4단계: 검사/처방 (SP)
├── SP_CEL01001M.xml (검체채취) / SP_IMG01001M.xml (영상)
│   └── Transaction: (예시) SaveLabOrder.mhi
│   └── Command: (예시) SaveLabOrderCMD.java
│   └── PC: (예시) LabPC.java
│   └── 다음: 결과조회
│
5단계: 조제/투약 (SP)
├── SP_PHA01003M.xml (조제관리)
│   └── Transaction: (예시) SaveDispense.mhi
│   └── Command: (예시) SaveDispenseCMD.java
│   └── PC: (예시) PharmacyPC.java
│   └── 다음: 퇴원수납
│
6단계: 퇴원/수납 (MR)
└── MR_COM01004M.xml (퇴원수납)
    └── Transaction: (예시) DischargeReceipt.mhi
    └── Command: (예시) DischargeReceiptCMD.java
    └── PC: (예시) ReceiptPC.java
    └── 종료
```

---

*참고: 본 문서는 실제 NPH 프로젝트 소스코드를 기반으로 한 통계입니다.*
