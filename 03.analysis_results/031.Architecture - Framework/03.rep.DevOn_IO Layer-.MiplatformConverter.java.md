# MiplatformConverter.java 상세 분석

> MiPlatform Dataset ↔ DevOn LData/LMultiData 변환을 담당하는 핵심 클래스

---

## 개요

### 파일 정보

| 항목 | 내용 |
|------|------|
| **파일명** | `MiplatformConverter.java` |
| **패키지** | `devonx.nph.miplatform` |
| **위치** | `COMMON/src/devonx/nph/miplatform/` |
| **저작권** | LG CNS, Inc. |

### 핵심 역할

**MiplatformConverter는 MiPlatform 클라이언트와 DevOn 서버 사이의 "번역기" 역할을 합니다.**

```
MiPlatform Client (Dataset)
    ↓ (HTTP 전송)
Server → MiplatformRequest.receiveData()
    ↓
MiplatformConverter.convertToLMultiDataWithJobType()
    ↓
DevOn LMultiData (Command/PC 사용)
    ↓
비즈니스 로직 처리
    ↓
MiplatformConverter.convertToDataset()
    ↓
MiPlatform Dataset (Client 응답)
```

---

## 1. 주요 메소드

### 1.1 Dataset → LMultiData 변환 (수신)

```java
/**
 * MiPlatform Dataset을 DevOn LMultiData로 변환
 * CUD 상태 정보를 포함하여 변환
 */
public static LMultiData convertToLMultiDataWithJobType(Dataset ds) {
    LMultiData mData = new LMultiData("convertedMultiData");

    // 1. 일반 행 처리 (INSERT/UPDATE)
    int count = ds.getRowCount();
    for (int rowIdx = 0; rowIdx < count; rowIdx++) {
        // 행 상태 확인 (INSERT, UPDATE, NORMAL)
        String rowType = ds.getRowStatus(rowIdx);
        String jobType = convertToJobType(rowType);

        // _CUD 필드로 작업 유형 표시
        mData.add("_CUD", jobType);  // C=Create, U=Update

        // 행 데이터 변환
        setMultiDataset(ds, rowIdx, mData, isOrder, usid);
    }

    // 2. 삭제된 행 처리
    int delCount = ds.getDeleteRowCount();
    for (int rowIdx = 0; rowIdx < delCount; rowIdx++) {
        mData.add("_CUD", "D");  // D=Delete
        setDeleteMultiDataset(ds, rowIdx, mData);
    }

    return mData;
}

// 행 상태 → JobType 변환
private static String convertToJobType(String rowType) {
    if (rowType.equalsIgnoreCase("INSERT")) {
        return "C";  // Create
    } else if (rowType.equalsIgnoreCase("UPDATE")) {
        return "U";  // Update
    }
    return "";  // Normal (변경 없음)
}
```

**역할:**
- MiPlatform Dataset의 각 행을 LData로 변환
- **행 상태(INSERT/UPDATE/DELETE)**를 `_CUD` 필드로 변환
- 삭제된 행(Delete Row)도 별도로 처리

### 1.2 LMultiData → Dataset 변환 (응답)

```java
/**
 * DevOn LMultiData를 MiPlatform Dataset으로 변환
 */
public static Dataset convertToDataset(LMultiData mData) {
    Dataset ds = new Dataset("Result");

    if (mData != null) {
        // 1. 메타데이터 추출 (컬럼 정보)
        LMetaData metaData = mData.getMetaData();

        // 2. 컬럼 정의 생성
        for (String colName : metaData.getColumnNames()) {
            int colType = metaData.getColumnType(colName);
            String miType = convertToMiPlatformType(colType);
            ds.addColumn(colName, miType);
        }

        // 3. 데이터 행 추가
        int dataCount = mData.getDataCount();
        for (int i = 0; i < dataCount; i++) {
            int row = ds.appendRow();
            LData data = mData.getLData(i);

            // 각 컬럼 데이터 설정
            for (String colId : data.getKeys()) {
                Object value = data.getObject(colId);
                ds.setColumn(row, colId, value);
            }
        }
    }

    return ds;
}
```

**역할:**
- PC에서 반환한 LMultiData를 MiPlatform Dataset으로 변환
- 컬럼 메타데이터를 기반으로 Dataset 구조 생성
- 클라이언트로 전송할 응답 데이터 준비

---

## 2. 데이터 타입 변환

### 2.1 MiPlatform ↔ DevOn 타입 매핑

```java
public class MiplatformConverter {

    // MiPlatform 타입 → DevOn 타입
    public static int convertMiPlatformTypeToColumnType(String miType) {
        switch (miType.toUpperCase()) {
            case "STRING":
                return LMetaData.STRING;
            case "INT":
            case "INTEGER":
                return LMetaData.INT;
            case "DECIMAL":
            case "NUMBER":
                return LMetaData.DECIMAL;
            case "DATE":
            case "DATETIME":
                return LMetaData.DATE;
            case "BLOB":
                return LMetaData.BLOB;
            default:
                return LMetaData.STRING;
        }
    }

    // DevOn 타입 → MiPlatform 타입
    public static String convertToMiPlatformType(int columnType) {
        switch (columnType) {
            case LMetaData.STRING:
                return "STRING";
            case LMetaData.INT:
                return "INT";
            case LMetaData.DECIMAL:
                return "DECIMAL";
            case LMetaData.DATE:
                return "DATE";
            case LMetaData.BLOB:
                return "BLOB";
            default:
                return "STRING";
        }
    }
}
```

| MiPlatform 타입 | DevOn 타입 | SQL 타입 |
|-----------------|------------|----------|
| STRING | String | VARCHAR, CHAR |
| INT | int | INTEGER |
| DECIMAL | BigDecimal | NUMERIC, DECIMAL |
| DATE | Date | DATE, TIMESTAMP |
| BLOB | byte[] | BLOB, CLOB |

---

## 3. 변환 과정 상세

### 3.1 Client → Server (요청)

```java
// MiPlatform Client
function btn_Save_OnClick() {
    // Dataset에 데이터 설정
    ds_UserInfo.setColumn(0, "usid", "admin");
    ds_UserInfo.setColumn(0, "name", "홍길동");
    ds_UserInfo.setRowStatus(0, "UPDATE");  // 수정 상태

    // Transaction 호출
    Transaction("SaveUser", "/az/bizcom/userNavi/SaveUser.mhi",
        "ds_Input=ds_UserInfo:u", "", "", "callback");
}
```

```java
// Server - MiplatformServlet
protected void beforeCatchService() {
    // 1. HTTP 요청 수신
    MiplatformRequest platformReq = new MiplatformRequest(req);
    platformReq.receiveData();  // MiPlatform 프로토콜 파싱

    // 2. DatasetList 추출
    DatasetList dsList = platformReq.getDatasetList();
    Dataset ds_Input = dsList.getDataset("ds_Input");

    // 3. 변환 (MiplatformConverter 사용)
    LMultiData inputData = MiplatformConverter.convertToLMultiDataWithJobType(ds_Input);

    // 결과:
    // inputData.getString(0, "_CUD") = "U" (UPDATE)
    // inputData.getString(0, "usid") = "admin"
    // inputData.getString(0, "name") = "홍길동"
}
```

### 3.2 Server → Client (응답)

```java
// Server - Command
public void execute() {
    // 1. PC 호출
    LoginPC pc = (LoginPC) TxServiceUtil.getNTxService("az.bizcom.LoginPC");
    LMultiData result = pc.retrieveUserList(param);

    // 2. 변환 (MiplatformConverter 사용)
    Dataset ds_Result = MiplatformConverter.convertToDataset(result);

    // 3. 응답에 추가
    platformResponse.addDataset("ds_Result", ds_Result);
}
```

```java
// Client - 콜백 함수
function callback(serviceId, errorCode, errorMsg) {
    if (errorCode == 0) {
        // ds_Result Dataset이 자동으로 채워짐
        var userName = ds_Result.getColumn(0, "name");
        alert("사용자: " + userName);
    }
}
```

---

## 4. 핵심 기능: _CUD 필드

### 4.1 _CUD란?

**_CUD** = **C**reate / **U**pdate / **D**elete

MiPlatform Client에서 데이터 행의 상태를 서버에 전달하기 위한 특수 필드입니다.

```
┌─────────────────────────────────────────────────────┐
│  MiPlatform Dataset Row Status  →  _CUD 필드         │
├─────────────────────────────────────────────────────┤
│  INSERT (신규)                  →  "C" (Create)     │
│  UPDATE (수정)                  →  "U" (Update)     │
│  NORMAL (변경없음)              →  "" (빈값)         │
│  Delete Row (삭제)              →  "D" (Delete)     │
└─────────────────────────────────────────────────────┘
```

### 4.2 _CUD 활용 예시

```java
// PC에서 _CUD 필드를 확인하여 분기 처리
public class UserPC {
    public void saveUsers(LMultiData userList) {
        for (int i = 0; i < userList.getDataCount(); i++) {
            LData user = userList.getLData(i);
            String jobType = user.getString("_CUD");

            switch (jobType) {
                case "C":
                    insertUser(user);  // INSERT
                    break;
                case "U":
                    updateUser(user);  // UPDATE
                    break;
                case "D":
                    deleteUser(user);  // DELETE
                    break;
                default:
                    // 무시 (변경 없음)
                    break;
            }
        }
    }
}
```

---

## 5. 전체 데이터 흐름

### 요청 → 응답 전체 흐름도

```
┌─────────────────────────────────────────────────────────────────┐
│                      Client (MiPlatform)                        │
├─────────────────────────────────────────────────────────────────┤
│  Form (JavaScript)                                              │
│    │                                                           │
│    ▼ cf_Transaction("SaveUser", "/az/.../SaveUser.mhi", ...)   │
│    │                                                           │
│    ▼ Dataset 생성                                               │
│       - ds_Input: [{usid:"admin", name:"홍길동", _CUD:"U"}]    │
│    │                                                           │
│    ▼ HTTP POST (Binary/XML)                                     │
└────┬────────────────────────────────────────────────────────────┘
     │
     ▼ Network
┌────┴────────────────────────────────────────────────────────────┐
│                      Server (DevOn)                             │
├─────────────────────────────────────────────────────────────────┤
│  MiplatformServlet.doPost()                                     │
│    │                                                           │
│    ▼ PlatformRequest.receiveData()                              │
│       - VariableList 파싱                                       │
│       - DatasetList 파싱                                        │
│    │                                                           │
│    ▼ MiplatformConverter.convertToLMultiDataWithJobType()       │
│       - Dataset → LMultiData 변환                              │
│       - _CUD 필드 추가                                          │
│    │                                                           │
│    ▼ Command.execute()                                          │
│       - LMultiData 사용                                          │
│       - PC 호출                                                  │
│    │                                                           │
│    ▼ PC 비즈니스 로직                                            │
│       - LQueryService 실행                                       │
│       - DB 처리                                                  │
│       - LMultiData 반환                                          │
│    │                                                           │
│    ▼ MiplatformConverter.convertToDataset()                     │
│       - LMultiData → Dataset 변환                               │
│    │                                                           │
│    ▼ MiplatformResponse.sendData()                              │
│       - HTTP 응답 생성                                           │
└────┬────────────────────────────────────────────────────────────┘
     │
     ▼ HTTP Response
┌────┴────────────────────────────────────────────────────────────┐
│                      Client (MiPlatform)                        │
├─────────────────────────────────────────────────────────────────┤
│  Dataset 파싱 → ds_Result 채워짐                                │
│    │                                                           │
│    ▼ 콜백 함수 실행                                              │
│       - 화면 갱신                                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. 다른 변환 메소드

### 6.1 VariableList 변환

```java
/**
 * MiPlatform VariableList를 LData로 변환
 * 파라미터(스칼라 값) 처리
 */
public static LData convertVariableListToLData(VariableList varList) {
    LData data = new LData();

    for (int i = 0; i < varList.count(); i++) {
        String name = varList.key(i);
        String value = varList.getValue(i);
        data.setString(name, value);
    }

    return data;
}
```

**용도:**
- 단일 파라미터 값 (usid, deptCode 등)
- 검색 조건, 키 값 등

### 6.2 LData → VariableList 변환

```java
/**
 * LData를 MiPlatform VariableList로 변환
 */
public static VariableList convertLDataToVariableList(LData data) {
    VariableList varList = new VariableList();

    for (String key : data.getKeys()) {
        varList.setValue(key, data.getString(key));
    }

    return varList;
}
```

---

## 7. 성능 고려사항

### 7.1 대용량 데이터 처리

```java
// 대용량 데이터 변환 시 메모리 주의
public static LMultiData convertToLMultiDataWithJobType(Dataset ds) {
    LMultiData mData = new LMultiData();

    // 행 수가 많을 경우
    int rowCount = ds.getRowCount();
    if (rowCount > 10000) {
        // 로깅 또는 경고
        logger.warn("Large dataset conversion: " + rowCount + " rows");
    }

    // ... 변환 로직
    return mData;
}
```

### 7.2 메모리 관리

```java
// Dataset 변환 후 명시적 해제 (선택적)
public void execute() {
    Dataset inputDs = platformRequest.getDataset("ds_Input");
    LMultiData input = MiplatformConverter.convertToLMultiDataWithJobType(inputDs);

    // inputDs는 더 이상 필요 없음 (GC 대상)
    inputDs = null;

    // 비즈니스 로직 처리
    // ...
}
```

---

## 8. 예외 처리

### 8.1 변환 오류 처리

```java
public static LMultiData convertToLMultiDataWithJobType(Dataset ds) {
    try {
        LMultiData mData = new LMultiData();
        // ... 변환 로직
        return mData;
    } catch (Exception e) {
        logger.error("Dataset conversion failed: " + e.getMessage(), e);
        throw new MiplatformConversionException(
            "Failed to convert Dataset to LMultiData", e);
    }
}
```

### 8.2 타입 불일치 처리

```java
// 타입 변환 실패 시 기본값 처리
public static Object convertValue(Object value, int targetType) {
    try {
        switch (targetType) {
            case LMetaData.INT:
                return Integer.parseInt(value.toString());
            case LMetaData.DECIMAL:
                return new BigDecimal(value.toString());
            case LMetaData.DATE:
                return parseDate(value.toString());
            default:
                return value.toString();
        }
    } catch (Exception e) {
        // 변환 실패 시 원본 값 반환
        logger.warn("Type conversion failed: " + value + " to type " + targetType);
        return value;
    }
}
```

---

## 9. 라이선스 및 주의사항

### LG CNS 저작권

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

**주의사항:**
- 이 클래스는 LG CNS 소유
- 변경 시 DevOn 개발담당자에게 통보 필요
- 외부 유출 금지

---

## 10. 요약

### MiplatformConverter의 역할

```
┌────────────────────────────────────────────────────────────────┐
│                    MiplatformConverter                         │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  변환 방향 1: 수신 (Client → Server)                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Dataset → LMultiData                                     │  │
│  │  • convertToLMultiDataWithJobType()                      │  │
│  │  • _CUD 필드 추가 (C/U/D 상태 표시)                      │  │
│  │  • 삭제된 행 처리                                         │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  변환 방향 2: 응답 (Server → Client)                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  LMultiData → Dataset                                     │  │
│  │  • convertToDataset()                                    │  │
│  │  • 메타데이터 기반 컬럼 생성                              │  │
│  │  • 데이터 행 추가                                         │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  부가 기능:                                                     │
│  • 타입 변환 (STRING/INT/DECIMAL/DATE/BLOB)                  │
│  • VariableList ↔ LData 변환                                 │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### 핵심 포인트

| 항목 | 설명 |
|------|------|
| **역할** | MiPlatform Dataset ↔ DevOn LData/LMultiData "번역기" |
| **핵심 메소드** | `convertToLMultiDataWithJobType()`, `convertToDataset()` |
| **핵심 필드** | `_CUD` (Create/Update/Delete 상태) |
| **타입 처리** | 양쪽 타입 시스템 간 자동 변환 |
| **위치** | Client-Server 경계에서 변환 수행 |

### 결론

**MiplatformConverter는 DevOn 프레임워크의 핵심 연결고리입니다.**

- MiPlatform 클라이언트와 DevOn 서버가 **다른 데이터 모델**을 사용
- 이 둘을 연결하는 **중재자(Adapter)** 역할
- 변환을 통해 양쪽에서 일관된 데이터 처리 가능

---

*MiplatformConverter는 DevOn Framework의 핵심 컴포넌트로, MiPlatform과 DevOn 간의 원활한 데이터 교환을 가능하게 합니다.*
