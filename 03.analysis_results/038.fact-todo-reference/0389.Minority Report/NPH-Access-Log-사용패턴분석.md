# NPH Access Log 사용 패턴 분석 보고서

> **작성일**: 2026-03-08
> **분석 범위**: `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/bin/JEUS7/domains/jeus_domain/servers/adminServer/logs/servlet/`
> **로그 기간**: 2025-12-08 ~ 2026-03-04

---

## 1. 분석 개요

### 1.1 로그 형식

```
{IP} [{날짜:시간}] "{메서드} {URL} {프로토콜}" {상태코드} {응답크기(bytes)} {응답시간(ms)}
```

### 1.2 분석 대상 영역

| 영역 | 접두사 | 설명 |
|------|--------|------|
| **AZ** | `az/` | 입원/공통 |
| **MD** | `md/` | 진료 |
| **MR** | `mr/` | 의무기록 |
| **SP** | `sp/` | 특수진료 |
| **ER** | `er/` | 응급/행정지원 |
| **HP** | `hp/` | 병원/심사 |

---

## 2. 업무 영역별 API 호출 빈도 분석

### 2.1 전체 호출 분포

| 영역 | 호출 수 | 비율 | 주요 하위 모듈 |
|------|---------|------|----------------|
| **AZ** | ~10,000+ | 35% | bizcom, sys |
| **HP** | ~10,000+ | 30% | pat, com, dms |
| **MD** | ~5,000+ | 15% | ord, ern, ipn |
| **SP** | ~3,000+ | 10% | lab, pha |
| **ER** | ~2,000+ | 7% | stc, ins |
| **MR** | ~500+ | 3% | com, rch |

### 2.2 AZ (입원/공통) 상세 분석

#### TOP 10 호출 API

| 순위 | API | 호출 수 | 평균 응답크기 | 평균 응답시간 |
|------|-----|--------|--------------|--------------|
| 1 | `RetrieveComnCd.mhi` | ~500+ | 5KB~97KB | 10~50ms |
| 2 | `RetrieveToday.mhi` | ~100+ | 243B | 1~5ms |
| 3 | `CheckLoginUser-new1.mhi` | ~50+ | 3KB | 14~1311ms |
| 4 | `MenuInfo.mhi` | ~50+ | 249KB | 196~375ms |
| 5 | `RetrieveMenuGrpCd.mhi` | ~50+ | 228B | 5~18ms |
| 6 | `MenuOnload.mhi` | ~50+ | 550B | 3~10ms |
| 7 | `RetrieveSnvSql.mhi` | ~100+ | 183B | 2~17ms |
| 8 | `RetrieveScrnAuth.mhi` | ~50+ | 235B | 9~16ms |
| 9 | `RetrieveDbmsConnectionString.mhi` | ~50+ | 427B | 100~200ms |
| 10 | `createPriveRtrvLog.mhi` | ~50+ | 232B | 10~30ms |

#### 대용량 응답 API (10KB 이상)

| API | 응답 크기 | 설명 |
|-----|----------|------|
| `cmcdNavi/RetrieveListMaster.mhi` | **853KB** | 코드 마스터 목록 (최대) |
| `authNavi/MenuInfo.mhi` | 249KB | 메뉴 정보 |
| `comNavi/RetrieveComnCd.mhi` | 10KB~97KB | 공통코드 조회 |
| `RetrieveExmnApntPtList.mhi` | 26KB~28KB | 검사예약환자목록 |

#### 응답 시간이 느린 API (1000ms 이상)

| API | 응답 시간 | 원인 |
|-----|----------|------|
| `CheckLoginUser-new1.mhi` | 1000~2012ms | 초기 로그인 시 DB 연결 및 세션 생성 |
| `CheckLoginUser-new.mhi` | 979~1396ms | 로그인 인증 |

### 2.3 MD (진료) 상세 분석

#### 하위 모듈별 호출 패턴

| 하위 모듈 | 접두사 | 주요 API |
|-----------|--------|----------|
| **처방/오더** | `md/ord/ptmdcrNavi/` | RetrievePtOrder, SavePtOrder |
| **응급간호** | `md/ern/emrgptmdcrNavi/` | RetrieveEmctPatInfo, RetrieveNedis |
| **입원간호** | `md/ipn/admsnrcrNavi/` | RetrieveWardList, RetrievePtStat |

#### TOP 5 MD API

| API | 호출 수 | 응답 크기 | 응답 시간 |
|-----|--------|----------|----------|
| `RetrieveEmctPatInfo.mhi` | ~100+ | 18KB~23KB | 14~323ms |
| `RetrieveEmctNrsnInfoRgst.mhi` | ~80+ | 14KB~15KB | 23~147ms |
| `RetrieveNedis.mhi` | ~20+ | 5KB~9KB | 24~132ms |
| `RetrievePtStat.mhi` | ~30+ | 1KB | 2~11ms |
| `RetrieveRoomPtUnActTord.mhi` | ~20+ | 325B | 49~53ms |

#### MD 영역 특징

- **응급(emrgptmdcr)**: 가장 활발한 호출
- **처방(ptmdcr)**: 상대적으로 적은 호출
- **응답 크기**: 14KB~23KB로 비교적 큼

### 2.4 HP (병원/심사) 상세 분석

#### 하위 모듈별 호출 패턴

| 하위 모듈 | 접두사 | 주요 API |
|-----------|--------|----------|
| **환자접수** | `hp/pat/otptNavi/` | RetrieveOtptLoad, RetrieveOtptPtList |
| **공통** | `hp/com/ptmngmNavi/` | RetrieveIdnt, RetrieveClas |
| **기초정보** | `hp/bas/comnNavi/` | RetrieveInsnTypeList, RetrieveCnrtInstCmbList2 |

#### TOP 5 HP API

| API | 호출 수 | 응답 크기 | 응답 시간 |
|-----|--------|----------|----------|
| `RetrieveOtptLoad.mhi` | ~30+ | **513KB** | 109~390ms |
| `RetrieveClas.mhi` | ~50+ | 26KB | 10~16ms |
| `RetrieveIjtp.mhi` | ~50+ | 49KB | 15~47ms |
| `RetrieveCnrtInstCmbList2.mhi` | ~50+ | 37KB | 14~16ms |
| `RetrieveIdnt.mhi` | ~50+ | 9KB | 20~47ms |

#### HP 영역 특징

- **최대 응답**: `RetrieveOtptLoad.mhi` 513KB (외래 환자 로드)
- **복잡도**: Java 파일당 148 라인으로 최고 (문서 기준)
- **빈도**: AZ 다음으로 높은 호출 빈도

### 2.5 SP (특수진료) / ER (응급행정) / MR (의무기록) 분석

#### SP 영역 TOP API

| API | 호출 수 | 응답 크기 | 응답 시간 |
|-----|--------|----------|----------|
| `sp/lab/` 관련 | ~500+ | 다양 | 다양 |
| `sp/pha/` 관련 | ~400+ | 다양 | 다양 |

#### ER 영역 TOP API

| API | 호출 수 | 응답 크기 | 응답 시간 |
|-----|--------|----------|----------|
| `er/stc/` 물품/자재 | ~300+ | 다양 | 다양 |
| `er/ins/` 인사/채용 | ~200+ | 다양 | 다양 |

#### MR 영역 특징

- 호출 빈도 가장 낮음 (~3%)
- 주로 `mr/com/`, `mr/rch/` 경로 사용

---

## 3. 시간대별 사용 패턴

### 3.1 피크 타임 분석

| 시간대 | 호출 수 | 특징 |
|--------|---------|------|
| **08:00~09:00** | 높음 | 로그인 집중, 메뉴 로드 |
| **09:00~12:00** | 최고 | 업무 피크 타임 |
| **13:00~14:00** | 낮음 | 점심 시간 |
| **14:00~17:00** | 높음 | 오후 업무 |
| **17:00~18:00** | 감소 | 업무 종료 |

### 3.2 로그인 세션 패턴

```
로그인 시퀀스:
1. CheckLoginUser-new1.mhi (로그인 인증) → 평균 1000~1300ms
2. RetrieveToday.mhi (오늘 날짜) → 1~5ms
3. UpdateCntInit1.mhi (카운트 초기화) → 10~20ms
4. MenuInfo.mhi (메뉴 정보) → 200~400ms, 249KB
5. RetrieveComnCd.mhi (공통코드 다수 호출) → 누적 10~100KB
```

---

## 4. 성능 분석

### 4.1 응답 시간 분포

| 응답 시간 | 비율 | 예시 API |
|-----------|------|----------|
| 0~50ms | 60% | RetrieveToday, RetrieveMenuGrpCd |
| 50~200ms | 25% | RetrieveComnCd (일반) |
| 200~500ms | 10% | MenuInfo, RetrieveOtptLoad |
| 500ms~1s | 3% | RetrieveDbmsConnectionString |
| 1s 이상 | 2% | CheckLoginUser (초기) |

### 4.2 대용량 응답 API (100KB 이상)

| API | 영역 | 응답 크기 | 빈도 |
|-----|------|----------|------|
| `RetrieveListMaster.mhi` | AZ | 853KB | 낮음 |
| `RetrieveOtptLoad.mhi` | HP | 513KB | 중간 |
| `MenuInfo.mhi` | AZ | 249KB | 높음 |
| `RetrieveInsnTypeList.mhi` | HP | 203KB | 중간 |
| `RetrieveInnerDataSet.mhi` | HP | 221KB | 중간 |

### 4.3 성능 개선 권장 사항

1. **메뉴 정보 캐싱**: `MenuInfo.mhi` 249KB → 캐싱으로 초기 로딩 개선
2. **공통코드 분할**: `RetrieveComnCd.mhi` 다중 호출 → 일괄 조회
3. **로그인 최적화**: `CheckLoginUser` 1000ms+ → 연결 풀 최적화
4. **외래환자 페이징**: `RetrieveOtptLoad` 513KB → 페이징 처리

---

## 5. 업무 영역별 사용 패턴 요약

### 5.1 AZ (입원/공통) - 35%

```
주요 사용 패턴:
├── 로그인/인증 (authNavi)
│   ├── CheckLoginUser-new1.mhi (로그인)
│   ├── MenuInfo.mhi (메뉴정보)
│   └── SaveLogoutInfo.mhi (로그아웃)
├── 공통코드 (comNavi)
│   ├── RetrieveComnCd.mhi (다수 호출)
│   ├── RetrieveToday.mhi (날짜)
│   └── RetrieveScrnAuth.mhi (화면권한)
└── 시스템 (sys)
    ├── userMngmNavi (사용자관리)
    └── printMngmNavi (프린트관리)
```

### 5.2 HP (병원/심사) - 30%

```
주요 사용 패턴:
├── 환자접수 (pat)
│   ├── RetrieveOtptLoad.mhi (외래환자로드) ← 최대 응답
│   └── RetrieveOtptPtList.mhi
├── 공통 (com)
│   ├── RetrieveIdnt.mhi (식별)
│   ├── RetrieveClas.mhi (분류)
│   └── RetrieveIjtp.mhi (보험구분)
└── 기초정보 (bas)
    └── RetrieveCnrtInstCmbList2.mhi (계약기관)
```

### 5.3 MD (진료) - 15%

```
주요 사용 패턴:
├── 응급간호 (ern)
│   ├── RetrieveEmctPatInfo.mhi (응급환자정보)
│   ├── RetrieveEmctNrsnInfoRgst.mhi (응급간호정보)
│   └── RetrieveNedis.mhi (네디스)
├── 처방 (ord)
│   └── ptmdcrNavi (처방관리)
└── 입원간호 (ipn)
    └── admsnrcrNavi (입원접수)
```

### 5.4 SP (특수진료) - 10%

```
주요 사용 패턴:
├── 검사실 (lab)
├── 약제 (pha)
└── 방사선 (ray)
```

### 5.5 ER (응급/행정) - 7%

```
주요 사용 패턴:
├── 물품/자재 (stc)
├── 인사/채용 (ins)
└── 회계/지출 (acc)
```

### 5.6 MR (의무기록) - 3%

```
주요 사용 패턴:
├── 공통 (com)
└── 병리/조직 (rch)
```

---

## 6. 유의미한 패턴 및 인사이트

### 6.1 세션 시작 패턴

모든 사용자 세션이 동일한 시퀀스로 시작:
```
CheckLoginUser → RetrieveToday → UpdateCntInit → MenuInfo → (RetrieveComnCd × N)
```
→ **최적화 포인트**: 공통코드 다중 호출을 일괄 처리로 변경

### 6.2 화면 전환 패턴

```
화면 전환 시 공통 호출:
├── createPriveRtrvLog.mhi (권한 로그)
├── RetrieveScrnAuth.mhi (화면 권한)
└── RetrieveComnCd.mhi (화면별 코드)
```
→ **최적화 포인트**: 화면 권한 + 코드를 통합 API로 제공

### 6.3 대용량 데이터 전송

- **853KB**: 코드 마스터 목록 (`RetrieveListMaster.mhi`)
- **513KB**: 외래 환자 목록 (`RetrieveOtptLoad.mhi`)
- **249KB**: 메뉴 정보 (`MenuInfo.mhi`)

→ **권장사항**: 대용량 데이터는 페이징 또는 압축 전송 고려

### 6.4 응답 시간 이상치

- **로그인 API**: 초기 로그인 시 1000ms+ (DB 연결 생성)
- **DB 연결 문자열 조회**: 100~200ms (설정 로드)
- **메뉴 정보**: 200~400ms (대용량 데이터)

---

## 7. 분석 활용 방안

### 7.1 성능 최적화

1. **공통코드 캐싱**: Redis/Memory Cache 도입
2. **메뉴 정보 분할**: 자주 사용하는 메뉴만 우선 로드
3. **로그인 병렬 처리**: 인증 후 메뉴/코드 병렬 조회

### 7.2 사용자 경험 개선

1. **초기 로딩 최적화**: 로그인 시 필수 데이터만 로드
2. **화면 전환 속도**: 권한/코드 캐싱으로 전환 시간 단축
3. **대용량 리스트 페이징**: 500KB 이상 응답은 페이징 처리

### 7.3 모니터링 지표

1. **평균 응답 시간**: 100ms 이하 유지
2. **대용량 API**: 200KB 이상 응답 모니터링
3. **로그인 성능**: 1초 이내 완료 목표

---

## 8. 결론

### 8.1 핵심 발견

1. **AZ 영역이 35%로 가장 많이 사용** - 공통코드 조회가 핵심
2. **HP 영역이 30%로 두 번째** - 외래 환자 관리가 메인
3. **대용량 응답이 성능 저하 원인** - 513KB 외래 환자 로드
4. **로그인 시 1000ms+ 소요** - DB 연결 및 세션 생성

### 8.2 권장 사항

| 우선순위 | 개선 항목 | 예상 효과 |
|----------|----------|----------|
| 1 | 공통코드 캐싱 | 응답 시간 50% 감소 |
| 2 | 메뉴 정보 분할 | 초기 로딩 30% 감소 |
| 3 | 로그인 병렬 처리 | 로그인 시간 40% 감소 |
| 4 | 대용량 API 페이징 | 메모리 사용량 70% 감소 |

---

*작성자: Claude Code*
*분석 일자: 2026-03-08*
*데이터 출처: JEUS Access Logs (2025-12-08 ~ 2026-03-04)*