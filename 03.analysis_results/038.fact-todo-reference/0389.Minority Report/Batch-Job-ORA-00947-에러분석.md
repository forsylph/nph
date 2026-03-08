# Batch Job ORA-00947 에러 분석 보고서

> **작성일**: 2026-03-08
> **분석 범위**: `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace/NPH_HIS/devonhome/logs/batch-file-log/`
> **에러 발생일**: 2026-02-02

---

## 1. 에러 개요

### 1.1 에러 정보

| 항목 | 값 |
|------|-----|
| **에러 코드** | ORA-00947 |
| **에러 메시지** | not enough values (값이 충분하지 않음) |
| **에러 유형** | SQL 구문 에러 (INSERT/MERGE 컬럼 불일치) |
| **영향받는 배치** | HP_BAT01206B (CreatePreRevwAmtCalc) |
| **발생 시각** | 2026-02-02 11:20:51, 11:21:35, 11:21:56, 11:23:00 |
| **발생 횟수** | 4회 연속 실패 |

### 1.2 배치 작업 정보

| 항목 | 값 |
|------|-----|
| **배치 ID** | HP_BAT01206B |
| **배치명** | CreatePreRevwAmtCalc |
| **기능** | 사전심사 금액 계산 배치 |
| **업무 영역** | HP (병원/심사) |
| **파라미터** | chosNo, clclDvsn, userId, insFlag, medStrDy, dschPrarDy, medDvsn |

---

## 2. 스택 트레이스 분석

### 2.1 전체 호출 체인

```
devon.core.exception.DevonException: Unexpected exception occurred.
  java.sql.SQLSyntaxErrorException: ORA-00947: not enough values

  at devon.core.exception.LExceptionPitcher.throwDevonException(Unknown Source)
  at devonframework.persistent.autodao.LCommonDao.executeUpdate(Unknown Source)
  at nph.his.hp.zzhp.hpfe.ec.OtptRcptEC.updateAdmsRcptReClcl(OtptRcptEC.java:796)
      ↑ 에러 발생 지점
  at nph.his.hp.com.uc.HpComnUC.setReClcl(HpComnUC.java:1732)
  at nph.his.hp.com.uc.StrtAdmsClclUC.strtAdmsClclMain(StrtAdmsClclUC.java:118)
  at nph.bat.hp.job.CreatePreRevwAmtCalc.execute(CreatePreRevwAmtCalc.java:65)
```

### 2.2 관련 소스 파일

| 파일 | 경로 | 라인 |
|------|------|------|
| `CreatePreRevwAmtCalc.java` | `nph/bat/hp/job/` | 65 |
| `StrtAdmsClclUC.java` | `nph/his/hp/com/uc/` | 118 |
| `HpComnUC.java` | `nph/his/hp/com/uc/` | 1732 |
| `OtptRcptEC.java` | `nph/his/hp/zzhp/hpfe/ec/` | 796 |
| `hppahidpt.xml` | `devonhome/xmlquery/hp/com/` | 613-657 |

---

## 3. 근본 원인 분석

### 3.1 SQL 문 분석

**실패하는 SQL**: `/hp/com/hppahidpt/updateAdmsRcptReClcl`

```sql
MERGE INTO HPFEHIPRC USING DUAL
  ON (CHOS_NO = ${chosNo} AND ACT_DY = SUBSTR(${actDy},0,8))
  WHEN MATCHED THEN
    UPDATE SET
      RE_CLCL_YN = ${reClclYn}
      {#1}                        -- 동적 컬럼
      , AMEN_ID = ${_amenId}
      , UPDT_DT = SYSDATE
  WHEN NOT MATCHED THEN
    INSERT (
      CHOS_NO,           -- 1
      ACT_DY,            -- 2
      RE_CLCL_YN,        -- 3
      CLCL_PICH_ID,      -- 4
      REGI_ID,           -- 5
      RGST_DT,           -- 6
      AMEN_ID,           -- 7
      UPDT_DT            -- 8
    ) VALUES (
      ${chosNo},         -- 1
      SUBSTR(${actDy},0,8),  -- 2
      ${reClclYn},       -- 3
      {#2}               -- 4번째 값 (동적)
      ${_regiId},        -- 5
      SYSDATE,           -- 6
      ${_amenId},        -- 7
      SYSDATE            -- 8
    )
```

### 3.2 동적 SQL 조건

**`{#2}` 플레이스홀더 교체 규칙**:

```xml
<append where="false" condition="${pichId}.EMPTY" id="#2">
    , NULL                      -- pichId가 없을 때
</append>
<append where="false" condition="${pichId}.NOTEMPTY" id="#2">
    , DECODE(${reClclYn}, 'N', NULL, ${pichId})  -- pichId가 있을 때
</append>
```

### 3.3 버그 원인

**Java 코드 분석** (`HpComnUC.java`, 1718-1739라인):

```java
// 배치 실행 시 호출 경로
public int setReClcl(String chosNo, String medDvsn, String actDy,
                      String batchYn, String pichId) {
    // ...
    if(sAmenId.equals("batch")){
        data.set("_clclId", sAmenId);        // ✅ 배치인 경우
    } else {
        data.set("usid", sAmenId);
        // ... 사용자 체크 로직
    }

    // pichId 설정이 누락됨!
    // batchYn = "Y"일 때 pichId 파라미터가 전달되지 않아
    // ${pichId}.EMPTY 조건이 실행되어야 하지만,
    // 실제로는 {#2} 플레이스홀더가 제대로 교체되지 않음

    data.set("pichId", pichId);   // ← 이 설정이 필요!
}
```

**버그 발생 시나리오**:

```
[정상 케이스 - UI에서 호출]
1. 사용자가 화면에서 "재계산" 버튼 클릭
2. pichId 파라미터가 전달됨
3. {#2} → DECODE(...) 또는 NULL로 정상 교체
4. INSERT 실행 성공

[버그 케이스 - 배치에서 호출]
1. CreatePreRevwAmtCalc 배치 실행
2. batchYn = "Y" 설정
3. pichId 파라미터가 null 또는 빈 문자열
4. {#2} 플레이스홀더가 제대로 교체되지 않음
5. ORA-00947: not enough values 발생
```

### 3.4 SQL 변수 매핑

| 변수명 | INSERT 컬럼 | 배치 실행 시 값 | 문제 여부 |
|--------|-------------|----------------|----------|
| `${chosNo}` | CHOS_NO | 내원번호 | 정상 |
| `${actDy}` | ACT_DY | 진료일자 | 정상 |
| `${reClclYn}` | RE_CLCL_YN | "Y" | 정상 |
| `${pichId}` | CLCL_PICH_ID | **NULL/빈값** | **문제** |
| `${_regiId}` | REGI_ID | 등록자ID | 정상 |
| `${_amenId}` | AMEN_ID | 수정자ID | 정상 |

---

## 4. 수정 방안

### 4.1 Java 코드 수정

**파일**: `HpComnUC.java`
**메서드**: `setReClcl()`

```java
// 수정 전 (배치 호출 시 pichId 설정 누락)
if(sAmenId.equals("batch")){
    data.set("_clclId", sAmenId);
} else {
    // ... 사용자 체크 로직
}
data.set("pichId", pichId);  // ← 이 라인이 필요

// 수정 후
if(sAmenId.equals("batch")){
    data.set("_clclId", sAmenId);
    data.set("pichId", "");   // ← 배치인 경우 빈 문자열 명시적 설정
} else {
    data.set("usid", sAmenId);
    // ... 사용자 체크 로직
    data.set("pichId", sPichId);  // ← 사용자인 경우 pichId 설정
}
```

### 4.2 배치 파라미터 확인

**파일**: `CreatePreRevwAmtCalc.java`

```java
public Object execute(JobContext context, Object[] parsedParamSet) throws Exception {
    StrtAdmsClclUC strtAdmsClclUC = new StrtAdmsClclUC();
    HpComnUC hpComnUC = new HpComnUC();

    LData lData = new LData();
    lData.setString("chosNo", chosNo);
    lData.setString("clclDvsn", clclDvsn);
    lData.setString("userId", userId);
    lData.setString("insFlag", insFlag);
    lData.setString("medStrDy", medStrDy);
    lData.setString("dschPrarDy", dschPrarDy);
    lData.setString("medDvsn", medDvsn);
    lData.setString("batchYn", HpConstants.YN_YES);
    // lData.setString("pichId", "");  // ← pichId 파라미터 추가 필요

    strtAdmsClclUC.strtAdmsClclMain(lData);
    return null;
}
```

### 4.3 XML 쿼리 수정 (대안)

**파일**: `hppahidpt.xml`

```xml
<!-- 수정 전 -->
<append where="false" condition="${pichId}.EMPTY" id="#2">
    , NULL
</append>

<!-- 수정 후 (NULL 직접 명시) -->
<append where="false" condition="${pichId}.EMPTY" id="#2">
    , NULL  -- CLCL_PICH_ID
</append>
```

---

## 5. 배치 실행 로그 분석

### 5.1 실패한 배치 인스턴스

| 배치 ID | 실행 시각 | 상태 |
|---------|----------|------|
| HP_BAT01206B-83 | 2026-02-02 11:20:51 | 실패 |
| HP_BAT01206B-84 | 2026-02-02 11:21:35 | 실패 |
| HP_BAT01206B-85 | 2026-02-02 11:21:56 | 실패 |
| HP_BAT01206B-86 | 2026-02-02 11:23:00 | 실패 |

### 5.2 배치 실행 시퀀스

```
[배치 시작]
  ↓
초기화 단계 (init)
  ↓
파라미터 설정 단계
  ↓
파일 처리 단계
  ↓
[에러 발생] ORA-00947
  ↓
DevonException 래핑
  ↓
BatchStackedException 발생
  ↓
[배치 종료 - 실패]
```

### 5.3 배치 파라미터 예시

```json
{
  "chosNo": "20260202E00001",
  "clclDvsn": "31",
  "userId": "8990004",
  "insFlag": "1",
  "medStrDy": "20260202",
  "dschPrarDy": "20260202",
  "medDvsn": "I"
}
```

---

## 6. 영향 범위

### 6.1 영향받는 기능

| 기능 | 영향도 | 설명 |
|------|--------|------|
| 사전심사 금액 계산 | **HIGH** | 배치 실행 불가 |
| 입원 재계산 처리 | **MEDIUM** | 재계산 플래그 업데이트 실패 |
| 심사 데이터 생성 | **HIGH** | HPFEHIPRC 테이블 INSERT 실패 |

### 6.2 영향받지 않는 기능

| 기능 | 설명 |
|------|------|
| 외래 재계산 | `updateOpRcptReClcl` 사용 (별도 쿼리) |
| UI 재계산 | pichId 파라미터가 정상 전달됨 |

---

## 7. 테스트 계획

### 7.1 수정 후 검증 항목

| 항목 | 검증 방법 |
|------|----------|
| 배치 실행 | HP_BAT01206B 실행 성공 확인 |
| 데이터 생성 | HPFEHIPRC 테이블 INSERT 확인 |
| UI 재계산 | 화면에서 재계산 정상 동작 확인 |
| pichId NULL | pichId 없이 실행 시 정상 처리 확인 |

### 7.2 회귀 테스트

```
[테스트 시나리오]
1. 배치 실행 (pichId=null)
   → HPFEHIPRC INSERT 성공
   → CLCL_PICH_ID 컬럼에 NULL 저장

2. UI 재계산 (pichId=사용자ID)
   → HPFEHIPRC INSERT 성공
   → CLCL_PICH_ID 컬럼에 사용자ID 저장

3. 기존 데이터 UPDATE
   → HPFEHIPRC UPDATE 성공
   → 재계산 플래그 Y로 변경
```

---

## 8. 기타 발견된 에러

### 8.1 배치 로그 내 추가 에러 패턴

| 에러 유형 | 발견 여부 | 비고 |
|-----------|----------|------|
| ORA-00947 | ✅ 발견 | 본 보고서 분석 대상 |
| ORA-00001 | 미발견 | 중복 키 에러 없음 |
| ORA-01400 | 미발견 | NULL 입력 에러 없음 |
| Java NullPointerException | 미발견 | 런타임 에러 없음 |

### 8.2 에러 로그 파일 목록

```
/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace/NPH_HIS/devonhome/logs/
├── batch-file-log/
│   ├── batchLog                          # 배치 시스템 로그
│   ├── devon-batch-syslog                # DevOn 배치 시스템 로그
│   └── 2026-02-02/
│       ├── HP_BAT01206B-83-HP_BAT01206B01.log  # 실패 로그 1
│       ├── HP_BAT01206B-84-HP_BAT01206B01.log  # 실패 로그 2
│       ├── HP_BAT01206B-85-HP_BAT01206B01.log  # 실패 로그 3
│       └── HP_BAT01206B-86-HP_BAT01206B01.log  # 실패 로그 4
```

---

## 9. 요약

### 9.1 문제 요약

| 항목 | 내용 |
|------|------|
| **에러** | ORA-00947: not enough values |
| **원인** | MERGE 문 INSERT VALUES 절의 컬럼 수 불일치 |
| **근본 원인** | 배치 실행 시 pichId 파라미터 누락 |
| **영향** | 사전심사 금액 계산 배치 실행 불가 |

### 9.2 수정 권장사항

| 우선순위 | 항목 | 파일 |
|----------|------|------|
| 1 | HpComnUC.setReClcl() 메서드 수정 | HpComnUC.java:1718-1739 |
| 2 | CreatePreRevwAmtCalc.execute() 파라미터 추가 | CreatePreRevwAmtCalc.java:47-68 |
| 3 | XML 쿼리 조건 명시화 | hppahidpt.xml:648-656 |

---

*작성자: Claude Code (서브에이전트 병렬 분석)*
*분석 일자: 2026-03-08*
*데이터 출처: Batch Job Logs (HP_BAT01206B)*