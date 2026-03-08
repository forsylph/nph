# 운영·장애대응 로그 관점 분석 (codex GPT5.4)

## 1. 목적
이 문서는 최근 확인한 사이트 로그와 배치 로그를 바탕으로, 이 시스템의 운영 분위기와 로그 관측성 수준을 별도 시각에서 정리한 Minority Report다.

핵심 목적은 아래 세 가지다.

1. 로그를 통해 이 시스템이 어떤 운영 철학으로 굴러가는지 읽는다.
2. 장애 대응 관점에서 현재 로그의 강점과 약점을 정리한다.
3. 나중에 실제 운영 개선이나 로깅 개선 작업을 할 때 바로 참고할 수 있는 관찰 포인트를 남긴다.

## 2. 이번 판단의 근거 범위

### 2.1 직접 본 로그 범위
- `devonhome/logs/20260304PMinfo.log`
- `devonhome/logs/20260304PMdebug.log`
- `devonhome/logs/20260304PMdbwrap.log`
- `devonhome/logs/20260304PMdevon.log`
- `devonhome/logs/batch-file-log/devon-batch-syslog`
- `devonhome/logs/batch-file-log/batchLog`
- `devonhome/logs/batch-file-log/2026-02-02/HP_BAT01206B-86.log`
- `devonhome/logs/batch-file-log/2026-02-02/HP_BAT01206B-86-HP_BAT01206B01.log`

### 2.2 함께 본 코드/설정
- [BatchExecutor.cmd](/home/forsylph/ollama/N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/cmd/BatchExecutor.cmd)
- [devon-batch-scheduler.xml](/home/forsylph/ollama/N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/devonhome_batch/conf/product/devon-batch-scheduler.xml)
- [navigation.xml](/home/forsylph/ollama/N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/devonhome/navigation/batch/navigation.xml)
- [BatchInfoUC.java](/home/forsylph/ollama/N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/src/nph/his/az/com/uc/BatchInfoUC.java)
- [Draft.버전관리-배포관리-드래프트.md](/home/forsylph/ollama/NPH/03.analysis_results/032.framework-core/0325.Version%20Control%20and%20Deployment/Draft.%EB%B2%84%EC%A0%84%EA%B4%80%EB%A6%AC-%EB%B0%B0%ED%8F%AC%EA%B4%80%EB%A6%AC-%EB%93%9C%EB%9E%98%ED%94%84%ED%8A%B8.md)

## 3. 로그가 보여주는 시스템 분위기

### 3.1 운영은 실제로 살아 있고, 사람 손을 전제로 설계된 시스템이다
로그를 보면 이 시스템은 "자동화가 모든 것을 숨겨주는 서비스형 플랫폼"이 아니다.

오히려 아래 특징이 더 강하다.

- 운영자가 내부 기능을 많이 알아야 한다.
- 관리 화면이 시스템 안에 깊게 들어와 있다.
- 배치도 완전 자동보다 운영자 관리형에 가깝다.
- 외부 연계가 많아 기동 직후 초기화 로그가 중요하다.

즉 이 시스템은 `세련된 자동화 플랫폼`보다는 `업무를 오래 버텨온 운영형 시스템`에 가깝다.

### 3.2 사이트 로그는 "일반 사용자 사용"보다 "초기화와 운영 흔적"이 더 잘 보인다
`2026-03-04` 로그를 보면 가장 먼저 보이는 건 화려한 업무 기능이 아니라 아래 흐름이다.

- 컨텍스트 초기화
- WorkQueue 초기화
- 외부 핸들러 등록
- 인증서 로드
- 공통 캐시 준비
- 로그인과 공통 조회

즉 이 시스템은 사용자의 행위보다 먼저 "운영을 위한 준비 상태"가 로그에 강하게 남는다.

이건 장점과 단점이 같이 있다.

- 장점: 기동 이상 여부를 빨리 알 수 있다.
- 단점: 실제 사용자가 무엇을 했는지는 상대적으로 약하게 남는다.

### 3.3 배치는 현대적인 스케줄러형보다 운영 관리형이다
배치 로그와 코드/설정을 같이 보면 이 시스템의 배치는 아래로 읽힌다.

```mermaid
flowchart LR
    UI[batchMgr UI]
    DB[DB metadata]
    CMD[BatchExecutor.cmd]
    LOG[batch logs]

    UI --> DB --> CMD --> LOG
```

중요한 건 `devon-batch-scheduler.xml` 하나가 아니라:

- `batchMgr`
- `GROUP_ID`
- `JOB_ID`
- `OVERRIDE`
- `EXEC_COUNT`
- 실행 로그

이 조합이다.

즉 이 배치는 "예약표가 알아서 돈다"보다 "운영자가 관리하고 실행기가 수행한다"에 더 가깝다.

## 4. 장애 대응 관점에서 본 강점

### 4.1 계층 흔적은 꽤 잘 남는다
현재 로그만으로도 아래 계층은 상당히 읽힌다.

- 사이트 기동 흔적
- 외부 연계 초기화 흔적
- `.mhi` 호출 흔적
- SQL 실행 흔적
- 배치 JobGroup/Job 상태 전이 흔적

이건 장애 대응에서 분명한 강점이다.

특히:
- `CheckLoginUser-new1.mhi`
- `RetrieveToday.mhi`
- `RetrieveDbmsConnectionString.mhi`
- `HP_BAT01206B`
- `HP_BAT01206B01`

같은 식별자는 추적 실마리로 충분히 유용하다.

### 4.2 배치 로그는 생각보다 좋다
`batchLog`, `devon-batch-syslog`, 그룹/잡 로그는 아래를 보여준다.

- 그룹 실행
- 동기/비동기 여부
- override 예정
- 상태 전이
- 예외 계층

즉 배치 쪽은 현대적이지는 않지만, 운영 흔적은 비교적 성실하다.

이건 "망가진 시스템"의 로그가 아니라 "운영 지식이 필요하지만 흔적은 남기는 시스템"의 로그다.

### 4.3 공통 캐시와 초기화 수치가 남는 건 장점이다
초기화 시점의:

- 공통코드 수량
- 사용자 수량
- 부서 수량

같은 건 시스템 상태를 대략 읽게 해준다.

예를 들어:
- 캐시 수량이 갑자기 비정상적으로 줄거나
- 특정 초기화가 안 찍히면
- 운영 이상을 빨리 감지할 수 있다.

## 5. 장애 대응 관점에서 본 약점

### 5.1 로그가 기술자 내부어 중심이다
현재 로그는 기술 계층은 잘 보이지만, 업무 의미가 약하다.

예:
- `RetrieveToday.mhi`
- `UpdateCntInit1.mhi`
- `RetrieveDbmsConnectionString.mhi`

이 이름만으로는 신규 유지보수자가 바로 이해하기 어렵다.

부족한 정보:
- 어떤 화면에서 발생했는지
- 어떤 메뉴에서 발생했는지
- 사용자가 무엇을 하던 중인지
- 이 호출이 성공/실패에 어떤 의미인지

즉 "호출은 보이는데 업무 의미가 약한" 로그다.

### 5.2 요청 단위 상관관계가 약하다
현재 로그를 읽으면 아래는 각각 보인다.

- 화면 호출
- DB SQL
- 배치 실행
- 외부 연계 초기화

하지만 이를 하나의 요청 단위로 강하게 묶는 공통 식별자는 약하다.

필요한 것:
- request id
- session id
- user id
- screen id
- menu id
- mhi
- command
- elapsed

이게 없으면 장애 상황에서 "하나의 사용자 행동"을 끝까지 따라가기 어렵다.

### 5.3 SQL 로그는 강하지만 너무 원시적이다
`dbwrap.log`는 분석용으로는 좋다.
하지만 운영용으로는 너무 낮은 레벨이다.

현재는:
- 실제 SQL
- 바인딩 흔적
- 테이블

은 잘 보인다.

반면 운영자가 더 보고 싶은 건 아래다.

- 어느 화면인가
- 어느 query path 인가
- 어느 xmlquery statement 인가
- 몇 건을 읽고 썼는가
- 얼마나 오래 걸렸는가

즉 SQL 자체보다 `업무-쿼리 매핑`이 더 필요하다.

### 5.4 외부 연계 로그는 "붙었다"는 건 보이는데 "얼마나 건강한지"는 약하다
`LST`, `POLNET`, `HttpClientUtil Loaded Cert` 같은 로그는 존재한다.

하지만 이걸로는 아래가 충분히 안 보인다.

- 외부 호출 성공률
- 최근 실패 횟수
- 평균 응답시간
- 타임아웃 비율
- 재시도 여부

즉 연계가 있다는 건 보이지만, 연계 품질은 잘 안 보인다.

## 6. 로그를 통해 추정되는 운영 스타일

### 6.1 SI형 엔터프라이즈 운영 스타일
로그와 구조를 같이 보면, 운영 스타일은 전형적인 SI형 엔터프라이즈에 가깝다.

특징:
- 공통화가 강함
- 관리 기능이 시스템 내부에 들어있음
- 설정과 운영 지식 의존도가 큼
- 외부 연계가 많음
- 배치와 온라인이 혼재

즉 "작고 단순한 웹 서비스" 감각으로 보면 안 맞는다.

### 6.2 숙련자에게는 강하지만 신규자에게는 비싸다
이 시스템은 숙련 운영자/유지보수자에게는 쓸 만하다.
왜냐하면 로그가 충분히 내부 정보를 많이 주기 때문이다.

반대로 신규자에게는 비싸다.
왜냐하면:

- 용어가 내부어 중심이고
- 계층이 많고
- 로그의 의미를 해석하려면 시스템 맥락을 알아야 하기 때문이다.

즉 이 시스템은 `정보 부족형`이라기보다 `정보 해석 비용형`이다.

## 7. 로그 개선 포인트

### 7.1 요청 단위 공통 키를 넣는 것이 최우선이다
최소한 아래는 공통 로그 포맷으로 묶는 게 좋다.

| 항목 | 이유 |
|---|---|
| requestId | 사용자 행동 단위 추적 |
| userId | 운영 책임 추적 |
| screenId | 화면 기준 분석 |
| menuId | 메뉴 기준 분석 |
| mhi | 서버 진입점 분석 |
| command | 프레임워크 내부 추적 |
| elapsedMs | 병목 판단 |
| resultCode | 실패 구분 |

### 7.2 `.mhi -> command -> xmlquery`를 로그에서 바로 읽게 해야 한다
현재는 이 체인을 문서와 소스에서 풀어야 한다.

개선 방향:
- `.mhi`
- `command`
- `query path`
- `xmlquery statement`
- `rows`
- `elapsed`
를 한 요청 묶음으로 남기기

이렇게 되면 운영자와 유지보수자가 같은 로그를 보고도 훨씬 빨리 합의할 수 있다.

### 7.3 배치는 `JOB_ID -> Java class`를 로그에 같이 찍어야 한다
현재 가장 아쉬운 점이다.

지금은:
- `HP_BAT01206B`
- `HP_BAT01206B01`
은 보이지만,
- 실제 Java Job 클래스는 바로 안 보인다.

배치 쪽 개선 1순위는:
- `GROUP_ID`
- `JOB_ID`
- `target class`
- `reader`
- `writer`
- `override`
를 같은 실행 레코드로 남기는 것이다.

### 7.4 외부 연계는 품질 지표 로그가 필요하다
적어도 아래는 있어야 한다.

- endpoint
- elapsed
- success/fail
- timeout
- retry count
- last error type

지금은 "연계 초기화됨"은 보이는데 "연계 품질"은 잘 안 보인다.

## 8. 로그 읽는 법

### 8.1 일반 사용자 문제
다음 순서로 읽는 게 가장 빠르다.

1. `debug.log`에서 `.mhi`
2. `dbwrap.log`에서 SQL
3. 필요 시 `031.front-channel` 문서로 화면 역추적
4. 필요 시 `032.framework-core` 문서로 command/PC/EC 역추적

### 8.2 배치 문제
다음 순서로 읽는 게 가장 빠르다.

1. `batch-file-log/.../GROUP_ID-EXEC_COUNT.log`
2. `GROUP_ID-EXEC_COUNT-JOB_ID.log`
3. `devon-batch-syslog`
4. `batchMgr UI / DB metadata`
5. `0323.batch-rule` 문서

### 8.3 외부 연계 문제
다음 순서로 읽는 게 가장 빠르다.

1. `info.log` 초기화 흔적
2. `debug.log` 또는 개별 기능 로그
3. `0332.integration` 문서
4. 인증서/설정 파일

## 9. 지금 로그만으로 운영 대시보드에 올릴 만한 항목

### 9.1 온라인
- 로그인 성공/실패 수
- `.mhi` 호출 상위 N개
- 평균 응답시간 상위 N개
- DB 쿼리 시간 상위 N개
- 외부 연계 호출 성공/실패율

### 9.2 배치
- JobGroup 실행 수
- Job 실패 수
- override 실행 횟수
- 최근 실패 Job ID
- 평균 실행시간 상위 JobGroup

### 9.3 초기화/운영
- 캐시 초기화 성공 여부
- 외부 핸들러 등록 성공 여부
- 인증서 로드 성공 여부
- datasource update 흔적

## 10. 현재 시점의 Minority Report 결론

1. 이 시스템은 로그를 아예 안 남기는 시스템이 아니다.
2. 오히려 운영 흔적은 꽤 성실히 남긴다.
3. 문제는 로그량 부족이 아니라 `해석 비용`이 높다는 점이다.
4. 즉 개선 포인트는 기능보다 `관측성`, `상관관계`, `운영자 가독성` 쪽이다.
5. 이 시스템은 망가진 시스템이라기보다, 운영 지식이 있어야 잘 다룰 수 있는 시스템으로 보인다.

## 11. 같이 볼 문서
- 배치 운영 구조는 [D.현행-스케줄-운영방식.md](/home/forsylph/ollama/NPH/03.analysis_results/032.framework-core/0323.batch-rule/D.%ED%98%84%ED%96%89-%EC%8A%A4%EC%BC%80%EC%A4%84-%EC%9A%B4%EC%98%81%EB%B0%A9%EC%8B%9D.md)
- 버전관리·배포관리 초안은 [Draft.버전관리-배포관리-드래프트.md](/home/forsylph/ollama/NPH/03.analysis_results/032.framework-core/0325.Version%20Control%20and%20Deployment/Draft.%EB%B2%84%EC%A0%84%EA%B4%80%EB%A6%AC-%EB%B0%B0%ED%8F%AC%EA%B4%80%EB%A6%AC-%EB%93%9C%EB%9E%98%ED%94%84%ED%8A%B8.md)
- SVN 로그 조회 사례는 [F.AZ_SYS03100M-SVN로그조회-실행체인.md](/home/forsylph/ollama/NPH/03.analysis_results/037.runtime-trace/F.AZ_SYS03100M-SVN%EB%A1%9C%EA%B7%B8%EC%A1%B0%ED%9A%8C-%EC%8B%A4%ED%96%89%EC%B2%B4%EC%9D%B8.md)
