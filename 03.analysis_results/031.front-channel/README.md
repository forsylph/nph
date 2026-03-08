# MiPlatform NPH 프로젝트 전체 구조

> **검증 기준**: `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace/NPH_HIS/` 경로 직접 분석 (2026-03-08)

## 1. 디렉토리 구조

```
/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/
├── bin/                          # 실행 환경
│   ├── apache-tomcat-9.0.105/    # Tomcat 웹서버
│   ├── eclipse/                  # Eclipse IDE
│   ├── jdk1.7.0_80.x86/          # JDK 1.7
│   ├── jdk1.8.0_181.x86_64/      # JDK 1.8
│   └── JEUS7/                    # JEUS 웹서버
│
└── workspace/                    # 개발 소스코드
    ├── COMMON/                   # 공통 라이브러리 (107 파일)
    ├── NPH_BAT/                  # 배치 프로젝트 (1 파일)
    ├── NPH_BUILD/                # 빌드 설정 (43 파일)
    ├── NPH_ECS/                  # 전자의무기록 (2,321 파일)
    └── NPH_HIS/                  # 메인 HIS 프로젝트 (65,437 파일)
        └── webapp/
            ├── index330.jsp      # 메인 진입점
            ├── Miplatform330/install/  # 클라이언트 설치
            │
            ├── ui/               # 화면 정의 (XML)
            │   ├── NPH_start.xml # AppGroup 설정
            │   ├── LIBs/         # 공통 JavaScript
            │   ├── com/          # 공통 화면 (Login3.xml 등)
            │   ├── AZ/           # 입원/공통 (409개)
            │   ├── MD/           # 진료 (969개)
            │   ├── MR/           # 의무기록 (246개)
            │   ├── SP/           # 특수진료 (910개)
            │   ├── ER/           # 응급의료 (1,262개)
            │   └── HP/           # 병원/심사 (547개)
            │
            └── devonhome/
                ├── navigation/mhi/  # 화면 네비게이션 (286개)
                └── xmlquery/        # SQL 매핑 (2,023개)
```

## 2. 업무 영역 네이밍 룰

### 2.1 약어 정의 (NPH_start.xml 기준)

| 약어 | 의미 | 하위 모듈 | 근거 |
|------|------|-----------|------|
| **AZ** | Admission Zone (입원/공통) | COM, UTL, INF, STA, CRT, SYS | `NPH_start.xml`: `AZ_COM`, `AZ_UTL`<br>화면: "수납환자", "미수납현황" |
| **ER** | Emergency Room (응급) | ACC, ADU, COM, COS, NUR, MED, PAY | `NPH_start.xml`: `ER_ACC`, `ER_NUR`<br>화면: "결의서작성", "간호사인사정보" |
| **HP** | Hospital/Health Plan (병원/심사) | BAS, COM, CTF, DMS, FEE, PAT | `NPH_start.xml`: `HP_DMS`, `HP_FEE`<br>화면: "사후심사마감", "보험조합관리" |
| **MD** | Medical (진료) | AKR, BAS, ERN, OPN, OPR, ORD, IPN | `NPH_start.xml`: `MD_ORD`, `MD_OPR`<br>화면: "처방", "수술신청" |
| **MR** | Medical Record (의무기록) | COM, RCH, RDI, STA | `NPH_start.xml`: `MR_RCH`, `MR_RDI`<br>화면: "개인정보보호요청등록" |
| **SP** | Special (특수진료) | LAB, RAY, PHA, REH, CEL, DEN, HBC | `NPH_start.xml`: `SP_LAB`, `SP_RAY`<br>화면: "외래채혈", "영상의학과 초기화면" |

### 2.2 파일 수 요약

| 약어 | UI XML | Java | 설명 |
|------|--------|------|------|
| **AZ** | 409 | 671 | 입원/공통업무 |
| **MD** | 969 | 2,453 | 진료 (외래/입원) |
| **MR** | 246 | 600 | 의무기록/원무 |
| **SP** | 910 | 2,193 | 특수진료 (검사/방사선/약제) |
| **ER** | 1,262 | 2,215 | 응급의료 |
| **HP** | 547 | 1,453 | 병원관리/보험심사 |

## 3. Form 파일 네이밍 룰

### 3.1 기본 패턴

```
{업무영역}_{하위모듈}{번호}{유형}.xml

예시:
MD_ORD01001M.xml  → MD(진료) + ORD(처방) + 01001 + M(메인화면)
HP_DMS02204M.xml  → HP(심사) + DMS(DRG) + 02204 + M(메인화면)
```

### 3.2 유형 접미사

| 접미사 | 의미 | 설명 |
|--------|------|------|
| `M` | Main | 메인 화면 |
| `P` | Popup | 팝업 화면 |
| `D` | Dialog | 다이얼로그 |

### 3.3 NPH_start.xml 연결

```xml
<AppGroup Prefix="AZ_COM" Type="form">
  <Script Baseurl="AZ/COM/"/>
</AppGroup>
<AppGroup Prefix="MD_ORD" Type="form">
  <Script Baseurl="MD/ORD/"/>
</AppGroup>
<AppGroup Prefix="HP_DMS" Type="form">
  <Script Baseurl="HP/DMS/"/>
</AppGroup>
```

## 4. 업무 영역별 파일 분포

### 4.1 UI XML (화면 정의)

| 영역 | 파일 수 | 총 라인 | 평균 라인/파일 | 비율 |
|------|--------|---------|----------------|------|
| ER | 1,262 | 962,408 | 762 | 28.2% |
| MD | 969 | 1,298,834 | 1,340 | 21.6% |
| SP | 910 | 848,481 | 932 | 20.3% |
| HP | 547 | 605,812 | 1,107 | 12.2% |
| AZ | 409 | 185,120 | 452 | 9.1% |
| MR | 246 | 223,293 | 907 | 5.5% |

### 4.2 Java (서버사이드)

| 영역 | 파일 수 | 총 라인 | 평균 라인/파일 | 비율 |
|------|--------|---------|----------------|------|
| MD | 2,453 | 246,301 | 100.4 | 25.6% |
| ER | 2,215 | 187,782 | 84.8 | 23.1% |
| SP | 2,193 | 191,036 | 87.1 | 22.9% |
| HP | 1,453 | 215,157 | 148.1 | 15.1% |
| AZ | 671 | 49,849 | 74.3 | 7.0% |
| MR | 600 | 44,904 | 74.8 | 6.3% |

### 4.3 mhi (화면 네비게이션)

| 영역 | 파일 수 | 비율 |
|------|--------|------|
| SP | 99 | 34.6% |
| ER | 92 | 32.2% |
| AZ | 29 | 10.1% |
| MD | 22 | 7.7% |
| HP | 20 | 7.0% |
| MR | 20 | 7.0% |

### 4.4 xmlquery (SQL 매핑)

| 영역 | 파일 수 | 비율 |
|------|--------|------|
| MD | 549 | 27.1% |
| ER | 451 | 22.3% |
| SP | 411 | 20.3% |
| HP | 388 | 19.2% |
| AZ | 76 | 3.8% |
| MR | 64 | 3.2% |

## 5. 복잡도 분석

| 영역 | XML 복잡도 | Java 복잡도 | 종합 |
|------|-----------|-------------|------|
| **HP** | 1,107 (2위) | **148 (1위)** | **최고** |
| **MD** | **1,340 (1위)** | 100 (2위) | **높음** |
| MR | 907 (3위) | 75 (5위) | 중간 |
| SP | 932 (4위) | 87 (3위) | 중간 |
| ER | 762 (5위) | 85 (4위) | 중간 |
| AZ | 452 (6위) | 74 (6위) | **낮음** |

## 6. 기술적 아키텍처

### 6.1 MiPlatform 3계층 구조

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Client (MiPlatform)                            │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  UI Layer - 화면 XML                                                 │    │
│  │  • Form 정의 (Design + Data + Event)                                │    │
│  │  • Dataset 바인딩                                                    │    │
│  │  • JavaScript 이벤트 핸들러                                          │    │
│  │  • Transaction() 호출 → .mhi URL                                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ HTTP/HTTPS
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Server (DEVON Framework)                        │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Navigation Layer - mhi XML                                          │    │
│  │  • 화면 요청 URL 매핑                                                │    │
│  │  • Command 클래스 지정                                               │    │
│  │  • 요청/응답 Dataset 정의                                            │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                      │                                      │
│                                      ▼                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Command Layer - Java                                                │    │
│  │  • 요청 전처리/검증                                                  │    │
│  │  • PC (Process Component) 호출                                      │    │
│  │  • 응답 Dataset 생성                                                 │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                      │                                      │
│                                      ▼                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Business Layer - PC/UC/EC                                           │    │
│  │  • PC (Process Component): 업무 로직                                 │    │
│  │  • UC (Use Case): 유스케이스                                         │    │
│  │  • EC (Entity Component): 데이터 접근                               │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                      │                                      │
│                                      ▼                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Data Access Layer - xmlquery XML                                    │    │
│  │  • SQL 문 정의                                                      │    │
│  │  • 파라미터 매핑                                                    │    │
│  │  • 결과셋 변환                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ JDBC
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Database (Oracle)                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 요청 흐름 상세

```
[화면 XML]                    [mhi XML]                    [Java]
    │                             │                           │
    │  Transaction(               │                           │
    │    "RetrievePtOrder",       │                           │
    │    "/md/ord/ptmdcrNavi/     │                           │
    │     RetrievePtOrder.mhi"    │                           │
    │  )                          │                           │
    │────────────────────────────▶│                           │
    │                             │  <action name="Retrieve  │
    │                             │   PtOrder">               │
    │                             │    <command>nph.his.md.   │
    │                             │     ord.ptmdcr.cmd.      │
    │                             │     RetrievePtOrderCMD   │
    │                             │    </command>            │
    │                             │  </action>                │
    │                             │──────────────────────────▶│
    │                             │                           │  RetrievePtOrderCMD
    │                             │                           │    .execute()
    │                             │                           │      │
    │                             │                           │      ▼
    │                             │                           │  PtmdcrPC
    │                             │                           │    .retrievePtOrder()
    │                             │                           │      │
    │                             │                           │      ▼
    │                             │                           │  [xmlquery]
    │                             │                           │    retrievePtOrder
    │                             │                           │      │
    │                             │                           │      ▼
    │                             │                           │  [Database]
    │                             │                           │      │
    │                             │                           │      ▼
    │                             │                           │  Dataset 반환
    │◀────────────────────────────│◀──────────────────────────│
    │  callback 함수 실행         │  Dataset XML 응답         │
    │                             │                           │
```

### 6.3 파일 유형별 역할

| 파일 유형 | 위치 | 역할 | 예시 |
|-----------|------|------|------|
| **화면 XML** | `ui/*.xml` | UI 정의, Dataset, Event | `MD_ORD01001P.xml` |
| **mhi XML** | `navigation/mhi/*.xml` | URL → Command 매핑 | `ptmdcrNavi.xml` |
| **Java** | `src/nph/his/*/cmd/` | 요청 처리, PC 호출 | `RetrievePtOrderCMD.java` |
| **PC Java** | `src/nph/his/*/pc/` | 업무 로직 | `PtmdcrPC.java` |
| **xmlquery XML** | `xmlquery/*.xml` | SQL 정의 | `mdord.xml` |

### 6.4 mhi 구조 상세

```xml
<!-- navigation/mhi/md/ord/ptmdcrNavi.xml 예시 -->
<navigator>
  <action name="RetrievePtOrder">
    <command>nph.his.md.ord.ptmdcr.cmd.RetrievePtOrderCMD</command>
    <dataset>ds_input</dataset>
    <dataset>ds_output</dataset>
  </action>

  <action name="SavePtOrder">
    <command>nph.his.md.ord.ptmdcr.cmd.SavePtOrderCMD</command>
    <dataset>ds_input</dataset>
    <dataset>ds_output</dataset>
  </action>
</navigator>
```

### 6.5 xmlquery 구조 상세

```xml
<!-- xmlquery/md/ord/mdord.xml 예시 -->
<querys>
  <select id="retrievePtOrder">
    <statement>
      SELECT PT_NO, PT_NM, ORD_DT, ORD_CD
      FROM   PT_ORDER
      WHERE  PT_NO = :ptNo
      AND    ORD_DT BETWEEN :frDt AND :toDt
    </statement>
    <param name="ptNo" type="VARCHAR"/>
    <param name="frDt" type="VARCHAR"/>
    <param name="toDt" type="VARCHAR"/>
  </select>
</querys>
```

### 6.6 Java 계층 구조

```
nph.his.{업무}.{하위}.{계층}
       │      │      │
       │      │      ├── cmd/    (Command)
       │      │      │           - 요청 진입점
       │      │      │           - Dataset 변환
       │      │      │
       │      │      ├── pc/     (Process Component)
       │      │      │           - 업무 로직
       │      │      │           - 트랜잭션 관리
       │      │      │
       │      │      ├── uc/     (Use Case)
       │      │      │           - 유스케이스 단위
       │      │      │
       │      │      └── ec/     (Entity Component)
       │      │                  - 데이터 접근
       │      │                  - xmlquery 호출
       │      │
       │      └── 하위 모듈 (예: ord, opr, ipn)
       │
       └── 업무 영역 (az, md, mr, sp, er, hp)

예시:
nph.his.md.ord.ptmdcr.cmd.RetrievePtOrderCMD    (Command)
nph.his.md.ord.ptmdcr.pc.PtmdcrPC                (Process Component)
nph.his.md.ord.ptmdcr.ec.PtmdcrEC                (Entity Component)
```

### 6.7 요약: 기술 스택 연결

| 계층 | 기술 | 파일 위치 | 호출 관계 |
|------|------|-----------|----------|
| **Presentation** | MiPlatform XML | `ui/*.xml` | `Transaction()` → mhi URL |
| **Navigation** | mhi XML | `navigation/mhi/*.xml` | URL → Command 클래스 매핑 |
| **Controller** | Java Command | `cmd/*.java` | PC 호출 |
| **Business** | Java PC/UC | `pc/*.java`, `uc/*.java` | EC 호출 |
| **Data Access** | Java EC + xmlquery | `ec/*.java`, `xmlquery/*.xml` | SQL 실행 |
| **Database** | Oracle | - | - |

---

## 7. 관련 문서

- [ref.MiPlatform-전체구조-트리.md](./0311.miplatform/ref.MiPlatform-전체구조-트리.md) - 상세 구조
- [A.Miplatform.md](./0311.miplatform/A.Miplatform.md) - MiPlatform 개요
- [B.MiPlatform-Transaction-패턴.md](./0311.miplatform/B.MiPlatform-Transaction-패턴.md) - 트랜잭션 패턴
- [D.miplatform_analysis_manual.md](./0311.miplatform/D.miplatform_analysis_manual.md) - 분석 매뉴얼

---

*검증 기준: `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace/NPH_HIS/` 경로 직접 분석 (2026-03-08)*