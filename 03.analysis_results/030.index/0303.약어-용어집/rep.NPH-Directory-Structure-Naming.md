# NPH 디렉토리 구조 및 네이밍 룰 분석

> 분석 완료: 2026-03-07

---

## 1. 프로젝트 모듈 구조

```
NPH/
├── NPH_HIS/      # 병원정보시스템 (메인)
├── NPH_ECS/      # 전자동의시스템 (EMR 연동)
├── NPH_BAT/      # 배치 처리
├── NPH_BUILD/    # 빌드 설정
└── COMMON/       # 공통 라이브러리
```

---

## 2. Java 클래스 네이밍 규칙

| 접미사 | 용도 | 예시 |
|--------|------|------|
| `CMD` | Command (컨트롤러) | `CheckLoginNewCMD.java` |
| `PC` | Presentation Component | `LoginPC.java` |
| `EC` | Enterprise Component | `LoginEC.java` |
| `VO` | Value Object | `UserVO.java` |
| `UTIL` | Utility | `DateUTIL.java` |

### 계층 구조

```
┌─────────────┐
│   CMD       │  ← Navigation XML에서 매핑
│  (Command)  │
└──────┬──────┘
       │ 호출
       ▼
┌─────────────┐
│    PC       │  ← 업무 흐름 조합
│ (Process    │     여러 EC/UC 호출
│  Component) │
└──────┬──────┘
       │ 호출
       ▼
┌─────────────┐
│  EC / UC    │  ← DB 접근, 공통 규칙
│(Enterprise/ │
│  Usecase)   │
└─────────────┘
```

---

## 3. 비즈니스 모듈 약어

| 약어 | 한글명 | 영문명 |
|------|--------|--------|
| `az` | 입원 | Admission Zone |
| `er` | 응급 | Emergency Room |
| `hp` | 병원 | Hospital |
| `md` | 진료 | Medical |
| `mr` | 의무기록 | Medical Record |
| `sp` | 특수 | Special |

### 패키지 구조 예시

```
nph.his.<모듈>.<업무>.<계층>

예: nph.his.az.bizcom.auth.cmd.CheckLoginMiCMD
       │    │   │      │    └── CMD 클래스
       │    │   │      └── 업무 (auth, bizcom...)
       │    │   └── 모듈 (az: 입원)
       │    └── his: NPH_HIS
       └── nph: 최상위 패키지
```

---

## 4. NPH_HIS vs NPH_ECS 비교

| 항목 | NPH_HIS | NPH_ECS |
|------|---------|---------|
| **패키지 구조** | `nph.his.<모듈>` | `BKSNP.EMR` |
| **프레임워크** | DevOn Framework | 레거시/서블릿 |
| **네이밍 스타일** | 접미사 (CMD, PC, EC) | 접두사 (BK_) |
| **UI** | MiPlatform UI | ActiveX/HTML |

---

## 5. 레거시 ECS 네이밍 (NPH_ECS)

### 패턴

```
BK_<목적>.java          # BK 접두사
<목적>_<액션>.java       # 목적_액션 형식
<대문자>_<목적>.java     # 완전 대문자
```

### 예시

| 파일명 | 설명 |
|--------|------|
| `BK_logMsg.java` | 로그 메시지 |
| `BK_util.java` | 유틸리티 |
| `DB_CONN.java` | DB 연결 (대문자) |
| `TBL_PROC_ocsapabh.java` | 테이블 프로시저 |

---

## 6. 분석 문서 디렉토리 네이밍 규칙

### 디렉토리 구조

```
03.analysis_results/
├── 030.index/              # 인덱스, 규칙, 용어집
│   ├── 0301.운영규칙 및 문서맵/
│   ├── 0303.약어-용어집/
│   └── 0304.읽는순서/
├── 031.front-channel/      # 프론트 채널
│   ├── 0311.miplatform/
│   ├── 0312.navigation-command/
│   └── 0313.ui-entry/
├── 032.framework-core/     # 프레임워크 코어
│   ├── 0321.overview/
│   ├── 0322.data-access/
│   └── 0323.batch-rule/
├── 033.platform-services/  # 플랫폼 서비스
│   ├── 0331.security-auth/
│   ├── 0332.integration/
│   └── 0333.Solutions/
└── ...
```

### 디렉토리 네이밍

| 레벨 | 패턴 | 예시 |
|------|------|------|
| 1단계 | `0XX.카테고리명` | `032.framework-core` |
| 2단계 | `0XXY.서브카테고리명` | `0323.batch-rule` |

### 파일 네이밍

| 유형 | 패턴 | 예시 |
|------|------|------|
| 문서 | `[알파벳].[주제]-[상세].md` | `A.DevOn-Batch-컨테이너-개요.md` |
| 개요 | `README.md` | 디렉토리 개요/인덱스 |
| 백업 | `[원본].[스크립트]-[타임스탬프].bak` | `...md.dedup-20260307-2102.bak` |

---

## 7. 관련 문서

- [약어-용어집.md](./약어-용어집.md) - 클래스명, 계층명 상세
- [README.md](../README.md) - 프로젝트 전체 개요