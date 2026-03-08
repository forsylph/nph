# 운영 대시보드 샘플표 (codex GPT5.4)

## 1. 목적

이 문서는 로그 기반 운영 분석에서 바로 써먹을 수 있는 대시보드 항목을 분리해 정리한 보조 문서다.

- 기준 방법론 문서:
  - [로그-기반-시스템-분석-방법론 (codex GPT5.4).md](./로그-기반-시스템-분석-방법론%20%28codex%20GPT5.4%29.md)
- 선행 사례 문서:
  - [NPH-로그파일-분석보고서.md](./NPH-%EB%A1%9C%EA%B7%B8%ED%8C%8C%EC%9D%BC-%EB%B6%84%EC%84%9D%EB%B3%B4%EA%B3%A0%EC%84%9C.md)
  - [NPH-Access-Log-사용패턴분석.md](./NPH-Access-Log-%EC%82%AC%EC%9A%A9%ED%8C%A8%ED%84%B4%EB%B6%84%EC%84%9D.md)

## 2. 기본 원칙

이 시스템의 로그는 많지만, 업무 의미로 바로 번역되지는 않는다. 그래서 대시보드는 단순한 카운트보다 아래 3가지를 같이 보여줘야 한다.

1. 무엇이 호출됐는가
2. 그 호출이 어떤 업무 의미를 가지는가
3. 느림/실패/재시도 같은 운영 위험이 있는가

## 3. 온라인 운영 대시보드

### 3.1 핵심 표

| 지표 | 의미 | 실제 사례 |
|---|---|---|
| 로그인 성공/실패 수 | 인증 이상 감지 | `CheckLoginUser-new1.mhi` |
| `.mhi` 호출 상위 20개 | 자주 쓰는 기능 파악 | `RetrieveToday.mhi`, `RetrieveComnCd.mhi` |
| 평균 응답시간 상위 20개 | 느린 화면 탐지 | `MenuInfo.mhi`, `RetrieveListMaster.mhi` |
| 실패 `.mhi` 상위 10개 | 오류 집중 지점 파악 | 실패 응답 코드 기준 |
| 장시간 SQL 상위 N개 | 병목 쿼리 탐지 | `dbwrap.log` 기준 |
| 평균 응답크기 상위 20개 | 대용량 데이터 전송 탐지 | `MenuInfo.mhi`, `RetrieveOtptLoad.mhi` |
| 세션당 공통 조회 횟수 | 비효율적인 반복 호출 탐지 | `RetrieveComnCd.mhi` |

### 3.2 예시 카드

- `CheckLoginUser-new1.mhi` 호출 수
- `RetrieveToday.mhi` 평균 응답시간
- `UpdateCntInit1.mhi` 실패율
- `RetrieveDbmsConnectionString.mhi` 호출 빈도
- `MenuInfo.mhi` 평균 응답크기
- `RetrieveComnCd.mhi` 세션당 호출 수

### 3.3 해석 포인트

- 로그인은 단일 API 1건이 아니라 초기 조회 시퀀스의 시작점이다.
- 공통 코드/메뉴/기초 데이터 조회가 실제 체감 성능에 큰 영향을 준다.
- 대형 업무 화면만 최적화해도 충분하지 않다.

## 4. 배치 운영 대시보드

### 4.1 핵심 표

| 지표 | 의미 | 실제 사례 |
|---|---|---|
| JobGroup 실행 수 | 배치 부하 규모 | `HP_BAT01206B` |
| 실패 JobGroup 수 | 운영 위험 | 실행 상태 전이 로그 |
| 최근 실패 Job ID | 즉시 점검 대상 | `HP_BAT01206B01` |
| override 실행 횟수 | 수동 개입 정도 | `override될 예정입니다` |
| 평균 실행시간 상위 JobGroup | 병목 배치 | 그룹별 로그 집계 |
| 최근 예외 유형 | 장애 패턴 | `BatchStackedException`, `JobGroupExecutionException` |
| 실행 주체별 실행 건수 | 수동/운영자介入 추적 | `EXECUTOR_ID` |

### 4.2 예시 카드

- `HP_BAT01206B` 최근 7일 실행 수
- `HP_BAT01206B01` 최근 실패 횟수
- override 사용 상위 JobGroup
- `BatchStackedException` 발생 추세

### 4.3 해석 포인트

- 배치는 `scheduler xml`만 보면 안 된다.
- 실제 운영은 `batchMgr UI + DB 메타데이터 + BatchExecutor.cmd + 실행 로그` 조합으로 읽어야 한다.
- 같은 JobGroup이라도 `override`, `exec count`, 하위 `JOB_ID`를 분리해서 봐야 한다.

## 5. 외부 연계 운영 대시보드

### 5.1 핵심 표

| 지표 | 의미 | 실제 사례 |
|---|---|---|
| LST/POLNET 초기화 성공 여부 | 기동 정상성 | `handler registered` |
| 인증서 로드 성공 여부 | 연계 준비 상태 | `HttpClientUtil Loaded Cert` |
| 외부 연계 호출 성공률 | 연계 품질 | endpoint 호출 로그 |
| timeout 횟수 | 네트워크 문제 | 실패/timeout 로그 |
| endpoint별 평균 응답시간 | 연계 성능 | REST/SOAP/FTP 호출 |

### 5.2 예시 카드

- `HttpClientUtil Loaded Cert`
- `LST handler registered`
- `POLNET handler registered`
- 최근 실패 endpoint 수

### 5.3 해석 포인트

- 이 시스템은 외부 연계가 적지 않다.
- 기동 로그에서 인증서/핸들러 등록이 정상인지 먼저 봐야 한다.
- 온라인 화면 장애처럼 보여도 실제 원인은 외부 연계 지연일 수 있다.

## 6. 시간대/활동량 대시보드

기존 Access Log 분석 기준으로 시간대 분포도 운영 판단에 중요하다.

| 시간대 | 의미 |
|---|---|
| `08:00~09:00` | 로그인 집중, 메뉴 로드 |
| `09:00~12:00` | 오전 업무 피크 |
| `14:00~17:00` | 오후 업무 피크 |

추가하면 좋은 항목:

- 시간대별 `.mhi` 호출 수
- 시간대별 로그인 지연
- 시간대별 대용량 응답 API 상위 N개
- 시간대별 외부 연계 실패율

## 7. 실제 있었던 일로 보는 적용 예시

### 7.1 2026-03-04: 사이트는 살아 있고 일반 운영은 진행됨

로그에서 확인된 것:

- `AsyncWorkContextListener Initializing context`
- `WorkQueueManager Max Queue Capacity set to: 100`
- `LST handler`, `POLNET handler` 등록
- `HttpClientUtil` 인증서 로드
- `CheckLoginUser-new1.mhi`
- `RetrieveToday.mhi`
- `RetrieveDbmsConnectionString.mhi`

이 날 대시보드 해석:

- 온라인 운영 보드는 정상 신호가 많아야 한다.
- 배치 보드보다 기동/연계 보드가 더 중요하다.
- 로그인/공통 조회가 평소보다 느리면 오전 업무 시작 지연으로 이어질 수 있다.

### 7.2 2026-02-02: 배치 JobGroup이 실제로 돌았다

로그에서 확인된 것:

- `HP_BAT01206B-86.log`
- `HP_BAT01206B-86-HP_BAT01206B01.log`
- `동기 실행`
- `override될 예정입니다`
- `BatchStackedException`

이 날 대시보드 해석:

- 배치 보드에서 `HP_BAT01206B`를 중심으로 봐야 한다.
- `override 사용 빈도`와 `하위 JOB_ID 실패율`이 중요하다.
- 같은 날 온라인 보드보다 배치 보드 우선순위가 높다.

## 8. 운영자가 바로 볼 추천 보드 순서

1. 온라인 운영 보드
2. 외부 연계 보드
3. 배치 운영 보드
4. 시간대/활동량 보드

이 순서는 최근 로그 성격과 실제 시스템 사용 흐름을 같이 반영한 것이다.

## 9. Minority Report 관점 결론

이 시스템은 로그가 없는 시스템이 아니라, 로그를 업무 의미로 묶어주는 시점이 부족한 시스템이다.

따라서 대시보드는 “많이 찍는 것”보다 아래를 먼저 해결해야 한다.

- `.mhi`와 업무 의미 연결
- `GROUP_ID / JOB_ID`와 운영 의미 연결
- 외부 연계 핸들러/인증서 초기화와 기동 정상성 연결

이 세 가지를 먼저 잡아야 로그가 운영 도구가 된다.
