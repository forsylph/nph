# MiPlatform NPH 프로젝트 전체 구조

## 1. 디렉토리 구조

```
/home/forsylph/ollama/NPH/03.analysis_results/
├── 032.Architecture - Miplatform/
│   ├── miplatform_analysis_manual.md
│   └── ref.MiPlatform-전체구조-트리.md (이 파일)
│
/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/
├── bin/                          # 실행 환경
│   ├── apache-tomcat-9.0.105/    # Tomcat 웹서버
│   ├── eclipse/                  # Eclipse IDE
│   ├── jdk1.7.0_80.x86/          # JDK 1.7
│   ├── jdk1.8.0_181.x86_64/      # JDK 1.8
│   └── JEUS7/                    # JEUS 웹서버
│
└── workspace/                    # 개발 소스코드
    ├── COMMON/                   # 공통 라이브러리
    ├── NPH_BAT/                  # 배치 프로젝트
    ├── NPH_BUILD/                # 빌드 설정
    ├── NPH_ECS/                  # 전자의무기록
    ├── NPH_HIS/                  # 메인 HIS 프로젝트
    │   └── webapp/
    │       ├── index330.jsp      # 메인 진입점
    │       ├── Miplatform330/install/  # 클라이언트 설치
    │       ├── ui/               # 화면 정의
    │       │   ├── NPH_start.xml # AppGroup 설정
    │       │   ├── LIBs/         # 공통 JavaScript
    │       │   ├── AZ/           # 관리 업무
    │       │   ├── MD/           # 모바일/외래
    │       │   ├── MR/           # 원무/수납
    │       │   ├── SP/           # 검사/방사선
    │       │   └── ER/           # 응급
    │       └── devonhome/        # DEVON 설정
    │           ├── conf/
    │           ├── navigation/
    │           └── xmlquery/
    └── Servers/                  # 서버 설정
```

## 2. Form 파일 네이밍 룰

### 2.1 기본 패턴

| 패턴 | 의미 | 예시 | 설명 |
|------|------|------|------|
| `XXXXX00000M.xml` | **메인 화면** | `AZ_COM01001M.xml` | 주 업무 화면 |
| `XXXXX00000P.xml` | **팝업 화면** | `AZCOMF001P.xml` | 팝업/모달 창 |
| `XXXXX00000D.xml` | **다이얼로그** | - | 다이얼로그 창 |

### 2.2 접두어(Prefix) 구조

| 위치 | 의미 | 예시 |
|------|------|------|
| `AZ_` | 업무 영역 (관리/공통) | `AZ_COM` |
| `ER_` | 업무 영역 (전자의무기록) | `ER_MED` |
| `MD_` | 업무 영역 (모바일/외래) | `MD_OPN` |
| `MR_` | 업무 영역 (원무/수납) | `MR_COM` |
| `SP_` | 업무 영역 (검사/방사선) | `SP_***` |

### 2.3 예시 분해

```
AZ_COM01001M.xml
│  │   │   │
│  │   │   └── M: 메인 화면
│  │   └────── 001: 일련번호
│  │   └──── 01: 서브 카테고리
│  └──────── COM: 공통 업무
└─────────── AZ: 관리 업무 영역
```

```
AZCOMF001P.xml
│  │   │  │
│  │   │  └── P: 팝업 화면
│  │   └───── F001: 기능 번호
│  └───────── COM: 공통 업무
└──────────── AZ: 관리 업무 영역
```

### 2.4 NPH_start.xml 연결

```xml
<AppGroup Prefix="AZ_COM" Type="form">
  <Script Baseurl="AZ/COM/"/>
</AppGroup>
<AppGroup Prefix="MD_OPN" Type="form">
  <Script Baseurl="MD/OPN/"/>
</AppGroup>
```

---

*작성일: 2026-03-07*
*참조: miplatform_analysis_manual.md*
