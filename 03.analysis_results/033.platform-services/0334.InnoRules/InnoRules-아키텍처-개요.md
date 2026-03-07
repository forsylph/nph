# NPH InnoRules Rule Engine 아키텍처 분석

> **분석 대상**: `N:\99.SourceCode Backup\NPH\AADEV_NPH\workspace\NPH_HIS\devonhome\conf\ruleEngine\`
>
> **제품 정보**: InnoRules (innoexpert Co., Ltd.)
>
> **작성일**: 2026-03-07
>
> **검증 상태**: ⚠️ 설정 파일 분석 기반 (실제 Rule/코드 확인 필요)
>
> **주의**: 본 문서는 설정 파일 분석만을 기반으로 작성되었으며, 실제 Rule 정의 및 Java 코드 사용 패턴 확인을 통해 지속 업데이트될 예정입니다.

> **리뷰 메모**
> - 이 문서는 `devonhome/conf/ruleEngine` 및 관련 설정을 기준으로 InnoRules 솔루션 자체를 설명한다.
> - DevOn Batch 컨테이너 자체 설명은 `../../032.framework-core/0323.batch-rule/DevOn-Batch-컨테이너-개요.md`로 분리했다.
> - 본문 중 사용 목적 추정은 그대로 남겨두되, 현재 코드에서 직접 확인된 InnoRules 사용은 청구/심사, 임상 Rule, 인사 Rule UC 쪽이 중심이다.

---

## 1. 개요

### 1.0 제품 관점과 NPH 실사용 관점

공개 자료 기준으로 InnoRules는 RuleBuilder, DDM, 시뮬레이션/테스트, Rule 저작/관리 기능을 함께 제공하는 BRMS 제품군으로 보인다. 이 관점은 제품 철학과 기능 범위를 이해하는 데 도움을 준다.

하지만 NPH 실코드 기준으로 직접 확인되는 운영 표면은 그와 다소 다르다.

- `제품 관점`: RuleBuilder, DDM, Rule 저작/관리, 시뮬레이션, 저장소 관리
- `NPH 실사용 관점`: `RulesInterface`, `RuleReq`, `ruleclient2.ServletInitializer`, `innorules.xml` 초기화 모듈, `rulesclient.innorulesj.InterfaceFactory`
- `현재 미확인`: `rule_content` 실제 원문, IRL 실문법, `IRClient`/`IRSession`의 업무 코드 직접 사용 비중 및 class 실체

따라서 이 문서는 공개 제품 자료를 보조 프레임으로 참고하되, NPH 분석의 주 근거는 코드/설정/JAR에서 직접 확인한 사실에 둔다.

### 1.1 InnoRules란?

InnoRules는 **innoexpert(이노엑스퍼트)**사에서 개발한 **BRMS (Business Rule Management System)**이다. 공개 자료 기준으로는 Drools, Red Hat Decision Manager, FICO Blaze Advisor와 유사한 엔터프라이즈 규칙 엔진 제품군으로 읽힌다.

| 속성 | 내용 |
|------|------|
| **제조사** | innoexpert Co., Ltd. (한국) |
| **유형** | BRMS (Business Rule Management System) |
| **실행 방식** | Java-based Rule Engine |
| **Rule 언어** | IRL (InnoRule Language) |
| **DB 저장소** | Oracle/Tibero |

### 1.3 현재 확인된 운영 표면 요약

현재 워크스페이스 전체 JAR/소스/설정 스캔 기준으로 보면 InnoRules의 운영 표면은 다음처럼 정리된다.

| 구분 | 현재 확인 결과 | 해석 |
|------|----------------|------|
| `RulesInterface`, `RuleReq` | `innorules-api.jar`에 class 실체 존재, 업무 소스 10개 파일에서 직접 사용 확인 | NPH 운영 코드의 주 실행 표면 |
| `IRClient`, `IRSession` | 설정/문서/예시 흔적은 있으나 현재 JAR 스캔과 업무 소스에서 class 실체 및 직접 사용 미확인 | 엔진/초기화/예시 계층 후보, 운영 주표면으로 보기 어려움 |
| `ruleclient2.ServletInitializer` | `web.xml`에서 확인 | 웹 초기화 계층 |
| `innorulesj.batch.initializer.*` | `innorules.xml`에서 확인 | 모듈 초기화 계층 |
이 표 때문에 이후 본문에서 IRClient, IRSession이 등장하더라도, 현재 NPH 운영 코드의 직접 표면을 대표하는 것으로 읽으면 안 된다.
### 1.2 NPH에서의 사용 목적 (추정 + 코드 근거 보강)

현재까지 확인된 코드와 설정을 기준으로 보면 InnoRules 사용은 청구/심사, DMS, 응급 SEPSIS, 인사성 Rule 쪽에서 직접 확인됩니다. 다음 표는 확인 강도에 따라 다시 정리한 것입니다:

| 사용 영역 | 근거 | 신뢰도 |
|-----------|-----------|--------|
| **청구 생성/청구 검증 Rule** | `ClamCrtnUC`, `ClamChkRuleUC`, `ClamCrtnRuleUC`에서 `#S00000327`, `#S00000317`, `#S00000634`, `#S00000667` 확인 | ✅ 높음 |
| **DMS/사전심사 Rule** | `DmsChkRuleUC`에서 `DMS00001` 확인 | ✅ 높음 |
| **응급/처방 SEPSIS Rule** | `DmsPreRuleUC`, `PrscSEPSISRuleUC`에서 `SEPSIS` 확인 | ✅ 높음 |
| **인사/연말정산 Rule** | `PayUC`에서 `HR0001R` 확인 | ✅ 높음 |
| **업무 유효성 검증/보조 함수 사용** | `UserDefinedFunctionLoader`, `user-defined-function.xml` 존재 | 🟡 중간 |
| **인증/권한 규칙** | 현재 코드/설정 기준 직접 근거 부족 | 🔴 약함 |

---

## 2. 시스템 아키텍처

### 2.1 전체 구조

```
┌─────────────────────────────────────────────────────────────┐
│                    NPH Application                          │
│                  (DevOn Framework)                          │
├─────────────────────────────────────────────────────────────┤
│  PC (Process Component)                                     │
│  EC (Entity Component)                                      │
│  CMD (Command)                                              │
└──────────────────────────┬──────────────────────────────────┘
                           │ 호출
                           ▼
┌─────────────────────────────────────────────────────────────┐
│     Client / API Layer                                      │
│  web: ruleclient2 / app: rulesclient / init: innorulesj     │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                InnoRules Engine                             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │  Rule Cache     │    │  Rule Executor  │                │
│  │  (Memory)       │◄──►│                 │                │
│  └────────┬────────┘    └────────┬────────┘                │
│           │                      │                          │
│           └──────────────────────┘                          │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Rule Repository                        │   │
│  │              (RULESDB_DEV)                          │   │
│  └────────────────────────┬────────────────────────────┘   │
└───────────────────────────┼─────────────────────────────────┘
                            │ JDBC
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Database                                 │
│  ┌─────────────────┐    ┌─────────────────┐              │
│  │     Oracle      │    │     Tibero      │              │
│  │   (NPHHISDEV)   │    │   (NPHHISDEV)   │              │
│  └─────────────────┘    └─────────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 초기화 흐름

```
NPH Application 시작
        │
        ▼
┌─────────────────────────────────────┐
│  Priority 10: LoggerConfigurator    │
│  - 로깅 설정 초기화                 │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  Priority 20: JNDIInitializer       │
│  - JNDI 환경 설정                   │
│  - provider-urls: default           │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  Priority 30: UserDefinedFunctionLoader │
│  - 사용자 정의 함수 로드            │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  Priority 40: RuleInitializer       │
│  - Rule Repository 초기화           │
│  - repositories: rule               │
│  - DB 연결 (RULESDB_DEV)            │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  Priority 60: RuleClientInitializer │
│  - Rule Client 초기화               │
│  - Rule client 초기화 완료          │
└─────────────────────────────────────┘
```

---

## 3. 설정 파일 상세 분석

### 3.1 innorules.xml - 모듈 초기화 설정

**파일 위치**: `devonhome/conf/ruleEngine/innorules.xml`

```xml
<innorules-modules>
    <module>
        <module-class>com.innoexpert.innorulesj.batch.initializer.LoggerConfigurator</module-class>
        <init-priority>10</init-priority>
    </module>
    <module>
        <module-class>com.innoexpert.innorulesj.batch.initializer.JNDIInitializer</module-class>
        <init-priority>20</init-priority>
        <init-param>
            <param-name>provider-urls</param-name>
            <param-value>default</param-value>
        </init-param>
    </module>
    <module>
        <module-class>com.innoexpert.innorulesj.batch.initializer.UserDefinedFunctionLoader</module-class>
        <init-priority>30</init-priority>
    </module>
    <module>
        <module-class>com.innoexpert.innorulesj.batch.initializer.RuleInitializer</module-class>
        <init-priority>40</init-priority>
        <init-param>
            <param-name>repositories</param-name>
            <param-value>rule</param-value>
        </init-param>
    </module>
    <module>
        <module-class>com.innoexpert.innorulesj.batch.initializer.RuleClientInitializer</module-class>
        <init-priority>60</init-priority>
    </module>
</innorules-modules>
```

#### 분석 포인트

| 항목 | 내용 | 분석 |
|------|------|------|
| `batch.initializer` 패키지 | InnoRules 초기화 모듈 | ✅ 설정 파일에서 직접 확인, 다만 업무 코드의 주표면 API는 아님 |
| `provider-urls: default` | JNDI Provider | 🟡 JNDI 연동 설정으로 보이나 실제 컨테이너 구현 상세는 미확인 |
| `repositories: rule` | 저장소 이름 | ✅ 단일 저장소 사용 |

### 3.2 rule.conf - 저장소 설정

**파일 위치**: `devonhome/conf/ruleEngine/rule.conf`

```properties
# JNDI 설정
REPOSITORY.PROPS.JNDI.FACTORYCLASS=com.innoexpert.utility.jndi.IrInitialContextFactory
REPOSITORY.PROPS.JNDI.PROVIDERURL=default

# 데이터소스 설정
REPOSITORY.DATASOURCE.NAME=RULESDB_DEV

# 저장소 설정
REPOSITORY.NAME=rule
REPOSITORY.PROPS.LOAD.VERSION=NONE
REPOSITORY.PROPS.LOAD.DEFERRED=NO

# 성능/로깅 설정
REPOSITORY.LOADERPROPS.TRACE.DISABLED=true
REPOSITORY.LOADERPROPS.OPTIMIZE.MEMORY.LEVEL=20
```

#### 설정 상세

| 설정 키 | 값 | 설명 |
|--------|-----|------|
| `REPOSITORY.NAME` | `rule` | 저장소 이름 |
| `LOAD.VERSION` | `NONE` | ⚠️ 버전 관리 미사용 |
| `LOAD.DEFERRED` | `NO` | ⚠️ 지연 로딩 미사용 (시작 시 전체 로드) |
| `OPTIMIZE.MEMORY.LEVEL` | `20` | 메모리 최적화 레벨 (1~100) |
| `TRACE.DISABLED` | `true` | 추적 로깅 비활성화 |

### 3.3 jndi.xml - 데이터베이스 연결

**파일 위치**: `devonhome/conf/ruleEngine/jndi.xml`

```xml
<Resources>
    <!-- Oracle 설정 (활성화) -->
    <Resource
        jndi-type="com.innoexpert.utility.jdbc.VOCDataSource"
        jndi-name="RULESDB_DEV"
        defaultAutoCommit="false"
        driverClassName="oracle.jdbc.driver.OracleDriver"
        url="jdbc:oracle:thin:@(description=(address=(protocol=tcp)(host=10.60.210.26)(port=1521))(connect_data=(service_name=NPHHISDEV)))"
        username="nph_inr"
        password="inrdev"
        initCnt="5"
        maxActive="10"
        maxIdle="5"
        maxWait="5000"
        preparedStmtCacheCnt="10"/>
</Resources>
```

#### DB 연결 설정

| 속성 | 값 | 설명 |
|------|-----|------|
| `jndi-name` | `RULESDB_DEV` | JNDI 이름 |
| `jndi-type` | `VOCDataSource` | InnoRules 전용 DataSource |
| `defaultAutoCommit` | `false` | ⚠️ 자동 커밋 비활성화 (TX 수동 관리) |
| `maxActive` | `10` | 최대 연결 수 |
| `maxIdle` | `5` | 유휴 연결 수 |
| `maxWait` | `5000` | 연결 대기 시간 (ms) |

---

## 4. Rule 저장소 구조

### 4.1 저장소 접근 방식

```
┌─────────────────────────────────────────────┐
│          InnoRules Repository               │
├─────────────────────────────────────────────┤
│  Table: RULE_DEFINITION                     │
│  ├── RULE_ID (PK)                           │
│  ├── RULE_NAME                              │
│  ├── RULE_CONTENT (IRL)                     │
│  ├── VERSION                                │
│  ├── STATUS (ACTIVE/INACTIVE)               │
│  └── LAST_MODIFIED                          │
├─────────────────────────────────────────────┤
│  Table: RULESET                             │
│  ├── RULESET_ID                             │
│  ├── RULESET_NAME                           │
│  └── RULE_IDS (FK)                          │
└─────────────────────────────────────────────┘
```

> ⚠️ **추정**: 실제 테이블 구조는 DB 스키마 확인 필요

### 4.2 Rule 캐싱 전략

| 설정 | 값 | 의미 |
|------|-----|------|
| `OPTIMIZE.MEMORY.LEVEL=20` | 낮음 | 메모리 사용 최소화, 디스크 접근 증가 가능 |
| `LOAD.DEFERRED=NO` | 즉시 로드 | 애플리케이션 시작 시 모든 Rule 메모리 적재 |

---

## 5. NPH와의 연동 방식

### 5.1 예상 연동 아키텍처

```
┌─────────────────────────────────────────────┐
│              NPH Application                │
│           (DevOn Framework)                 │
├─────────────────────────────────────────────┤
│  PC (Process Component)                     │
│  EC (Entity Component)                      │
│  CMD (Command)                              │
└──────────────┬──────────────────────────────┘
               │ 호출
               ▼
┌─────────────────────────────────────────────┐
│              IRClient API                   │
│      com.innoexpert.innorulesj.*            │
└──────────────┬──────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────┐
│            InnoRules Engine                 │
│         (Rule Execution)                    │
└──────────────┬──────────────────────────────┘
               │ 규칙 로드
               ▼
┌─────────────────────────────────────────────┐
│              RULESDB_DEV                    │
│         (Oracle/Tibero)                     │
└─────────────────────────────────────────────┘
```

### 5.2 예상 Java 호출 패턴

```java
// [⚠️ 추정] 실제 확인 필요
import com.innoexpert.innorulesj.client.IRClient;
import com.innoexpert.innorulesj.session.IRSession;

public class SomePC {
    public LData calculateFee(LData input) {
        // Rule Engine 세션 획득
        IRSession session = IRClient.getSession("rule");

        // Rule 실행
        session.setInputData(input);
        session.execute("DRG_CALCULATION_RULE");

        // 결과 반환
        return session.getOutputData();
    }
}
```

> ⚠️ **미확인**: 실제 API는 위와 다를 수 있습니다. `com.innoexpert.innorulesj` 패키지의 실제 사용 코드 확인 필요.

---

## 6. 향후 검토 필요사항 (Future Review Items)

### 6.1 DB 테이블 스키마 확인 필요 ⭐

> **⚠️ 중요**: 아래 항목들은 DB 접속 및 테이블 조회가 필요합니다. 사용자가 추후 덤프를 제공할 예정입니다.

| # | 항목 | 확인 방법 | 상태 |
|---|------|-----------|:----:|
| 1 | **Rule 테이블 스키마** | `RULESDB_DEV` (nph_inr 계정) 접속 | ⏳ 대기중 |
| 2 | **Rule 정의 내용 조회** | Rule 코드별 실제 IRL/Rule 내용 확인 | ⏳ 대기중 |
| 3 | **Rule 버전 관리 테이블** | 버전 이력 관리 여부 확인 | ⏳ 대기중 |
| 4 | **Rule 실행 로그 테이블** | Rule 실행 이력/성능 데이터 확인 | ⏳ 대기중 |

**필요한 정보**:
```sql
-- 확인 필요 SQL
-- 1. Rule 테이블 목록
SELECT table_name FROM user_tables WHERE table_name LIKE '%RULE%';

-- 2. Rule 정의 조회 (예시)
SELECT * FROM rule_definition WHERE rule_code IN ('#S00000327', 'DMS00001', 'SEPSIS');

-- 3. Rule 코드별 내용
SELECT rule_code, rule_content, version, status FROM rule_master;
```

### 6.2 소스 코드 분석 완료 항목 ✅

| # | 항목 | 확인 방법 | 상태 |
|---|------|-----------|:----:|
| 1 | Java 코드에서의 사용 패턴 | `com.innoexpert` 패키지 import 검색 | ✅ 완료 |
| 2 | 배치 모듈과의 연동 | `OtptClamCrtnJob.java` 분석 | ✅ 완료 |
| 3 | Rule 사용 UC 클래스 | 6개 클래스 확인 | ✅ 완료 |
| 4 | Rule 코드 목록 | 9개 Rule 코드 확인 | ✅ 완료 |

### 6.3 추가 확인 권장사항

| # | 항목 | 확인 방법 | 우선순위 |
|---|------|-----------|:--------:|
| 1 | Rule 수정/배포 프로세스 | 운영 담당자 인터뷰 | 🟡 중간 |
| 2 | Rule 성능 모니터링 | 실행 시간/에러 로그 확인 | 🟡 중간 |
| 3 | UDF 클래스 상세 구현 | `user-defined-function.xml` 파싱 | 🟡 중간 |

### 6.4 확인되지 않은 가설

| 가설 | 근거 | 검증 필요 |
|------|------|:---------:|
| Rule은 DB에만 저장 | `LOAD.DEFERRED=NO` | ⏳ DB 확인 후 검증 |
| 배치 초기화 패키지 존재 | `batch.initializer` 패키지 | 🟡 설정 기준 강함, 그러나 업무 코드의 직접 사용은 UC 쪽이 더 넓음 |
| 수가/DRG 계산에 사용 | 병원 시스템 특성 | ⏳ Rule 내용 확인 필요 |

---

## 7. 마이그레이션 시 고려사항

### 7.1 현재 구조의 마이그레이션 복잡도

| 구성요소 | 복잡도 | 설명 |
|----------|:------:|------|
| Rule 저장소 (DB) | 중간 | 테이블 구조 마이그레이션 필요 |
| Rule 언어 (IRL) | 높음 | Drools/DMN 등으로 변환 필요 |
| Java API 연동 | 낮음 | 대체 API로 교체 |
| UDF (사용자 함수) | 높음 | Java 코드로 재구현 필요 |

### 7.2 대안 Rule Engine 비교

| 제품 | 마이그레이션 난이도 | 특징 |
|------|:-------------------:|------|
| **Drools** | 중간 | 오픈소스, 활발한 커뮤니티 |
| **Red Hat Decision Manager** | 중간 | Drools 기업판, 지원 가능 |
| **Camunda DMN** | 낮음 | BPMN/DMN 표준, NPH에 적합할 수 있음 |
| **직접 구현** | 높음 | 유지보수 비용 발생 |

---

## 8. 참고 자료

### 8.1 설정 파일 경로

| 파일 | 경로 | 용도 |
|------|------|------|
| `innorules.xml` | `devonhome/conf/ruleEngine/innorules.xml` | 모듈 초기화 |
| `rule.conf` | `devonhome/conf/ruleEngine/rule.conf` | 저장소 설정 |
| `jndi.xml` | `devonhome/conf/ruleEngine/jndi.xml` | DB 연결 |
| `user-defined-function.xml` | `devonhome/conf/ruleEngine/user-defined-function.xml` | UDF 설정 |

### 8.2 외부 참고

- InnoRules 공식 문서 (제조사 문의 필요)
- BRMS 일반: https://www.omg.org/spec/BMM/
- DMN 표준: https://www.omg.org/spec/DMN/

---

## 문서 이력

| 일자 | 버전 | 변경 내용 | 작성자 |
|------|------|-----------|--------|
| 2026-03-07 | 0.1 | 초안 작성 | Claude |

---

*본 문서는 설정 파일 분석만을 기반으로 작성되었습니다.*






