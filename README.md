# NPH 프로젝트 분석 보고서

> 국립의료원 병원정보시스템(NPH) 소스코드 분석
> 분석일: 2025-03-05

---

## 개요

| 항목 | 내용 |
|------|------|
| **프로젝트명** | NPH (National XXX Hospital Information System) |
| **분석 대상** | `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace` |
| **총 파일 수** | 약 17,800개 |
| **구성 프로젝트** | COMMON, NPH_ECS, NPH_HIS, NPH_BAT, NPH_BUILD |

---

## 기술 스택

### 프레임워크

| 구분 | 기술 | 버전 | 비고 |
|------|------|------|------|
| UI 플랫폼 | **MiPlatform** | 3.2 / 3.3 | 투비소프트(구 토비소프트) |
| 백엔드 프레임워크 | **DevOn Framework** | 3.0 ~ 4.1 | LG CNS |
| WAS | JEUS | 6.0 | 티맥스소프트 |
| DB | Oracle / Tibero | - | - |
| JDK | Java | 1.7.0_80 | - |

### MiPlatform 버전 현황

| 프로젝트 | JAR | 실행 환경 | 비고 |
|---------|-----|----------|------|
| COMMON | 3.2 | 3.2 | 단일 버전 |
| NPH_HIS | 3.2 | 3.2 + 3.3 | **병행 사용** |
| NPH_ECS | - | - | 미사용 |

> **참고**: NPH_HIS는 MiPlatform 3.2와 3.3을 병행 사용 중입니다.
> - 주요 화면 (index.jsp, login-new.jsp): 3.3
> - 레거시 화면 (sp_ray, login-se.jsp): 3.2

---

## 프로젝트 구조

```
workspace/
├── COMMON/                 # 공통 라이브러리 (nphCom.jar)
├── NPH_BAT/                # 배치 작업
├── NPH_BUILD/              # 빌드 설정
├── NPH_ECS/                # 전자동의시스템 (EMR 연동)
└── NPH_HIS/                # 병원정보시스템 (메인)
    ├── Miplatform320/      # MiPlatform 3.2
    ├── Miplatform330/      # MiPlatform 3.3
    ├── devonhome/          # DevOn Framework 설정
    └── webapp/
        ├── EMR_DATA/       # EMR 데이터 처리
        └── ui/             # MiPlatform 화면 파일
```

---

## 프로젝트별 상세 분석

### 1. COMMON

**역할**: 공통 라이브러리 (nphCom.jar)

**주요 패키지**:
- `devonx.nph.miplatform.*`: MiPlatform 연동 클래스
- `devonx.nph.system.servlet.*`: GeneralServlet, MiplatformServlet
- `devonx.nph.util.*`: 공통 유틸리티

**의존성**:
- MiPlatform 3.2 (miplatform-3.2.jar)
- DevOn Framework (devon-core.jar, devon-framework.jar)

**배포**: NPH_HIS의 `WEB-INF/lib/`로 복사

---

### 2. NPH_ECS

**역할**: 전자동의시스템 (Electronic Consent System)

**특징**:
- MiPlatform 미사용
- 전자서명 (SignGate) 연동
- EDViewer (ActiveX/OCX) 사용

**주요 패키지**:
- `BKSNP.EMR.*`: EMR 데이터 연동 및 처리

**핵심 클래스**:
| 클래스 | 설명 |
|--------|------|
| `ExternalDBPusher.java` | EMR 외부 DB 연동 |
| `TBL_PROC_ocsapabh.java` | 입원 환자 진료 데이터 처리 |
| `BK_ajax.java` | EMR 데이터 처리 지원 |

**데이터소스**: `jdbc/NPHBKDB` (Tibero)

---

### 3. NPH_HIS

**역할**: 병원정보시스템 (Hospital Information System)

**특징**:
- DevOn Framework + MiPlatform 3.2/3.3
- 다중 데이터소스 (8개)
- 배치 관리 시스템 포함

**주요 모듈**:
| 모듈 | 설명 |
|------|------|
| AZ | 원무/접수 |
| ER | 응급 |
| HP | 입원 |
| MD | 진료 |
| MR | 의무기록 |
| SP | 외래 |

**데이터소스**:
```
jdbc/NPHDB      (메인)
jdbc/NPHENCDB   (암호화)
jdbc/NPHSMSDB   (SMS)
jdbc/NPHDIFDB   (DIF)
jdbc/NPHBKDB    (백업)
jdbc/NPHQCDB    (QC)
jdbc/NPHINFDB   (INF)
jdbc/NPHGW      (게이트웨이)
```

---

### 4. EMR 시스템 분석

EMR 관련 기능은 두 프로젝트에 분산되어 있습니다:

#### NPH_ECS: EMR 데이터 연동
- 외부 시스템 ↔ OCS/HIS 연동
- 전자서명 및 문서 저장
- EMR 데이터 전송/동기화

#### NPH_HIS: EMR 조회 및 진료
- EMR 뷰어 (EDViewer)
- 응급의료진료 관리
- 모바일 EMR 지원

**EMR 관련 경로**:
```
NPH_ECS/
├── src/BKSNP/EMR/          # EMR 데이터 처리 클래스
└── webapp/EMR_DATA/        # EMR 데이터 저장

NPH_HIS/
├── webapp/EMR_DATA/        # EMR 데이터 처리 JSP
└── webapp/jsp/md_mobile/emr/  # 모바일 EMR
```

---

## DevOn Framework (LG CNS)

### 출처 확인

해당 시스템은 **LG CNS의 DevOn Framework**를 기반으로 개발되었습니다.

**저작권 정보** (MiplatformConverter.java):
```java
/*
 * Copyright ⓒ LG CNS, Inc. All rights reserved.
 *
 * DevOn 의 클래스를 실제 프로젝트에 사용하는 경우 DevOn 개발담당자에게
 * 프로젝트 정식명칭, 담당자 연락처(Email)등을 mail로 알려야 한다.
 *
 * (주의!) 원저자의 허락없이 재배포 할 수 없으며
 * LG CNS 외부로의 유출을 하여서는 안 된다.
 */
```

**버전 정보**:
- DevOn Framework: 3.0.0 ~ 4.1.0
- 개발사: LG CNS, Inc.
- 연락처: devon@lgcns.com
- 홈페이지: http://www.dev-on.com

### 라이선스 주의사항

> 소스를 변경하여 사용하는 경우 DevOn 개발담당자에게 변경된 소스 전체와 변경된 사항을 알려야 한다. 원저자의 허락없이 재배포 할 수 없으며 LG CNS 외부로의 유출을 하여서는 안 된다.

---

## 마이그레이션 분석

### 복잡도 평가

| 프로젝트 | 복잡도 | 주요 이슈 |
|----------|--------|-----------|
| COMMON | 중 | MiPlatform 3.2 의존성, DevOn Framework 의존성 |
| NPH_ECS | 하 | MiPlatform 미사용, SignGate/EDViewer 유지 필요 |
| NPH_HIS | 상 | MiPlatform 3.2/3.3 병행, DevOn Framework, 화면 수 다수 |

### Nexacro 사용 여부

**결과**: 사용되지 않음

- Nexacro 관련 파일 (.xfdl, 설정 파일): **0건**
- MiPlatform만 사용됨

---

## 참고 사항

### MiPlatform 수명 주기
- **MiPlatform 3.3**: 2025년 12월 31일 판매 종료 예정
- **권장**: Nexacro Platform으로의 마이그레이션

### 의존성 정리

```
NPH_HIS
├── COMMON (nphCom.jar)
│   ├── MiPlatform 3.2
│   └── DevOn Framework
├── MiPlatform 3.2/3.3
└── DevOn Framework

NPH_ECS
├── MiPlatform 미사용
└── SignGate (전자서명)
```

---

## 결론

1. **NPH 시스템**은 LG CNS의 DevOn Framework와 투비소프트의 MiPlatform을 기반으로 한 레거시 병원정보시스템입니다.

2. **MiPlatform 3.2와 3.3이 병행** 사용되고 있어 마이그레이션 시 두 버전을 모두 고려해야 합니다.

3. **NPH_ECS는 MiPlatform을 사용하지 않으나** SignGate/EDViewer 등 ActiveX 기반 컴포넌트가 있어 모던 브라우저 마이그레이션이 필요합니다.

4. **DevOn Framework는 LG CNS의 라이선스**를 가지므로 변경 시 통지 및 외부 유출 금지 등의 조건을 준수해야 합니다.

---

*분석 완료*
