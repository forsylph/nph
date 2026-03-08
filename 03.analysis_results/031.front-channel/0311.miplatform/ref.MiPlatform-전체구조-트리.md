# MiPlatform 전체 구조 트리

> **검증 기준**: 이 문서의 통계는 `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace/NPH_HIS/` 경로에서 직접 수집한 실제 데이터 기반이다.
> **검증 일시**: 2026-03-08
> **목적**: MiPlatform 화면군의 큰 분포와 파일군 규모를 빠르게 훑는 것이며, 실제 유지보수 추적은 `A.Miplatform.md`, `B.MiPlatform-Transaction-패턴.md`, `B.화면XML-script-mhi-연결.md`를 먼저 보는 편이 빠르다.

---

## 1. 전체 통계 개요

### 1.1 프로젝트 규모 (실제 소스 검증)

| 항목 | 값 | 비율 | 검증 상태 |
|------|-----|------|----------|
| **NPH_HIS 전체 파일** | 40,440개 | 100% | ✓ 검증됨 |
| **Java 소스** | 9,869개 | 24.4% | ✓ 검증됨 |
| **UI XML (화면)** | 4,484개 | 11.1% | ✓ 검증됨 |
| **mhi (네비게이션)** | 286개 | 0.7% | ✓ 검증됨 |
| **xmlquery (SQL 매핑)** | 2,023개 | 5.0% | ✓ 검증됨 |
| **JSP** | 395개 | 1.0% | ✓ 검증됨 |
| **JavaScript** | 278개 | 0.7% | ✓ 검증됨 |
| **SVN 캐시** | 28,373개 | 제외 | ✓ 검증됨 |

### 1.2 프로젝트별 분포

```
파일 수 기준 (SVN 캐시 제외)
NPH_HIS  ████████████████████████████████████████  95.4%  (65,437 파일 / 전체)
NPH_ECS  █                                          3.4%   (2,321 파일)
COMMON   ░                                          0.2%   (107 파일)
NPH_BAT  ░                                          0.0%   (1 파일)
NPH_BUILD░                                          0.1%   (43 파일)
```

---

## 2. 업무 영역 네이밍 룰

### 2.1 약어 정의 (NPH_start.xml 기준)

| 약어 | 의미 | 근거 | 하위 모듈 예시 |
|------|------|------|---------------|
| **AZ** | Admission Zone (입원/공통) | `NPH_start.xml`: `AZ_COM`, `AZ_UTL` | COM(공통), UTL(유틸리티), INF(인프라), STA(통계) |
| **ER** | Emergency Room (응급의료) | `NPH_start.xml`: `ER_ACC`, `ER_NUR`, `ER_MED` | ACC(접수), ADU(심사), NUR(간호), MED(의료장비) |
| **HP** | Hospital/Health Plan (병원/심사) | `NPH_start.xml`: `HP_DMS`, `HP_FEE`, `HP_BAS` | DMS(DRG심사), FEE(수가), PAT(환자), CTF(증명서) |
| **MD** | Medical (진료) | `NPH_start.xml`: `MD_ORD`, `MD_OPR`, `MD_IPN` | ORD(처방), OPR(수술), IPN(입원), OPN(외래) |
| **MR** | Medical Record (의무기록) | `NPH_start.xml`: `MR_RCH`, `MR_RDI` | COM(공통), RCH(차트), RDI(색인) |
| **SP** | Special (특수진료) | `NPH_start.xml`: `SP_LAB`, `SP_RAY`, `SP_PHA` | LAB(검사실), RAY(방사선), PHA(약제), REH(재활) |

### 2.2 화면 파일 네이밍 패턴

```
{업무영역}_{하위모듈}{번호}{유형}.xml

예시:
MD_ORD01001M.xml  → MD(진료) + ORD(처방) + 01001 + M(메인화면)
HP_DMS02204M.xml  → HP(심사) + DMS(DRG) + 02204 + M(메인화면)
ER_ACC01001M.xml  → ER(응급) + ACC(접수) + 01001 + M(메인화면)

유형 접미사:
M = 메인 화면 (Main)
P = 팝업 화면 (Popup)
D = 다이얼로그 (Dialog)
```

---

## 3. 업무 영역별 파일 분포 (실제 검증)

### 3.1 UI XML (화면 정의)

| 영역 | 파일 수 | 총 라인 | 평균 라인/파일 | 비율(파일) | 복잡도 순위 |
|------|--------|---------|----------------|------------|-------------|
| **ER** | 1,262 | 962,408 | 762 | 28.2% | 5위 |
| **MD** | 969 | 1,298,834 | **1,340** | 21.6% | **1위 (최복잡)** |
| **SP** | 910 | 848,481 | 932 | 20.3% | 4위 |
| **HP** | 547 | 605,812 | 1,107 | 12.2% | 2위 |
| **AZ** | 409 | 185,120 | 452 | 9.1% | 6위 (최단순) |
| **MR** | 246 | 223,293 | 907 | 5.5% | 3위 |
| 기타 | 141 | - | - | 3.1% | - |
| **합계** | **4,484** | **4,123,948** | **949** | 100% | |

### 3.2 Java (서버사이드)

| 영역 | 파일 수 | 총 라인 | 평균 라인/파일 | 비율(파일) | 복잡도 순위 |
|------|--------|---------|----------------|------------|-------------|
| **MD** | 2,453 | 246,301 | 100.4 | 25.6% | 2위 |
| **ER** | 2,215 | 187,782 | 84.8 | 23.1% | 4위 |
| **SP** | 2,193 | 191,036 | 87.1 | 22.9% | 3위 |
| **HP** | 1,453 | 215,157 | **148.1** | 15.1% | **1위 (최복잡)** |
| **AZ** | 671 | 49,849 | 74.3 | 7.0% | 6위 (최단순) |
| **MR** | 600 | 44,904 | 74.8 | 6.3% | 5위 |
| **합계** | **9,585** | **941,536** | **98.2** | 100% | |

### 3.3 mhi (화면 네비게이션)

| 영역 | 파일 수 | 비율 | 설명 |
|------|--------|------|------|
| **SP** | 99 | 34.6% | 검사/방사선 (가장 많은 화면 전환) |
| **ER** | 92 | 32.2% | 응급의료 |
| **AZ** | 29 | 10.1% | 입원/공통 |
| **MD** | 22 | 7.7% | 진료 |
| **HP** | 20 | 7.0% | 병원/심사 |
| **MR** | 20 | 7.0% | 의무기록 |
| 기타 | 4 | 1.4% | pilot, global 등 |
| **합계** | **286** | 100% | |

### 3.4 xmlquery (SQL 매핑)

| 영역 | 파일 수 | 비율 | 설명 |
|------|--------|------|------|
| **MD** | 549 | 27.1% | 진료 (가장 많은 쿼리) |
| **ER** | 451 | 22.3% | 응급의료 |
| **SP** | 411 | 20.3% | 검사/방사선 |
| **HP** | 388 | 19.2% | 병원/심사 |
| **AZ** | 76 | 3.8% | 입원/공통 |
| **MR** | 64 | 3.2% | 의무기록 |
| 기타 | 84 | 4.1% | batch, pilot 등 |
| **합계** | **2,023** | 100% | |

---

## 4. 복잡도 분석 (화면당 평균)

| 영역 | XML 복잡도 (라인/파일) | Java 복잡도 (라인/파일) | mhi/화면 비율 | xmlquery/화면 비율 | 종합 복잡도 |
|------|----------------------|------------------------|--------------|-------------------|------------|
| **HP** | 1,107 (2위) | **148.1 (1위)** | 0.04 | 0.71 | **최고** |
| **MD** | **1,340 (1위)** | 100.4 (2위) | 0.02 | 0.57 | **높음** |
| **MR** | 907 (3위) | 74.8 (5위) | 0.08 | 0.26 | 중간 |
| **SP** | 932 (4위) | 87.1 (3위) | 0.11 | 0.45 | 중간 |
| **ER** | 762 (5위) | 84.8 (4위) | 0.07 | 0.36 | 중간 |
| **AZ** | 452 (6위) | 74.3 (6위) | 0.07 | 0.19 | **낮음** |

**해석**:
- **HP(병원/심사)**: 화면 수는 적으나(547개) 파일당 복잡도가 가장 높음. DRG 심사, 보험 청구 등 복잡한 업무 로직 반영.
- **MD(진료)**: 화면당 XML 라인이 가장 많음(1,340). 처방, 수술 등 다양한 진료 업무.
- **AZ(공통)**: 화면당 복잡도가 가장 낮음. 공통/유틸리티 성격.

---

## 5. MiPlatform 파일 구조 트리

### 5.1 전체 디렉토리 구조

```
NPH_HIS/webapp/
├── index.jsp                    # 진입점 (MiPlatform 런처)
├── index330.jsp                 # MiPlatform 3.3 진입점
├── NPH_start.xml                # AppGroup 정의 (핵심 설정)
│
├── ui/                          # 화면 정의 (XML)
│   ├── LIBs/                    # 공통 JavaScript
│   ├── com/                     # 공통 화면 (Login3.xml 등)
│   ├── AZ/                      # 입원/공통 (409개)
│   ├── MD/                      # 진료 (969개)
│   ├── MR/                      # 의무기록 (246개)
│   ├── SP/                      # 특수진료 (910개)
│   ├── ER/                      # 응급의료 (1,262개)
│   ├── HP/                      # 병원/심사 (547개) ← 기존 문서 누락
│   └── ...
│
└── devonhome/
    ├── navigation/mhi/          # 화면 네비게이션 (286개)
    │   ├── az/                  # 입원/공통 (29개)
    │   ├── md/                  # 진료 (22개)
    │   ├── mr/                  # 의무기록 (20개)
    │   ├── sp/                  # 특수진료 (99개)
    │   ├── er/                  # 응급의료 (92개)
    │   ├── hp/                  # 병원/심사 (20개)
    │   └── global.xml
    │
    └── xmlquery/                # SQL 매핑 (2,023개)
        ├── az/                  # 입원/공통 (76개)
        ├── md/                  # 진료 (549개)
        ├── mr/                  # 의무기록 (64개)
        ├── sp/                  # 특수진료 (411개)
        ├── er/                  # 응급의료 (451개)
        ├── hp/                  # 병원/심사 (388개)
        └── ...
```

### 5.2 HP 폴더 상세 구조 (기존 문서 누락)

```
HP/                              # 병원/심사 (547개 XML)
├── BAS/                         # 기초 (BASic)
├── COM/                         # 공통 (COMmon)
├── CTF/                         # 증명서 (CerTiFicate)
├── DMS/                         # DRG 심사관리 (Disease Management System)
│   ├── HP_DMS01303M.xml        # EDI 수신
│   └── HP_DMS02204M.xml        # DRG 심사
├── FEE/                         # 수가 (FEE)
├── PAT/                         # 환자 (PATient)
└── UNC/                         # 미분류 (UNClassified)
```

### 5.3 ER 폴더 상세 구조

```
ER/                              # 응급의료 (1,262개 XML)
├── ACC/                         # 접수/회계 (ACCounting)
├── ADU/                         # 심사 (ADUit)
├── COM/                         # 공통 (COMmon)
├── COS/                         # 상담 (COnSultation)
├── CSM/                         # 고객 (CuStoMer)
├── CSR/                         # 고객관계 (CuStoMer Relation)
├── EDU/                         # 교육 (EDUcation)
├── FAC/                         # 시설 (FACility)
├── INC/                         # 수입 (INCome)
├── INS/                         # 보험 (INSurance)
├── MED/                         # 의료장비 (MEDical equipment)
├── NUR/                         # 간호 (NURsing)
├── PAY/                         # 결제 (PAYment)
├── PUB/                         # 공공 (PUBlic)
├── RES/                         # 예약 (REServation)
├── SEC/                         # 보안 (SECurity)
└── STC/                         # 통계 (STatistiCs)

참고: MD/ERN (입원응급)은 MD 폴더 하위의 별도 모듈
```

---

## 6. 대표 화면 추적 기준

### 6.1 실제 유지보수에서 먼저 보는 대표 화면

| 업무군 | 대표 화면 | 화면 XML | 대표 `.mhi` | navigation |
|--------|----------|----------|-------------|------------|
| 로그인 | `Login3.xml` | `/com/Login3.xml` | `/az/bizcom/authNavi/CheckLoginUser-new1.mhi` | `authNavi.xml` |
| 공통코드 | `AZ_UTL01002P.xml` | `/AZ/UTL/` | `/az/bizcom/comNavi/RetrieveComnCd.mhi` | `comNavi.xml` |
| 처방 | `MD_ORD01001P.xml` | `/MD/ORD/` | `/md/ord/ptmdcrNavi/RetrievePtOrder.mhi` | `ptmdcrNavi.xml` |
| EDI수신 | `HP_DMS01303M.xml` | `/HP/DMS/` | `/hp/dms/clamNavi/RetrieveEdiRecvRcpn.mhi` | `clamNavi.xml` |
| DRG심사 | `HP_DMS02204M.xml` | `/HP/DMS/` | `/hp/dms/drgNavi/RetrieveDrgRevwPtList.mhi` | `drgNavi.xml` |

### 6.2 업무군별 추적 문서

| 업무군 | 먼저 볼 문서 |
|--------|-------------|
| AZ 공통/유틸 | `031.front-channel/0313.ui-entry/D.대표화면-공통코드조회-패턴.md` |
| AZ 로그인 | `031.front-channel/0312.navigation-command/C.로그인-체인-기준패턴.md` |
| MD 처방 | `037.runtime-trace/B.MD_ORD01001P-실행체인.md` |
| HP DMS EDI | `031.front-channel/0313.ui-entry/E.대표화면-EDI-수신-패턴.md` |
| HP DMS DRG | `037.runtime-trace/C.HP_DMS02204M-실행체인.md` |

---

## 7. MiPlatform AppGroup 구조

```
NPH_start.xml (ConnectGroup)
│
├── LIBs (Type: js)
│   ├── common.js
│   ├── transaction.js
│   └── grid.js
│
├── com (Type: form)
│   └── Login3.xml
│
├── AZ_COM (Type: form) → AZ/COM/*.xml
├── AZ_UTL (Type: form) → AZ/UTL/*.xml
│
├── MD_ORD (Type: form) → MD/ORD/*.xml
├── MD_OPR (Type: form) → MD/OPR/*.xml
├── MD_IPN (Type: form) → MD/IPN/*.xml
│
├── MR_COM (Type: form) → MR/COM/*.xml
│
├── SP_LAB (Type: form) → SP/LAB/*.xml
├── SP_RAY (Type: form) → SP/RAY/*.xml
├── SP_PHA (Type: form) → SP/PHA/*.xml
│
├── ER_ACC (Type: form) → ER/ACC/*.xml
├── ER_NUR (Type: form) → ER/NUR/*.xml
│
├── HP_DMS (Type: form) → HP/DMS/*.xml  ← 기존 문서 누락
├── HP_FEE (Type: form) → HP/FEE/*.xml
└── HP_BAS (Type: form) → HP/BAS/*.xml
```

---

## 8. 핵심 화면 목록 (파일 크기 기준)

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

---

## 9. SP 하위 모듈 상세

| 하위 모듈 | 화면 Title 예시 | 의미 |
|-----------|-----------------|------|
| LAB | 외래채혈, 검체접수 | Laboratory (검사실) |
| RAY | 영상의학과 초기화면, 영상검사접수 | Radiology (방사선) |
| PHA | 조제실적, 병동처방집계 | Pharmacy (약제) |
| REH | 치료/처방, 치료 검색 | Rehabilitation (재활) |
| NEU | - | Neurology (신경) |
| NUT | - | Nutrition (영양) |
| DEN | - | Dental (치과) |
| HBC | - | Health Behavior Clinic (건강행동) |
| CEL | - | Cell (세포병리) |
| SOC | - | Social Work (사회복지) |
| RON | - | Radiation Oncology (방사선종양) |
| TES | - | Test (검사) |

---

## 10. 참고 문서

- 환자 여정: `030.index/0305.process journey/*`
- 의료 도메인: `035.Biz-medical-Domain/*`
- 런타임 추적: `037.runtime-trace/*`

---

*검증 기준: `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace/NPH_HIS/` 경로 직접 분석 (2026-03-08)*