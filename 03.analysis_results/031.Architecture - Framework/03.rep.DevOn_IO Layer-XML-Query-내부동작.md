# DevOn XML Query 내부 동작 분석

> iBatis/MyBatis 없이 DevOn이 직접 구현한 XML → JDBC 직접 실행 구조

---

## 핵심 결론

**DevOn = XML Query Parser + JDBC 직접 실행**

- ❌ iBatis 사용 안 함
- ❌ MyBatis 사용 안 함
- ✅ **DevOn 자체 구현** (LQueryService)

```
XML 파일 → LQueryService 파싱 → JDBC PreparedStatement → ResultSet → LMultiData
```

---

## 1. iBatis/MyBatis와의 차이

### iBatis/MyBatis 구조
```
SQL Mapper XML
    ↓
SqlSessionFactory
    ↓
Mapper Interface / SqlSession
    ↓
JDBC
```

**특징:**
- `#{param}` → PreparedStatement `?` 변환
- ResultMap을 통한 객체 매핑
- 캐싱, 동적 SQL 등 고급 기능

### DevOn 구조
```
XML Query File
    ↓
LQueryService.executeQuery()
    ↓
XML 파싱 (DOM/SAX)
    ↓
${param} → 값 치환 (문자열 치환)
    ↓
JDBC Statement.executeQuery()
    ↓
ResultSet → LMultiData
```

**특징:**
- `${param}` → **문자열 치환** (PreparedStatement 아님)
- 별도의 Mapper 인터페이스 없음
- **단순 문자열 조립** 후 직접 실행

---

## 2. XML Query 파일 구조

### 실제 XML 형태
```xml
<!-- devonhome/xmlquery/app/emp/login.xml -->
<?xml version="1.0" encoding="EUC-KR"?>
<queries>
    <query name="retrieveUserInfo">
        <statement>
            SELECT USER_ID, USER_NAME, DEPT_CODE
            FROM USER_MASTER
            WHERE USER_ID = '${usid}'          <!-- 주의: 문자열 치환 -->
              AND PASSWORD = '${passwd}'
              AND USE_YN = 'Y'
        </statement>
    </query>

    <query name="updateLoginTime">
        <statement>
            UPDATE USER_MASTER
            SET LAST_LOGIN_TIME = SYSDATE
            WHERE USER_ID = '${usid}'
        </statement>
    </query>

    <query name="retrieveUserList">
        <statement>
            SELECT * FROM USER_MASTER
            WHERE DEPT_CODE = '${deptCode}'
            <if test="status != null">
                AND STATUS = '${status}'
            </if>
            ORDER BY USER_NAME
        </statement>
    </query>
</queries>
```

**주의사항:**
- `${param}` 문법 사용 (iBatis의 `#{param}`과 다름)
- **문자열 직접 치환** (PreparedStatement의 `?` 바인딩 아님)
- SQL 인젝션 주의 필요

---

## 3. LQueryService 내부 동작

### 3.1 전체 실행 흐름

```java
// PC에서 호출
public class LoginPC {
    public LMultiData validateUser(LData input) {
        // 1. XML 파일 경로와 쿼리 이름 지정
        LMultiData result = LQueryService.executeQuery(
            "app/emp/login",      // XML 파일 경로 (확장자 없음)
            "retrieveUserInfo",   // 쿼리 이름
            input                 // 파라미터 (LData)
        );
        return result;
    }
}
```

### 3.2 LQueryService 구현 (가상 코드)

```java
public class LQueryService {

    private static Map<String, Document> xmlCache = new HashMap<>();

    public static LMultiData executeQuery(String xmlPath, String queryName, LData param) {
        // 1. XML 파일 로드 (캐싱)
        Document doc = loadXmlDocument(xmlPath + ".xml");

        // 2. 쿼리 노드 찾기
        Element queryElement = findQueryElement(doc, queryName);
        String sql = queryElement.getChildText("statement");

        // 3. ${param} 문자열 치환
        String executedSql = replaceParameters(sql, param);
        // 예: SELECT * FROM USER WHERE ID = '${usid}'
        //   → SELECT * FROM USER WHERE ID = 'admin'

        // 4. JDBC 실행
        Connection conn = DataSourceManager.getConnection();
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery(executedSql);

        // 5. 결과 변환
        LMultiData result = convertResultSetToLMultiData(rs);

        // 6. 리소스 정리
        close(rs, stmt, conn);

        return result;
    }

    // ${param} 문자열 치환
    private static String replaceParameters(String sql, LData param) {
        String result = sql;
        for (String key : param.getKeys()) {
            String value = param.getString(key);
            // SQL Injection 방지를 위한 기본적인 escape
            value = escapeSql(value);
            result = result.replace("${" + key + "}", value);
        }
        return result;
    }
}
```

### 3.3 ResultSet → LMultiData 변환

```java
private static LMultiData convertResultSetToLMultiData(ResultSet rs)
    throws SQLException {

    LMultiData multiData = new LMultiData("queryResult");
    ResultSetMetaData metaData = rs.getMetaData();
    int columnCount = metaData.getColumnCount();

    // 컬럼 메타데이터 설정
    LMetaData lMetaData = new LMetaData();
    for (int i = 1; i <= columnCount; i++) {
        String columnName = metaData.getColumnName(i);
        String columnType = metaData.getColumnTypeName(i);
        lMetaData.addColumn(columnName, convertSqlType(columnType));
    }
    multiData.setMetaData(lMetaData);

    // 데이터 행 변환
    while (rs.next()) {
        LData rowData = new LData();
        for (int i = 1; i <= columnCount; i++) {
            String columnName = metaData.getColumnName(i);
            Object value = rs.getObject(i);
            rowData.setObject(columnName, value);
        }
        multiData.add(rowData);
    }

    return multiData;
}
```

---

## 4. DevOn vs iBatis/MyBatis 비교

### 4.1 파라미터 바인딩

| 구분 | iBatis/MyBatis | DevOn LQueryService |
|------|----------------|---------------------|
| **문법** | `#{param}` | `${param}` |
| **방식** | PreparedStatement `?` | **문자열 치환** |
| **SQL Injection** | 자동 방지 | **수동 escape 필요** |
| **타입 처리** | 자동 | 수동 |

```java
// iBatis - PreparedStatement 사용
SELECT * FROM USER WHERE ID = #{userId}
// → SELECT * FROM USER WHERE ID = ?
// → stmt.setString(1, userId)

// DevOn - 문자열 치환
SELECT * FROM USER WHERE ID = '${usid}'
// → "SELECT * FROM USER WHERE ID = '" + userId + "'"
// → Statement.executeQuery()  (PreparedStatement 아님)
```

### 4.2 결과 매핑

| 구분 | iBatis/MyBatis | DevOn LQueryService |
|------|----------------|---------------------|
| **결과 타입** | DTO/VO 객체 | **LMultiData** (Map-like) |
| **매핑** | ResultMap / 자동 매핑 | ResultSet → LData |
| **타입 변환** | 자동 | 수동 |

```java
// iBatis
User user = sqlSession.selectOne("getUser", id);
// → User 객체로 자동 매핑

// DevOn
LMultiData result = LQueryService.executeQuery("app/emp/login", "getUser", param);
String userName = result.getString(0, "USER_NAME");
// → Map-like 구조로 수동 접근
```

### 4.3 기능 비교

| 기능 | iBatis/MyBatis | DevOn |
|------|----------------|-------|
| **캐싱** | 1차/2차 캐시 지원 | ❌ 없음 |
| **동적 SQL** | `<if>`, `<choose>`, `<foreach>` | ⚠️ 제한적 지원 |
| **연관 매핑** | `<resultMap>` 복잡한 매핑 | ❌ 없음 |
| **지연 로딩** | 지원 | ❌ 없음 |
| **트랜잭션** | 외부 관리 | TxServiceUtil 연동 |

---

## 5. 장단점

### 5.1 장점

1. **단순함**
   - iBatis/MyBatis 의존성 불필요
   - 설정 파일 단순
   - 학습 곡선 낮음

2. **MiPlatform 연동**
   - LMultiData가 MiPlatform Dataset과 자연스럽게 연결
   - 별도의 DTO 변환 불필요

3. **실시간 SQL 수정**
   - XML 파일만 수정하면 재배포 없이 적용 (개발 환경)

### 5.2 단점

1. **SQL Injection 위험**
   ```java
   // 위험: 문자열 직접 치환
   usid = "admin' OR '1'='1";
   // → SELECT * FROM USER WHERE ID = 'admin' OR '1'='1'
   ```

2. **성능**
   - PreparedStatement 미사용으로 DB 쿼리 캐싱 불가
   - 매번 SQL 파싱/컴파일

3. **타입 안정성 없음**
   - 모든 결과가 String/Object로 처리
   - 컴파일 타임 체크 없음

4. **디버깅 어려움**
   - 최종 실행 SQL을 보려면 로그 설정 필요

---

## 6. 실제 사용 예시

### 6.1 간단한 조회

```xml
<!-- xmlquery/az/comn/cmcd.xml -->
<queries>
    <query name="retrievePbhlCdList">
        <statement>
            SELECT PBHL_CD, PBHL_NM, USE_YN
            FROM PBHL_CD_MASTER
            WHERE USE_YN = 'Y'
            ORDER BY PBHL_CD
        </statement>
    </query>
</queries>
```

```java
// PC
public class CmcdPC {
    public LMultiData retrievePbhlCdList(LData param) {
        return LQueryService.executeQuery(
            "az/comn/cmcd",
            "retrievePbhlCdList",
            param
        );
    }
}
```

### 6.2 파라미터 사용

```xml
<!-- xmlquery/md/opn/patient.xml -->
<queries>
    <query name="retrievePatientByDept">
        <statement>
            SELECT PID, NAME, BIRTH_DATE
            FROM PATIENT
            WHERE DEPT_CODE = '${deptCode}'
              AND VISIT_DATE >= '${startDate}'
              AND VISIT_DATE <= '${endDate}'
            ORDER BY NAME
        </statement>
    </query>
</queries>
```

```java
// Command
LData param = new LData();
param.setString("deptCode", "MD");
param.setString("startDate", "20240301");
param.setString("endDate", "20240331");

LMultiData patients = LQueryService.executeQuery(
    "md/opn/patient",
    "retrievePatientByDept",
    param
);
```

### 6.3 INSERT/UPDATE

```xml
<!-- xmlquery/mr/comn/receipt.xml -->
<queries>
    <query name="insertReceipt">
        <statement>
            INSERT INTO RECEIPT (
                RECEIPT_NO, PID, AMOUNT, RECEIPT_DATE, REG_ID
            ) VALUES (
                SEQ_RECEIPT.NEXTVAL,
                '${pid}',
                ${amount},
                SYSDATE,
                '${regId}'
            )
        </statement>
    </query>
</queries>
```

```java
// 트랜잭션 내에서 실행
public class ReceiptPC {
    public void saveReceipt(LData param) {
        // INSERT
        LQueryService.executeQuery(
            "mr/comn/receipt",
            "insertReceipt",
            param
        );

        // UPDATE (같은 트랜잭션)
        LQueryService.executeQuery(
            "mr/comn/receipt",
            "updatePatientStatus",
            param
        );
    }
}
```

---

## 7. XML 쿼리 경로 규칙

### 7.1 디렉토리 구조

```
devonhome/xmlquery/
├── app/                    # 공통
│   ├── emp/
│   │   ├── login.xml
│   │   └── user.xml
├── az/                     # 원무/접수
│   ├── comn/
│   │   └── cmcd.xml
│   └── biz/
├── md/                     # 진료
│   ├── opn/               # 외래
│   └── inp/               # 입원
├── mr/                     # 의무기록
├── sp/                     # 검사
└── er/                     # 응급
```

### 7.2 네이밍 규칙

```java
// 경로 규칙
LQueryService.executeQuery(
    "업무/기능/파일명",  // 확장자 제외
    "쿼리이름",
    param
);

// 예시
"az/comn/cmcd"          → devonhome/xmlquery/az/comn/cmcd.xml
"md/opn/patient"        → devonhome/xmlquery/md/opn/patient.xml
"app/emp/login"         → devonhome/xmlquery/app/emp/login.xml
```

---

## 8. 결론

### DevOn XML Query 정리

```
┌─────────────────────────────────────────────────────────────┐
│                    DevOn XML Query                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  XML 파일 (${param})                                         │
│       ↓                                                     │
│  LQueryService.executeQuery()                                │
│       ↓                                                     │
│  문자열 치환 (replace ${param} → 'value')                   │
│       ↓                                                     │
│  JDBC Statement.executeQuery()                               │
│       ↓                                                     │
│  ResultSet → LMultiData 변환                                │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ 특징:                                                       │
│ • iBatis/MyBatis 미사용 (DevOn 자체 구현)                   │
│ • PreparedStatement 아닌 문자열 치환                         │
│ • 결과는 LMultiData (Map-like)                              │
│ • SQL과 Java 코드 분리                                       │
└─────────────────────────────────────────────────────────────┘
```

### iBatis/MyBatis 대비 DevOn 선택 이유

**당시(2000년대 중반) 상황:**
1. iBatis는 이미 존재했지만, DevOn은 **자체 구현 선택**
2. MiPlatform과의 연동을 위해 **LMultiData 통합** 필요
3. **단순함** 우선 (iBatis의 복잡한 기능 불필요)
4. **LG CNS 내부 표준화** (외부 라이브러리 최소화)

**현재 관점:**
- SQL Injection 위험으로 인해 **PreparedStatement 권장**
- iBatis/MyBatis의 기능이 더 안전하고 강력
- DevOn 방식은 **레거시**로 분류됨

---

*DevOn의 XML Query는 iBatis의 아이디어를 참고했지만, 문자열 치환 방식으로 단순화하여 구현한 것으로 보입니다.*
