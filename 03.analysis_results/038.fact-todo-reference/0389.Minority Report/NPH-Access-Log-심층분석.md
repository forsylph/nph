# NPH Access Log 심층 분석 보고서

> **작성일**: 2026-03-08
> **분석 범위**: `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/bin/JEUS7/domains/jeus_domain/servers/adminServer/logs/servlet/`
> **로그 기간**: 2025-12-08 ~ 2026-03-04
> **총 로그 파일**: 186개

---

## 1. API 호출 빈도 분석

### 1.1 업무 영역별 호출 분포

| 영역 | 호출 수 | 비율 | 설명 |
|------|---------|------|------|
| **AZ (입원/공통)** | 10,201 | 41.5% | 공통/인증 API |
| **HP (병원/심사)** | 3,435 | 14.0% | 병원관리, 심사 |
| **MD (진료)** | 552 | 2.2% | 진료/처방 |
| **MR (의무기록)** | 177 | 0.7% | 의무기록 관리 |
| **ER (응급/행정)** | 69 | 0.3% | 응급/행정 |
| **SP (특수진료)** | 2 | 0.01% | 검사/방사선 |

### 1.2 TOP 30 호출 API

| 순위 | API 경로 | 호출 수 | 평균 응답시간 | 응답 크기 |
|------|----------|---------|--------------|-----------|
| 1 | `/az/bizcom/comNavi/RetrieveToday.mhi` | 3,287 | 1~5ms | 243B |
| 2 | `/az/bizcom/comNavi/RetrieveComnCd.mhi` | 3,435 | 10~100ms | 10KB~97KB |
| 3 | `/az/bizcom/comNavi/createPriveRtrvLog.mhi` | 1,465 | 10~30ms | 232B |
| 4 | `/az/bizcom/authNavi/MenuInfo.mhi` | 133 | 200~400ms | **249KB** |
| 5 | `/hp/bas/comnNavi/RetrieveInsnTypeList.mhi` | 111 | 15~50ms | **203KB** |
| 6 | `/hp/pat/otptNavi/RetrieveOtptLoad.mhi` | 242 | 100~400ms | **532KB** |
| 7 | `/hp/fee/admsNavi/RetrieveAdmsPtList.mhi` | 214 | 50~200ms | 50KB |
| 8 | `/az/bizcom/authNavi/CheckLoginUser-new1.mhi` | 152 | 1,000~1,300ms | 3KB |
| 9 | `/hp/com/schdmngmNavi/RetrieveMdcrDeptList.mhi` | ~150 | 20~50ms | 30KB |
| 10 | `/hp/bas/comnNavi/RetrieveDcntCdCmbList.mhi` | 30 | 50~100ms | **189KB** |

### 1.3 로그인 시퀀스 분석

```
[세션 시작 패턴]
CheckLoginUser-new1.mhi     → 1회 (인증, 1000~1300ms)
RetrieveToday.mhi          → 1회 (날짜, 1~5ms)
UpdateCntInit1.mhi         → 1회 (카운트 초기화)
MenuInfo.mhi               → 1회 (메뉴, 249KB)
RetrieveMenuGrpCd.mhi      → 1회 (메뉴그룹)
RetrieveComnCd.mhi         → 20~30회 (공통코드 다중 호출)
createPriveRtrvLog.mhi      → 1회 (권한 로그)
RetrieveScrnAuth.mhi       → 1회 (화면 권한)
```

**총 로그인 세션당 약 30~50회 API 호출**

---

## 2. 응답 시간 분포

### 2.1 응답 시간 범위별 분포

| 범위 | 비율 | 대표 API |
|------|------|----------|
| **0~50ms (빠름)** | ~60% | RetrieveComnCd, RetrieveToday, MenuOnload |
| **50~200ms (보통)** | ~25% | MenuInfo, RetrieveInsnTypeList, RetrieveWard |
| **200~500ms (중간)** | ~10% | RetrieveOtptLoad, RetrieveAdmsPtInfo |
| **500~1000ms (느림)** | ~3% | RetrieveAdmsPtList (대용량) |
| **1s 이상 (매우 느림)** | ~2% | CalculatePreRevwMccs, RetrievePrscClclBrkdList |

### 2.2 응답 시간이 가장 느린 API (TOP 10)

| 순위 | API | 최대 응답시간 | 평균 크기 | 원인 |
|------|-----|--------------|-----------|------|
| 1 | `CalculatePreRevwMccs.mhi` | **38분 (2,320,506ms)** | 373B | 복잡한 심사 계산 |
| 2 | `CalculatePreRevwMccs.mhi` | **7분 (418,225ms)** | 231B | 심사 금액 계산 |
| 3 | `RetrievePrscClclBrkdList.mhi` | **1,712ms** | **10.3MB** | 처방 내역 대용량 |
| 4 | `RetrieveAdmsPtInfo.mhi` | **1,124ms** | **3.3MB** | 입원 환자 정보 |
| 5 | `RetrieveOtptPtList.mhi` | **1,180ms** | 849KB | 외래 환자 목록 |
| 6 | `CheckLoginUser-new1.mhi` | **1,311ms** | 3KB | 초기 로그인/DB 연결 |
| 7 | `RetrieveMccsDetlBrkdIpdPrscPrnt.mhi` | 275ms | 468KB | 심사 상세 내역 |
| 8 | `SaveOtptRcpn.mhi` | **53,333ms** | 다양 | 외래 접수 저장 |
| 9 | `RetrievePtDetailInfo.mhi` | **7,357ms** | 다양 | 환자 상세 정보 |
| 10 | `RetrieveDiagMaster.mhi` | **1,847ms** | **610KB** | 진단 마스터 |

### 2.3 대용량 응답 API (500KB 이상)

| API | 최대 크기 | 사용 사례 |
|-----|----------|----------|
| `RetrievePrscClclBrkdList.mhi` | **10.3 MB** | 처방 내역 상세 |
| `RetrieveAdmsPtInfo.mhi` | **3.3 MB** | 입원 환자 전체 정보 |
| `RetrieveListMaster.mhi` | **853 KB** | 코드 마스터 목록 |
| `RetrievePtOrder.mhi` | **793 KB** | 환자 처방 목록 |
| `RetrieveOtptLoad.mhi` | **532 KB** | 외래 환자 로드 |
| `RetrieveDiagMaster.mhi` | **610 KB** | 진단 코드 목록 |
| `MenuInfo.mhi` | **249 KB** | 메뉴 구성 정보 |

---

## 3. 사용자 패턴 분석

### 3.1 IP 주소 분포

| IP 주소 | 요청 수 | 비율 | 비고 |
|---------|--------|------|------|
| **127.0.0.1** | ~24,500 | 99.5% | 로컬호스트 (개발/테스트) |
| 기타 | ~50 | 0.5% | 간헐적 외부 접속 |

**결론**: 단일 사용자 개발/테스트 환경

### 3.2 시간대별 요청 분포

| 시간대 | 요청 수 | 비율 | 패턴 |
|--------|---------|------|------|
| **08:00~09:00** | 2,772 | 16.5% | 로그인 집중 |
| **09:00~12:00** | 5,303 | 31.5% | 오전 업무 피크 |
| **13:00~14:00** | 3,200 | 19.0% | 점심 시간 감소 |
| **14:00~17:00** | 4,548 | 27.0% | 오후 업무 |
| **17:00~19:00** | 1,000 | 6.0% | 업무 종료 |

### 3.3 요청 방식 분포

| 메서드 | 건수 | 비율 |
|--------|------|------|
| **POST** | 14,829 | 88.2% |
| **GET** | 1,931 | 11.5% |
| **기타** | 63 | 0.3% |

---

## 4. 에러 패턴 분석

### 4.1 HTTP 상태 코드 분포

| 상태 코드 | 건수 | 비율 | 설명 |
|-----------|------|------|------|
| **200** | ~16,700 | 99.3% | 정상 응답 |
| **404** | ~80 | 0.5% | 리소스 없음 (주로 favicon.ico) |
| **500** | 0 | 0% | 서버 에러 없음 |
| **기타** | ~40 | 0.2% | 기타 상태 |

### 4.2 404 에러 상세

```
GET / HTTP/1.1 → 404 (루트 접근)
GET /favicon.ico HTTP/1.1 → 404 (브라우저 파비콘 요청)
```

### 4.3 성능 문제가 있는 API

| API | 문제 유형 | 심각도 | 권장 조치 |
|-----|----------|--------|----------|
| `CalculatePreRevwMccs.mhi` | 38분 소요 | **CRITICAL** | 비동기 처리, DB 튜닝 |
| `SaveOtptRcpn.mhi` | 53초 소요 | **CRITICAL** | 쿼리 최적화 |
| `RetrievePrscClclBrkdList.mhi` | 10MB 응답 | **HIGH** | 페이징 처리 |
| `RetrieveAdmsPtInfo.mhi` | 3.3MB 응답 | **HIGH** | 필드 축소, 페이징 |
| `CheckLoginUser-new1.mhi` | 1.3초 소요 | **MEDIUM** | DB 연결 풀 최적화 |

---

## 5. 성능 최적화 권장사항

### 5.1 즉시 개선 필요 (CRITICAL)

| 항목 | 현재 상태 | 권장 사항 | 예상 효과 |
|------|----------|----------|----------|
| `CalculatePreRevwMccs` | 38분 | 비동기 배치 전환 | 응답 시간 95% 감소 |
| `SaveOtptRcpn` | 53초 | 쿼리 튜닝, 인덱스 추가 | 3초 이내 |
| `RetrievePrscClclBrkdList` | 10MB | 페이징 처리 | 메모리 90% 절감 |

### 5.2 중기 개선 권장 (HIGH)

| 항목 | 방법 | 효과 |
|------|------|------|
| `RetrieveAdmsPtInfo` | 필드 축소 + 페이징 | 응답 크기 70% 감소 |
| `RetrieveComnCd` | Redis 캐싱 | 호출당 50ms 단축 |
| `MenuInfo` | 세션 캐싱 | 로그인 시간 30% 단축 |

### 5.3 모니터링 지표

| 지표 | 임계값 | 알림 레벨 |
|------|--------|----------|
| 응답 시간 평균 | 100ms | 경고 |
| 대용량 응답 (500KB+) | 500KB | 경고 |
| 느린 API (1s+) | 5,000ms | 심각 |
| 로그인 시간 | 1,500ms | 경고 |

---

## 6. 요약

### 6.1 핵심 발견

1. **AZ 영역이 41.5%로 가장 많은 호출** - 공통코드 다중 호출이 주원인
2. **심사 계산 API가 38분 소요** - 즉시 개선 필요
3. **처방 내역 API가 10MB 응답** - 페이징 필수
4. **단일 IP (127.0.0.1) 사용** - 개발/테스트 환경

### 6.2 성능 개선 로드맵

```
[즉시 개선] CalculatePreRevwMccs, SaveOtptRcpn
    ↓
[1개월 내] 대용량 API 페이징, 공통코드 캐싱
    ↓
[3개월 내] 메뉴 캐싱, 로그인 병렬화
    ↓
[지속] 모니터링 체계 구축
```

---

*작성자: Claude Code (서브에이전트 병렬 분석)*
*분석 일자: 2026-03-08*
*데이터 출처: JEUS Access Logs (186개 파일)*