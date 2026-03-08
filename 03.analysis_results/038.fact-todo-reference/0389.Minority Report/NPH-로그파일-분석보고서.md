# NPH 로그파일 분석 보고서

> **작성일**: 2026-03-08
> **분석 범위**: `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/` 하위 로그 파일
> **총 로그 파일 수**: 475개 (3.1GB+)

---

## 1. 로그 파일 분류

### 1.1 로그 유형별 분포

| 카테고리 | 위치 | 파일 수 | 크기 | 분석 가치 |
|----------|------|---------|------|----------|
| **JEUS Server Logs** | `bin/JEUS7/domains/.../logs/` | ~100개 | 3.1GB | ★★★★★ |
| **DevOn Application Logs** | `workspace/NPH_HIS/devonhome/logs/` | ~245개 | 3.1GB | ★★★★★ |
| **Servlet Access Logs** | `.../logs/servlet/` | ~50개 | - | ★★★★☆ |
| **Transaction Logs** | `.../tmlog/` | 12개 | - | ★★★☆☆ |
| **Batch Job Logs** | `.../batch-file-log/` | 8개 | - | ★★★★★ |
| **JVM Logs** | `.../logs/jvm.log` | 1개 | - | ★★☆☆☆ |

### 1.2 로그 파일 명명 규칙

#### DevOn Application Logs
```
{YYYYMMDD}{AM|PM}{type}.log

예: 20260304PMinfo.log, 20260304PMerr.log, 20260304PMdevon.log
```

| 타입 | 설명 |
|------|------|
| `info.log` | 일반 정보 로그 |
| `err.log` | 에러 로그 |
| `debug.log` | 디버그 로그 |
| `devon.log` | DevOn 프레임워크 로그 |
| `dbwrap.log` | DB 래퍼 로그 |

#### JEUS Server Logs
```
JeusServer.log             # 현재 로그
JeusServer_{YYYYMMDD}.log  # 일별 로그
access.log                 # 서블릿 접근 로그
access_{YYYYMMDD}.log      # 일별 접근 로그
```

#### Batch Job Logs
```
HP_BAT{코드}-{일련번호}.log      # 개별 배치 로그
batchLog                         # 배치 시스템 로그
devon-batch-syslog               # 배치 시스템 로그
```

---

## 2. 주요 로그 분석 결과

### 2.1 DevOn Application Logs (`devonhome/logs/`)

#### 구조
```
devonhome/logs/
├── 20251208AMdbwrap.log      # DB 래퍼 로그 (354KB)
├── 20251208AMdebug.log       # 디버그 로그 (862KB)
├── 20251208AMerr.log         # 에러 로그 (969B)
├── 20251208AMinfo.log        # 정보 로그 (2.3KB)
├── ...
└── batch-file-log/
    ├── batchLog              # 배치 시스템 로그
    ├── devon-batch-syslog    # 배치 시스템 로그
    └── 2026-02-02/           # 날짜별 배치 로그
```

#### 주요 발견 내용

**비동기 작업 핸들러 등록**:
```
[AsyncWorkContextListener] Registered Handler: LST -> com.rest.external.lst.LstHandler
[AsyncWorkContextListener] Registered Handler: POLNET -> com.rest.external.polnet.PolNetHandler
```

**외부 API 연동**:
- `LST` (장기이식등록관리시스템): `api.lst.go.kr`
- `POLNET` (경찰청 네트워크): 외부 연동 핸들러

### 2.2 JEUS Server Logs

#### 주요 정보
```
Version: JEUS 7.0 (Fix#4) (7.0.0.4-b214)
Java: 1.8.0_181-1-ojdkbuild-b13
Domain: jeus_domain
Server: adminServer
Base Port: 9736
Host: JJU (10.60.215.14)
```

#### 서블릿 초기화 순서
```
1. NphBatchChannelServlet
2. NphCacheServlet
3. TprServlet
4. MiplatformServlet (핵심 진입점)
5. PropertiesInitializerServlet
6. ServletContainer (JAX-RS)
7. ServletDOMConfigurator (Log4j)
8. ServletInitializer (InnoRules Client)
```

### 2.3 Servlet Access Logs (API 호출 추적)

#### 로그 형식
```
{IP} [{날짜:시간}] "{메서드} {URL} {프로토콜}" {상태코드} {응답크기} {처리시간(ms)}
```

#### 주요 API 호출 패턴
```
127.0.0.1 [04/Mar/2026:13:17:20 +0900] "POST /NPH_HIS/az/bizcom/authNavi/CheckLoginUser-new1.mhi HTTP/1.1" 200 3076 1163
127.0.0.1 [04/Mar/2026:13:17:20 +0900] "POST /NPH_HIS/az/bizcom/comNavi/RetrieveToday.mhi HTTP/1.1" 200 243 95
127.0.0.1 [04/Mar/2026:13:17:27 +0900] "POST /NPH_HIS/az/bizcom/authNavi/MenuInfo.mhi HTTP/1.1" 200 249540 375
```

#### 로그인 후 초기화 시퀀스
1. `CheckLoginUser-new1.mhi` - 로그인 인증
2. `RetrieveToday.mhi` - 오늘 날짜 조회
3. `UpdateCntInit1.mhi` - 카운트 초기화
4. `RetrieveDbmsConnectionString.mhi` - DB 연결 문자열
5. `MenuInfo.mhi` - 메뉴 정보 (대용량: 249KB)
6. `RetrieveMenuGrpCd.mhi` - 메뉴 그룹 코드
7. `MenuOnload.mhi` - 메뉴 로드
8. `RetrieveComnCd.mhi` - 공통 코드 (다수 호출)

#### 대용량 응답 API
| API | 응답 크기 | 설명 |
|-----|----------|------|
| `RetrieveInnerDataSet.mhi` | 221KB | 내부 데이터셋 |
| `RetrieveInsnTypeList.mhi` | 203KB | 보험 유형 목록 |
| `MenuInfo.mhi` | 249KB | 메뉴 정보 |

### 2.4 Batch Job Logs

#### 배치 작업 구조
```
HP_BAT01206B (작업 그룹)
└── HP_BAT01206B01 (개별 작업)
    ├── 초기화 단계
    ├── 데이터 조회
    ├── 파일 처리
    ├── 작업 실행
    └── 종료
```

#### 발견된 에러
```
ORA-00947: not enough values (값이 충분하지 않음)
위치: OtptRcptEC.updateAdmsRcptReClcl (line 796)
호출 체인: CreatePreRevwAmtCalc → StrtAdmsClclUC → HpComnUC → OtptRcptEC
```

#### 배치 실행 정보
```json
{
  "job": {
    "chosNo": "20260202E00001",
    "clclDvsn": "31",
    "userId": "8990004",
    "insFlag": "1",
    "medStrDy": "20260202",
    "dschPrarDy": "20260202",
    "medDvsn": "I"
  }
}
```

### 2.5 Transaction Logs (tmlog)

#### 파일 구조
```
tmlog/
├── _adminServer_LOCATION_0_9736_10_60_203_164_14/
│   ├── jeusres_1.log
│   ├── jeusres_2.log
│   ├── jeustx_1.log
│   └── jeustx_2.log
└── _adminServer_LOCATION_0_9736_10_60_215_14_14/
    └── ...
```

---

## 3. 분석 가치 평가

### 3.1 높은 가치 (★★★★★)

| 로그 유형 | 활용 목적 |
|-----------|----------|
| **Access Logs** | API 호출 패턴 분석, 응답 시간 모니터링, 사용자 행동 추적 |
| **Batch Logs** | 배치 작업 흐름 파악, 에러 원인 분석, 실행 파라미터 확인 |
| **DevOn Info Logs** | 애플리케이션 시작/종료, 핸들러 등록, 외부 연동 확인 |
| **JEUS Server Logs** | 서버 생명주기, 서블릿 초기화 순서, 클러스터 상태 |

### 3.2 중간 가치 (★★★☆☆)

| 로그 유형 | 활용 목적 |
|-----------|----------|
| **Transaction Logs** | 분산 트랜잭션 추적, 리소스 관리 분석 |
| **Error Logs** | 에러 발생 패턴, 스택 트레이스 분석 |
| **Debug Logs** | 상세 디버깅 (용량 큼) |

### 3.3 낮은 가치 (★★☆☆☆)

| 로그 유형 | 활용 목적 |
|-----------|----------|
| **JVM Logs** | JVM 상태, GC 정보 |

### 3.4 DBWrap Logs (특별 가치 - ★★★★★)

| 로그 유형 | 활용 목적 |
|-----------|----------|
| **DBWrap Logs** | **실제 SQL 쿼리 추출, 실행 시간 분석, 테이블 구조 파악** |

**DBWrap 로그 핵심 가치**:
```
NO_ID_-963878643=PSTMT.EQ Laptime : 31
NO_ID_-963878643=        SELECT A.PID
NO_ID_-963878643=             , A.INFO_DVSN
NO_ID_-963878643=             , B.COMN_NM INFO_DVSN_NM
NO_ID_-963878643=          FROM HPPAHPTET A
NO_ID_-963878643=             , AZCMMCMCD B
NO_ID_-963878643=         WHERE A.INFO_DVSN = B.COMN_CD
NO_ID_-963878643=Result : select completed.
```

**확인된 테이블**:
- `HPPAHPTET` - 환자 정보?
- `AZCMMCMCD` - 공통 코드
- `HPPAHPTBS` - 환자 기본
- `AZCMMUSER` - 사용자 마스터
- `AZCMMDEPT` - 부서 마스터

**SQL 실행 시간 (Laptime)**: 밀리초 단위 측정 가능

---

## 4. 분석에 유의미한 로그 파일 목록

### 4.1 API 호출 패턴 분석용

```
bin/JEUS7/domains/jeus_domain/servers/adminServer/logs/servlet/access.log
bin/JEUS7/domains/jeus_domain/servers/adminServer/logs/servlet/access_*.log
```

**분석 가능 항목**:
- 사용자별 API 호출 빈도
- 응답 시간 분석 (느린 API 식별)
- 대용량 응답 API 식별
- 로그인 패턴 분석

### 4.2 배치 작업 분석용

```
workspace/NPH_HIS/devonhome/logs/batch-file-log/batchLog
workspace/NPH_HIS/devonhome/logs/batch-file-log/devon-batch-syslog
workspace/NPH_HIS/devonhome/logs/batch-file-log/2026-02-02/HP_BAT*.log
```

**분석 가능 항목**:
- 배치 작업 실행 체인
- 에러 발생 위치 및 원인
- 작업 파라미터 구조
- 실행 시간 패턴

### 4.3 시스템 아키텍처 분석용

```
bin/JEUS7/domains/jeus_domain/servers/adminServer/logs/JeusServer.log
workspace/NPH_HIS/devonhome/logs/*info.log
```

**분석 가능 항목**:
- 서블릿 초기화 순서
- 외부 연동 핸들러 (LST, POLNET)
- 서버 설정 정보
- 스레드 풀 구성

### 4.4 에러 분석용

```
workspace/NPH_HIS/devonhome/logs/*err.log
workspace/NPH_HIS/devonhome/logs/batch-file-log/batchLog
```

**분석 가능 항목**:
- SQL 에러 (ORA-XXXXX)
- DevonException 스택 트레이스
- 배치 작업 실패 원인

### 4.5 SQL 쿼리 분석용 (DBWrap Logs) - ★★★★★

```
workspace/NPH_HIS/devonhome/logs/*dbwrap.log
```

**분석 가능 항목**:
- 실제 실행된 SQL 쿼리 추출
- 쿼리 실행 시간 (Laptime)
- 테이블 조인 패턴
- 쿼리 ID 기반 추적

**샘플 쿼리 구조**:
```sql
-- 환자 정보 조회 (HPPAHPTET)
SELECT A.PID, A.INFO_DVSN, B.COMN_NM INFO_DVSN_NM
FROM HPPAHPTET A, AZCMMCMCD B
WHERE A.INFO_DVSN = B.COMN_CD AND B.CLSF_CD = 'HP153'

-- 공통 코드 전체 조회 (AZCMMCMCD)
SELECT CLSF_CD, COMN_CD, COMN_CD1, COMN_CD2, COMN_CD3, COMN_NM
FROM AZCMMCMCD WHERE DEL_YN = 'N' ORDER BY CLSF_CD

-- 사용자 목록 (AZCMMUSER)
SELECT USID, USER_NM, REVERSE(USID) RUSID
FROM AZCMMUSER ORDER BY RUSID

-- 부서 목록 (AZCMMDEPT)
SELECT DEPT_CD, DEPT_STR_DY, DEPT_END_DY, DEPT_NM, ENGL_CD, ENGL_NM
FROM AZCMMDEPT

-- 로그인 사용자 정보
SELECT USID, USER_NM, USER_ENGL_NM, PW, OCTY_CD, OCPS_CD, OCGR_CD, SMCR_YN
FROM AZCMMUSER WHERE ...
```

---

## 5. 추천 분석 방법

### 5.1 API 호출 패턴 분석

```bash
# 가장 많이 호출되는 API 추출
grep -oP 'POST /\S+\.mhi' access*.log | sort | uniq -c | sort -rn | head -20

# 응답 시간이 느린 API (1000ms 이상)
awk '$NF > 1000 {print $0}' access*.log | head -50

# 대용량 응답 API (100KB 이상)
awk '$(NF-1) > 100000 {print $(NF-1), $7}' access*.log | sort -rn | head -20
```

### 5.2 에러 패턴 분석

```bash
# ORA 에러 추출
grep -oP 'ORA-\d+' batchLog err*.log | sort | uniq -c

# 스택 트레이스에서 클래스 추출
grep -oP 'at [a-zA-Z0-9.]+' batchLog | sort | uniq -c | sort -rn | head -20
```

### 5.3 시계열 분석

```bash
# 시간대별 요청 수
grep -oP '\[\d{2}/\w{3}/\d{4}:\d{2}:' access*.log | cut -d: -f2 | sort | uniq -c
```

---

## 6. 로그 기간 및 날짜 범위

| 로그 유형 | 시작일 | 종료일 |
|-----------|--------|--------|
| JEUS Server | 2025-12-08 | 2026-03-04 |
| DevOn Logs | 2025-12-08 | 2026-03-04 |
| Access Logs | 2025-12-08 | 2026-03-04 |
| Batch Logs | 2026-01-19 | 2026-02-02 |

---

## 7. 주요 발견 사항

### 7.1 시스템 구성

```
Server: JJU (10.60.215.14)
WAS: JEUS 7.0 Fix#4
Java: OpenJDK 1.8.0_181
Domain: jeus_domain
Port: 9736 (base), 8088 (HTTP), 9941
```

### 7.2 외부 연동

| 핸들러 | URL | 목적 |
|--------|-----|------|
| LST | api.lst.go.kr | 장기이식등록관리시스템 |
| POLNET | - | 경찰청 네트워크 |

### 7.3 배치 작업 명명 규칙

```
HP_BAT{업무코드}{일련번호}

HP: 병원관리/심사 영역
BAT: 배치 작업
01206B: 업무 코드
83/84/85/86: 실행 일련번호
```

### 7.4 확인된 에러

| 에러 코드 | 위치 | 원인 |
|-----------|------|------|
| ORA-00947 | OtptRcptEC.updateAdmsRcptReClcl | SQL 값 부족 |

---

## 8. 다음 단계 권장 사항

1. **Access Log 심층 분석**: API 호출 빈도, 응답 시간 분포, 사용자 패턴
2. **Batch Job 에러 수정**: ORA-00947 에러 원인 파악 및 수정
3. **성능 저하 API 식별**: 응답 시간 > 1초 API 목록화
4. **대용량 API 최적화**: 응답 크기 > 100KB API 검토

---

*작성자: Claude Code*
*분석 일자: 2026-03-08*