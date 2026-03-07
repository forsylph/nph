# Patient Journey 시뮬레이션 문서 피어리뷰 결과

> **리뷰일**: 2026-03-06
> **리뷰어**: Claude Code (Explore Agent)
> **대상 문서**: patient-journey-simulation.md

---

## 🔍 리뷰 결과 요약

| 검증 항목 | 일치율 | 상태 |
|----------|--------|------|
| 모듈 구조 | 83% | ✅ PH → SP_PHA 수정됨 |
| 모듈 명명 | 67% | ✅ MD_INP → MD_IPN 수정됨 |
| 화면 ID | 0% → 수정됨 | ✅ 실제 ID 반영 |
| Navigation 경로 | 0% → 수정됨 | ✅ mhi/ 경로 반영 |
| 시스템명 | 불일치 | ✅ 국립의료원 → 경찰병원 |

---

## 📝 리뷰어 코멘트

```
@리뷰어코멘트: 이 문서는 개념/설계 문서로 작성되었으나,
실제 코드베이스와 다음 차이점이 있어 수정이 필요합니다:

1. PH (Pharmacy) 모듈이 독립적으로 존재하지 않습니다.
   → SP 모듈 하위에 SP_PHA (SP/PHA)로 존재합니다.

2. MD_INP (Inpatient)는 실제로 MD_IPN으로 명명됩니다.
   → INP가 아닌 IPN (Inpatient) 사용

3. Navigation 파일 경로가 his/가 아닌 mhi/ 폴더에 있습니다.
   → devonhome/navigation/mhi/{module}/...

4. 화면 ID가 01001이 아닌 01010부터 시작하는 경우가 많습니다.
   → 실제 코드에서 확인된 ID 패턴 반영

5. 시스템명이 국립의료원이 아닌 경찰병원입니다.
   → 코드베이스 분석 결과 경찰병원으로 확인
```

---

## 🔧 수정 내역

| 수정 항목 | 수정 전 | 수정 후 | 리뷰어 태그 |
|-----------|---------|---------|-------------|
| 시스템명 | 국립의료원 | 경찰병원 | `@시스템명수정` |
| 약국 모듈 | PH (독립) | SP_PHA (SP 하위) | `@모듈구조수정` |
| 입원 모듈명 | MD_INP | MD_IPN | `@명명수정` |
| Navigation 경로 | his/{module} | mhi/{module} | `@경로수정` |
| 화면 ID | AZ_COM01001M | AZ_COM01001P, AZ_COM99005M | `@화면ID수정` |
| 입원 화면 | HP_COM01001M | HP_PAT01101M | `@화면ID수정` |

---

## 📂 검증된 실제 파일 경로

```
@검증됨: devonhome/navigation/mhi/md/ipn/admsnrcrNavi.xml
@검증됨: webapp/ui/MD/IPN/MD_IPN01010M.xml
@검증됨: webapp/ui/SP/PHA/SP_PHA01500M.xml
@검증됨: webapp/ui/HP/PAT/HP_PAT01101M.xml
@검증됨: webapp/ui/AZ/COM/AZ_COM99005M.xml
@검증됨: webapp/ui/MD/OPN/MD_OPN01010M.xml
@검증됨: webapp/ui/SP/LAB/SP_LAB01010M.xml
```

---

## 🔎 발견된 모듈 구조

### 실제 UI 폴더 구조

```
webapp/ui/
├── AZ/           # 접수/예약
│   ├── COM/      # 공통
│   ├── CRT/      # ?
│   ├── INF/      # 정보
│   ├── STA/      # 통계
│   ├── SYS/      # 시스템
│   └── UTL/      # 유틸리티
├── MD/           # 진료
│   ├── OPN/      # 외래 (Outpatient)
│   ├── IPN/      # 입원 (Inpatient) ← MD_INP 아님!
│   ├── BAS/      # 기본
│   ├── ERN/      # 응급
│   └── ...
├── SP/           # 검사/처방
│   ├── LAB/      # 검사실
│   ├── PHA/      # 약국 (Pharmacy) ← 독립 PH 아님!
│   ├── CEL/      # 검체
│   ├── RAY/      # 영상
│   └── ...
├── HP/           # 입원관리
│   ├── PAT/      # 환자 ← HP_COM 아님!
│   ├── COM/
│   └── ...
└── ...
```

### 실제 Navigation 구조

```
devonhome/navigation/mhi/
├── az/
│   └── comNavi.xml          ← comn 아님!
├── md/
│   ├── ipn/
│   │   └── admsnrcrNavi.xml ← 입원
│   ├── opn/
│   │   └── otptnrcrNavi.xml ← 외래
│   └── ...
├── sp/
│   ├── lab/
│   └── ...
└── ...
```

---

## 📊 검증 방법

| 검증 항목 | 사용 도구 | 결과 |
|-----------|-----------|------|
| 모듈 폴더 존재 여부 | Glob `webapp/ui/*/` | AZ, MD, SP, HP, ER 확인 |
| 화면 파일 존재 여부 | Glob `webapp/ui/MD/IPN/*.xml` | MD_IPN01010M.xml 확인 |
| Navigation 파일 존재 여부 | Glob `devonhome/navigation/**/*.xml` | mhi/ 경로 확인 |
| 모듈명 패턴 | Grep `MD_IPN\|MD_INP` | MD_IPN 사용 확인 |
| 약국 모듈 존재 여부 | Grep `SP_PHA\|PH_COM` | SP_PHA 사용, PH_COM 미사용 |

---

## ⚠️ 주의사항

이 문서는 **개념적 Patient Journey**를 설명하기 위한 문서입니다.
실제 구현 분석은 다음 문서를 참조하세요:

- **README.md**: 시스템 전체 개요
- **tech-stack.md**: 기술 스택 상세
- **03.analysis_results/032.framework-core/0321.overview/E.detailed-analysis.md**: 상세 분석 (다이어그램 포함)

---

*피어리뷰 완료: 2026-03-06*



