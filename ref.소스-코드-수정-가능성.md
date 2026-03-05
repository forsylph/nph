# NPH 프로젝트 소스 코드 현황 및 수정 가능성 분석

> 실제 소스 코드 존재 여부 및 수정 가능성 평가

---

## 1. 소스 코드 현황

### 1.1 파일 존재 확인

| 항목 | 내용 |
|------|------|
| **소스 경로** | `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace/` |
| **Java 파일** | **9,921개** |
| **전체 소스** | **17,771개** (Java, XML, JSP, JS 등) |
| **프로젝트** | COMMON, NPH_BAT, NPH_BUILD, NPH_ECS, NPH_HIS |

### 1.2 주요 프로젝트 구조

```
/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace/
├── COMMON/                    # 공통 라이브러리
│   └── src/devonx/nph/
│       ├── miplatform/
│       │   └── MiplatformConverter.java    # 핵심 변환 클래스
│       ├── system/
│       │   ├── servlet/
│       │   │   ├── MiplatformServlet.java
│       │   │   └── GeneralServlet.java
│       │   └── cmd/
│       │       └── AbstractMiplatformCommand.java
│       ├── data/
│       │   ├── LData.java
│       │   └── LMultiData.java
│       └── util/
│           └── LActionContext.java
│
├── NPH_HIS/                   # 병원정보시스템 (메인)
│   └── src/nph/his/
│       ├── az/
│       │   ├── auth/
│       │   ├── bizcom/
│       │   └── user/
│       ├── md/               # 진료
│       ├── mr/               # 원무
│       ├── sp/               # 검사
│       └── er/               # 응급
│
├── NPH_ECS/                   # 전자동의시스템
│   └── src/BKSNP/EMR/
│       ├── ExternalDBPusher.java
│       └── TBL_PROC_ocsapabh.java
│
└── NPH_BAT/                   # 배치
```

---

## 2. 코드 수정 가능성

### 2.1 기술적 수정 가능성: ✅ 가능

| 구분 | 상태 | 설명 |
|------|------|------|
| **파일 접근** | ✅ 가능 | 읽기/쓰기 권한 있음 (777) |
| **파일 수정** | ✅ 가능 | root 권한으로 수정 가능 |
| **빌드 환경** | ⚠️ 확인 필요 | Eclipse, JEUS, JDK 1.7 필요 |
| **테스트 환경** | ⚠️ 확인 필요 | MiPlatform, JEUS 서버 필요 |

### 2.2 실제 파일 권한

```bash
# 파일 권한 확인 결과
$ ls -la MiplatformConverter.java
-rwxrwxrwx 1 root root 43715 Jul 13  2017 MiplatformConverter.java
```

- **권한**: `-rwxrwxrwx` (777)
- **소유자**: root
- **수정 가능**: ✅ 예

---

## 3. 법적/계약적 제약

### 3.1 LG CNS 라이선스 주의사항

```java
/*
 * Copyright ⓒ LG CNS, Inc. All rights reserved.
 *
 * DevOn 의 클래스를 실제 프로젝트에 사용하는 경우 DevOn 개발담당자에게
 * 프로젝트 정식명칭, 담당자 연락처(Email)등을 mail로 알려야 한다.
 *
 * 소스를 변경하여 사용하는 경우 DevOn 개발담당자에게
 * 변경된 소스 전체와 변경된 사항을 알려야 한다.
 * 저작자는 제공된 소스가 유용하다고 판단되는 경우 해당 사항을 반영할 수 있다.
 *
 * (주의!) 원저자의 허락없이 재배포 할 수 없으며
 * LG CNS 외부로의 유출을 하여서는 안 된다.
 */
```

### 3.2 수정 시 의무사항

| 행위 | 의무사항 |
|------|----------|
| **사용 시** | DevOn 개발담당자에게 프로젝트명, 담당자 연락처 통보 |
| **소스 변경 시** | 변경된 소스 **전체**와 변경사항을 DevOn 개발담당자에게 알려야 함 |
| **재배포** | ❌ **금지** (원저자 허락 없이) |
| **외부 유출** | ❌ **금지** |

### 3.3 수정 가능 범위

```
수정 가능 (내부 사용 목적)
├── ✅ 비즈니스 로직 (Command, PC)
├── ✅ XML Query (SQL)
├── ✅ Navigation 설정
├── ✅ MiPlatform 화면 (XML)
└── ✅ JSP 파일

주의 필요 (DevOn 코어)
├── ⚠️ MiplatformConverter.java (저작권 주석 유지)
├── ⚠️ AbstractMiplatformCommand.java
├── ⚠️ LQueryService.java
└── ⚠️ 기타 DevOn 프레임워크 클래스

수정 후 의무
└── 📧 변경사항을 DevOn 개발담당자에게 통보 (devon@lgcns.com)
```

---

## 4. 수정 작업 시 고려사항

### 4.1 빌드 환경

```
필수 환경
├── JDK 1.7.0_80
├── Eclipse (구버전 권장)
├── JEUS 6.0 (WAS)
├── MiPlatform 3.2/3.3
└── Oracle/Tibero DB

빌드 산출물
├── nphCom.jar (COMMON)
├── NPH_HIS.war
└── NPH_ECS.war
```

### 4.2 테스트 환경

| 환경 | 필요 여부 | 설명 |
|------|-----------|------|
| **JEUS 서버** | 필수 | WAS 구동 필요 |
| **MiPlatform 클라이언트** | 필수 | 화면 테스트 |
| **Oracle DB** | 필수 | 데이터 연동 |
| **테스트 데이터** | 권장 | 실제 운영 데이터와 분리 |

### 4.3 버전 관리

```bash
# 권장: 수정 전 백업
$ cp MiplatformConverter.java MiplatformConverter.java.bak.$(date +%Y%m%d)

# Git으로 버전 관리 (권장)
$ git init
$ git add .
$ git commit -m "Initial NPH source backup"
$ git branch feature/modification-name
```

---

## 5. 수정 작업 예시

### 5.1 비즈니스 로직 수정 (Command)

```java
// 수정 전: nph.his.az.bizcom.cmcd.cmd.RetrievePbhlCdCMD
public class RetrievePbhlCdCMD extends AbstractMiplatformCommand {
    public void execute() {
        LMultiData result = pc.retrievePbhlCdList(param);
        platformResponse.addDataset("ds_Result", result);
    }
}

// 수정 후: 조건 추가
public class RetrievePbhlCdCMD extends AbstractMiplatformCommand {
    public void execute() {
        // 추가: 사용 여부 체크
        String useYn = paramData.getString("useYn");
        if (useYn == null || useYn.isEmpty()) {
            useYn = "Y";  // 기본값 Y
        }
        paramData.setString("useYn", useYn);

        LMultiData result = pc.retrievePbhlCdList(paramData);
        platformResponse.addDataset("ds_Result", result);
    }
}
```

**의무사항**: 변경된 소스를 DevOn 개발담당자에게 통보

### 5.2 XML Query 수정

```xml
<!-- 수정 전: cmcd.xml -->
<query name="retrievePbhlCdList">
    <statement>
        SELECT * FROM PBHL_CD_MASTER
        WHERE USE_YN = 'Y'
    </statement>
</query>

<!-- 수정 후: 조건 추가 -->
<query name="retrievePbhlCdList">
    <statement>
        SELECT PBHL_CD, PBHL_NM, USE_YN
        FROM PBHL_CD_MASTER
        WHERE USE_YN = '${useYn}'
        ORDER BY PBHL_CD
    </statement>
</query>
```

**의무사항**: 변경사항 기록 후 통보

### 5.3 DevOn 코어 클래스 수정 (주의!)

```java
// MiplatformConverter.java 수정 시 주의사항

/*
 * Copyright ⓒ LG CNS, Inc. All rights reserved.
 *
 * 변경사항:
 * - 2024-03-05: 대용량 데이터 변환 시 메모리 최적화 추가
 * - by: xxx
 */

package devonx.nph.miplatform;

public class MiplatformConverter {

    // 기존 코드 유지

    // 추가: 메모리 최적화 로직
    private static final int MAX_ROW_COUNT = 10000;

    public static LMultiData convertToLMultiDataWithJobType(Dataset ds) {
        // 대용량 데이터 체크
        if (ds.getRowCount() > MAX_ROW_COUNT) {
            throw new DataSizeException("Row count exceeds limit: " + ds.getRowCount());
        }
        // ... 기존 로직
    }
}
```

**의무사항**:
- 저작권 주석 유지
- 변경사항 주석 추가
- **필수**: 변경된 소스 전체를 DevOn 개발담당자에게 통보

---

## 6. 수정 작업 체크리스트

### 수정 전
- [ ] 백업 생성
- [ ] 수정 목적 명확화
- [ ] 영향 범위 분석
- [ ] 테스트 계획 수립

### 수정 중
- [ ] 저작권 주석 유지 (DevOn 클래스)
- [ ] 변경사항 주석 기록
- [ ] 코드 컨벤션 준수
- [ ] 중요 로직 보존

### 수정 후
- [ ] 단위 테스트
- [ ] 통합 테스트
- [ ] 성능 테스트 (대용량)
- [ ] **DevOn 개발담당자 통보** (필수)
- [ ] 문서화

---

## 7. 결론

### 코드 수정 가능성 평가

| 구분 | 평가 | 설명 |
|------|------|------|
| **기술적 가능성** | ✅ **가능** | 파일 접근/수정 권한 있음 |
| **법적 가능성** | ⚠️ **조건부 가능** | 통보 의무 있음 |
| **실무적 가능성** | ⚠️ **환경 의존** | 빌드/테스트 환경 필요 |
| **권장 여부** | ⚠️ **신중하게** | DevOn 코어는 최소 수정 |

### 핵심 메시지

> **소스 코드 수정은 기술적으로 가능하나, LG CNS 라이선스 조건을 준수해야 합니다.**

**가능한 수정:**
- ✅ 비즈니스 로직 (Command, PC)
- ✅ XML Query (SQL)
- ✅ Navigation 설정
- ✅ 화면 파일 (MiPlatform XML, JSP)

**주의가 필요한 수정:**
- ⚠️ DevOn 프레임워크 클래스 (저작권 주석 유지, 통보 필수)

**불가능한 행위:**
- ❌ 외부 유출
- ❌ 무단 재배포

### 권장사항

1. **DevOn 코어 클래스는 수정하지 말 것** (MiplatformConverter, AbstractMiplatformCommand 등)
2. **비즈니스 로직만 수정** (Command, PC, XML Query)
3. **변경사항 반드시 기록** (버전 관리 + 통보)
4. **테스트 환경에서 충분히 검증 후 운영 반영**

---

*NPH 프로젝트의 소스 코드는 실제로 존재하며 수정 가능하지만, LG CNS 라이선스 조건을 반드시 준수해야 합니다.*
