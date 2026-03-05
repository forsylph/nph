# NPH 프로젝트 분석 보고서

> 국립의료원 병원정보시스템(NPH) 소스코드 분석
> 분석일: 2025-03-05

**📊 상세 분석 보고서**: [detailed-analysis.md](detailed-analysis.md) - 다이어그램 포함 상세 설명

---

## 개요

| 항목 | 내용 |
|------|------|
| **프로젝트명** | NPH (National XXX Hospital Information System) |
| **분석 대상** | `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace` |
| **총 파일 수** | 약 17,800개 |
| **구성 프로젝트** | COMMON, NPH_ECS, NPH_HIS, NPH_BAT, NPH_BUILD |

---

## 기술 스택

### 프레임워크

| 구분 | 기술 | 버전 | 비고 |
|------|------|------|------|
| UI 플랫폼 | **MiPlatform** | 3.2 / 3.3 | 투비소프트(구 토비소프트) |
| 백엔드 프레임워크 | **DevOn Framework** | 3.0 ~ 4.1 | LG CNS |
| WAS | JEUS | 6.0 | 티맥스소프트 |
| DB | Oracle / Tibero | - | - |
| JDK | Java | 1.7.0_80 | - |

### MiPlatform 버전 현황 (정량 분석)

| 프로젝트 | JAR | 실행 환경 | 비고 |
|---------|-----|----------|------|
| COMMON | 3.2 | 3.2 | 단일 버전 |
| NPH_HIS | 3.2 | **3.3 (97.2%)** | **주력 버전** |
| NPH_ECS | - | - | 미사용 |

#### NPH_HIS MiPlatform 버전 상세

**JSP 파일 기준 정량 분석 (총 395개)**

| 버전 | 파일 수 | 비율 | 주요 용도 |
|------|---------|------|-----------|
| **3.3** | 11개 | **2.8%** | 설치/인덱스/로그인 |
| **3.2** | 7개 | **1.8%** | SP-Ray(영상의학팀) |
| 미지정 | 377개 | 95.4% | 업무 로직 (인덱스에 따라 버전 결정) |

**주요 진입점 버전 현황**

| 진입점 | 버전 | 설명 |
|--------|------|------|
| `index.jsp` | 미지정 | 기본 진입점 (런처 호출) |
| `index330.jsp` | **3.3** | 주력 인덱스 (권장) |
| `index320.jsp` | **3.2** | 레거시 인덱스 |
| `login-new.jsp` | **3.3** | 주력 로그인 |
| `login-new320.jsp` | **3.2** | 레거시 로그인 |
| `jsp/sp_ray/index.jsp` | **3.2** | 영상의학팀 전용 |

**결론**: MiPlatform 3.3이 주력이며, 3.2는 영상의학팀(SP-Ray) 일부에서만 유지됩니다.

---

## MiPlatform 아키텍처 분석

### 매뉴얼 기준 구조 vs 실제 프로젝트 매핑

#### 1. AppGroup 구조

| 매뉴얼 개념 | 실제 프로젝트 | 예시 |
|-------------|---------------|------|
| AppGroup | `/webapp/ui/NPH_start.xml` | `<AppGroup Prefix="AZ_COM" Type="form">` |
| Prefix | 업무 코드 | AZ_COM, MD_OPN, SP |
| Type | form / js / Bs / File | form: 화면, js: 라이브러리 |
| BaseUrl | 상대 경로 | AZ/COM/, SP/ |

**실제 AppGroup 목록 (NPH_start.xml):**
- LIBs (js): 공통 JavaScript 라이브러리
- com (form): 공통 폼
- ROOT (form): 루트 폼
- MAIN (form): 메인 화면
- AZ_COM, AZ_UTL: 공통업무
- ER_ACC: 회계/재무
- MD_OPN: 외래진료
- MR_COM: 원무/수납
- SP: 검사/방사선

#### 2. Form (XML) 구조

**매뉴얼 구조:**
```xml
<Window>
  <Form Id="화면ID" Title="제목">
    <Datasets>...</Datasets>
    <Grid .../>
    <Button .../>
    <Script><![CDATA[...]]></Script>
  </Form>
</Window>
```

**실제 예시 (Login3.xml):**
```xml
<Form Height="802" Id="SP_CEL01010M" Title="검체인수확인"
      OnLoadCompleted="Form_OnLoadCompleted"
      OnKeyDown="Form_OnKeyDown">
  <Datasets>
    <Dataset Id="ds_PtInfo" UseClientLayout="1">
      <colinfo id="pid" type="STRING"/>
    </Dataset>
  </Datasets>
  <Grid BindDataset="ds_PtInfo" .../>
  <Script><![CDATA[...]]></Script>
</Form>
```

**파일 위치:** `/webapp/ui/{업무}/{화면코드}.xml`
- 예: `ui/AZ/COM/AZ_COM99005M.xml` (휴일관리)
- 예: `ui/SP/CEL/SP_CEL01010M.xml` (검체인수확인)

#### 3. Dataset 사용 패턴

| 속성 | 설명 | 예시 |
|------|------|------|
| `Id` | 데이터셋 식별자 | ds_PtInfo, ds_List |
| `DataSetType` | 유형 | Dataset |
| `UseClientLayout` | 클라이언트 레이아웃 | 1 |
| `_rowStatus` | 행 상태 | C(생성), U(수정), D(삭제) |

**Transaction 연결:**
```javascript
Transaction("SavePbhlCd",
  "/az/bizcom/cmcdNavi/SaveDetail.mhi",
  "ds_detail=ds_PbhlCd:u",
  "",
  "",
  "fTrCallBack");
```

#### 4. MiPlatform → DevOn 연동 플로우

```
[MiPlatform Client]
       │
       ▼ cf_Transaction()
[MiplatformServlet] (*.mhi)
       │
       ├── beforeCatchService()
       │      ├── new MiplatformRequest(req)
       │      │      └── receiveData()
       │      │             ├── VariableList 파싱
       │      │             └── DatasetList 파싱
       │      ├── new MiplatformResponse(res)
       │      └── ActionContext 저장
       │
       └── catchService()
              ├── Command.invoke()
              │      ├── AbstractMiplatformCommand()
              │      │      └── validateByRequisite()
              │      └── execute()
              │             ├── getMultiDatasetWithJobType()
              │             │      └── Dataset → LMultiData 변환
              │             ├── PC 호출 (TxServiceUtil)
              │             └── addDataset(result)
              │                   └── LMultiData → Dataset 변환
              └── sendData
                     └── HTTP 응답
```

---

## 프로젝트 구조

```
workspace/
├── COMMON/                 # 공통 라이브러리 (nphCom.jar)
├── NPH_BAT/                # 배치 작업
├── NPH_BUILD/              # 빌드 설정
├── NPH_ECS/                # 전자동의시스템 (EMR 연동)
└── NPH_HIS/                # 병원정보시스템 (메인)
    ├── Miplatform320/      # MiPlatform 3.2
    ├── Miplatform330/      # MiPlatform 3.3
    ├── devonhome/          # DevOn Framework 설정
    └── webapp/
        ├── EMR_DATA/       # EMR 데이터 처리
        └── ui/             # MiPlatform 화면 파일
```

---

## DevOn Framework (LG CNS) 심층 분석

### 개요

| 항목 | 내용 |
|------|------|
| **프레임워크** | DevOn Framework |
| **개발사** | LG CNS, Inc. |
| **버전** | 3.0.0 ~ 4.1.0 |
| **연락처** | devon@lgcns.com |
| **홈페이지** | http://www.dev-on.com |
| **목적** | 엔터프라이즈 시스템 개발 표준화 및 생산성 향상 |

### DevOn Framework 설계 목적

DevOn Framework는 다음을 위해 설계되었습니다:

1. **표준화**: 일관된 아키텍처와 코딩 규칙 적용
2. **생산성**: 반복적 코드 작성 최소화
3. **유지보수성**: 선언적 설정으로 변경 용이
4. **확장성**: 모듈식 구조로 기능 확장 가능
5. **MiPlatform 연동**: RIA 시스템 개발 지원

---

### DevOn 실행 플로우

#### 전체 아키텍처 플로우

```
[Client] → [Servlet] → [Interceptor] → [Command] → [PC] → [DAO] → [DB]
              ↑___________________________↓
                        Navigation
```

#### 상세 실행 순서

**1. HTTP Request 수신**
```
1.1 사용자가 *.mhi URL 호출
1.2 web.xml → MiPlatformChannelServlet 매핑
1.3 MiplatformServlet.doPost() 실행
```

**2. Request/Response 초기화**
```
2.1 beforeCatchService()
    ├── new MiplatformRequest(HttpServletRequest)
    │   └── PlatformRequest 생성
    │   └── receiveData() - MiPlatform 프로토콜 파싱
    │       ├── VariableList (파라미터)
    │       └── DatasetList (데이터셋)
    ├── new MiplatformResponse(HttpServletResponse)
    └── ActionContext에 저장 (ThreadLocal)
```

**3. Navigation 라우팅**
```
3.1 Navigation XML 로딩
    예: /az/bizcom/cmcdNavi/SaveDetail.mhi
        → az/bizcom/cmcdNavi.xml

3.2 Action 매핑
    <action name="SaveDetail">
        <command>nph.his.az.bizcom.cmcd.cmd.SaveDetailCMD</command>
        <interceptor>defaultStack</interceptor>
    </action>
```

**4. Interceptor Chain 실행**
```
4.1 defaultStack 순차 실행
    ├── loginCheck      : 로그인 여부 확인
    ├── converter       : 데이터 변환
    └── command         : Command 실행

4.2 각 Interceptor는 doIntercept()로 체인 진행
```

**5. Command 실행**
```
5.1 AbstractMiplatformCommand 생성자
    ├── platformRequest/Response 획득
    ├── paramData 추출
    └── Requisite 유효성 검증

5.2 execute() 메소드 실행
    ├── Input Dataset 변환
    │   └── Dataset → LMultiData (CUD 구분)
    ├── PC (Process Component) 호출
    │   └── TxServiceUtil.getNTxService()
    ├── 비즈니스 로직 실행
    └── Output Dataset 변환
        └── LMultiData → Dataset
```

**6. Transaction 처리**
```
6.1 TxServiceUtil을 통한 트랜잭션 관리
    ├── Transaction 시작
    ├── 비즈니스 로직 실행
    └── Commit 또는 Rollback

6.2 JDBC 기반 트랜잭션
    (devon-framework.xml의 transaction 매니저)
```

**7. XML Query 실행**
```
7.1 LQueryService를 통한 쿼리 실행
    └── devonhome/xmlquery/*.xml

7.2 예: app/emp/login.xml
    <statement name="retrieveUserInfo">
        SELECT * FROM users WHERE id = ${usid}
    </statement>
```

**8. Response 전송**
```
8.1 catchService() finally 블록
    └── platformRes.sendData
        ├── 결과 Dataset 변환
        └── HTTP Response 전송
```

---

### DevOn-MiPlatform 데이터 변환 플로우

```
[MiPlatform Client]
       │ Dataset
       ▼
[MiplatformRequest]
       │ receiveData()
       ▼
[PlatformRequest]
       │ getDatasetList()
       ▼
[MiplatformConverter]
       │ convertToLMultiDataWithJobType()
       ▼
[LMultiData] ──┐
               │ Command.execute()
               ▼
[PC] ──────────┤
               │
               ▼
[MiplatformConverter]
       │ convertToDataset()
       ▼
[Dataset] → [MiplatformResponse] → [Client]
```

---

### 핵심 컴포넌트

### 핵심 컴포넌트

#### 1. Servlet 계층

| 클래스 | 파일 경로 | 설명 |
|--------|-----------|------|
| `GeneralServlet` | `COMMON/src/devonx/nph/system/servlet/GeneralServlet.java` | 일반 HTTP 요청 처리 |
| `MiplatformServlet` | `COMMON/src/devonx/nph/system/servlet/MiplatformServlet.java` | MiPlatform 연동 서블릿 |

#### 2. Command 계층

| 클래스 | 파일 경로 | 설명 |
|--------|-----------|------|
| `AbstractMiplatformCommand` | `COMMON/src/devonx/nph/system/cmd/AbstractMiplatformCommand.java` | MiPlatform 연동 Command 기본 클래스 |
| `LBlobImageCommand` | `NPH_HIS/src/nph/his/core/cmd/LBlobImageCommand.java` | BLOB 이미지 출력 |
| `LFileDownLoadCommand` | `NPH_HIS/src/nph/his/core/cmd/LFileDownLoadCommand.java` | 파일 다운로드 |

#### 3. MiPlatform 연동

| 클래스 | 파일 경로 | 핵심 기능 |
|--------|-----------|-----------|
| `MiplatformConverter` | `COMMON/src/devonx/nph/miplatform/MiplatformConverter.java` | Dataset ↔ LData/LMultiData 변환 |
| `MiplatformRequest` | `COMMON/src/devonx/nph/miplatform/MiplatformRequest.java` | MiPlatform 요청 처리 |
| `MiplatformResponse` | `COMMON/src/devonx/nph/miplatform/MiplatformResponse.java` | MiPlatform 응답 처리 |

### 설정 파일 구조

```
devonhome/
├── conf/
│   ├── devon.xml                 # 메인 설정
│   ├── devon-core.xml            # Core 설정
│   ├── devon-framework.xml       # Framework 설정
│   ├── requisite.xml             # 입력값 검증
│   └── product/
│       ├── devon-batch-core.xml
│       └── devon-batch-scheduler.xml
├── navigation/                   # Navigation 설정
│   ├── his/                      # 병원정보시스템
│   ├── mhi/                      # MiPlatform 네비게이션
│   ├── img/                      # 이미지업무
│   └── ajax/                     # AJAX 네비게이션
└── xmlquery/                     # XML 기반 SQL
    ├── app/
    ├── md/
    └── er/
```

### 주요 설정 (devon-framework.xml)

#### DataSource 설정
```xml
<datasource name="default">
    <jndi-name>java:comp/env/jdbc/NPHDB</jndi-name>
</datasource>
<datasource name="encdb">
    <jndi-name>java:comp/env/jdbc/NPHENCDB</jndi-name>
</datasource>
```

#### Navigation 설정
```xml
<navigation name="his">
    <directory>#home/navigation/his</directory>
    <encoding-type>EUC-KR</encoding-type>
    <default-handler>forward</default-handler>
</navigation>
```

#### Interceptor Stack
```xml
<interceptor-stack name="defaultStack">
    <interceptor-ref name="loginCheck"/>
    <interceptor-ref name="converter"/>
    <interceptor-ref name="command"/>
</interceptor-stack>
```

### LG CNS 라이선스 주의사항

> **주의**: 소스를 변경하여 사용하는 경우 DevOn 개발담당자에게 변경된 소스 전체와 변경된 사항을 알려야 한다. 원저자의 허락없이 재배포 할 수 없으며 LG CNS 외부로의 유출을 하여서는 안 된다.
>
> (MiplatformConverter.java 저작권 주석)

---

## 프로젝트별 상세 분석

### 1. COMMON

**역할**: 공통 라이브러리 (nphCom.jar)

**주요 패키지**:
- `devonx.nph.miplatform.*`: MiPlatform 연동 클래스
- `devonx.nph.system.servlet.*`: GeneralServlet, MiplatformServlet
- `devonx.nph.util.*`: 공통 유틸리티

**의존성**:
- MiPlatform 3.2 (miplatform-3.2.jar)
- DevOn Framework (devon-core.jar, devon-framework.jar)

**배포**: NPH_HIS의 `WEB-INF/lib/`로 복사

---

### 2. NPH_ECS

**역할**: 전자동의시스템 (Electronic Consent System)

**특징**:
- MiPlatform 미사용
- 전자서명 (SignGate) 연동
- EDViewer (ActiveX/OCX) 사용

**주요 패키지**:
- `BKSNP.EMR.*`: EMR 데이터 연동 및 처리

**핵심 클래스**:
| 클래스 | 설명 |
|--------|------|
| `ExternalDBPusher.java` | EMR 외부 DB 연동 |
| `TBL_PROC_ocsapabh.java` | 입원 환자 진료 데이터 처리 |
| `BK_ajax.java` | EMR 데이터 처리 지원 |

**데이터소스**: `jdbc/NPHBKDB` (Tibero)

---

### 3. NPH_HIS

**역할**: 병원정보시스템 (Hospital Information System)

**특징**:
- DevOn Framework + MiPlatform 3.2/3.3
- 다중 데이터소스 (8개)
- 배치 관리 시스템 포함

**주요 모듈**:
| 모듈 | 설명 |
|------|------|
| AZ | 원무/접수 |
| ER | 응급 |
| HP | 입원 |
| MD | 진료 |
| MR | 의무기록 |
| SP | 외래 |

**데이터소스**:
```
jdbc/NPHDB      (메인)
jdbc/NPHENCDB   (암호화)
jdbc/NPHSMSDB   (SMS)
jdbc/NPHDIFDB   (DIF)
jdbc/NPHBKDB    (백업)
jdbc/NPHQCDB    (QC)
jdbc/NPHINFDB   (INF)
jdbc/NPHGW      (게이트웨이)
```

---

### 4. EMR 시스템 분석

EMR 관련 기능은 두 프로젝트에 분산되어 있습니다:

#### NPH_ECS: EMR 데이터 연동
- 외부 시스템 ↔ OCS/HIS 연동
- 전자서명 및 문서 저장
- EMR 데이터 전송/동기화

#### NPH_HIS: EMR 조회 및 진료
- EMR 뷰어 (EDViewer)
- 응급의료진료 관리
- 모바일 EMR 지원

**EMR 관련 경로**:
```
NPH_ECS/
├── src/BKSNP/EMR/          # EMR 데이터 처리 클래스
└── webapp/EMR_DATA/        # EMR 데이터 저장

NPH_HIS/
├── webapp/EMR_DATA/        # EMR 데이터 처리 JSP
└── webapp/jsp/md_mobile/emr/  # 모바일 EMR
```

---

## DevOn Framework (LG CNS)

### 출처 확인

해당 시스템은 **LG CNS의 DevOn Framework**를 기반으로 개발되었습니다.

**저작권 정보** (MiplatformConverter.java):
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

**버전 정보**:
- DevOn Framework: 3.0.0 ~ 4.1.0
- 개발사: LG CNS, Inc.
- 연락처: devon@lgcns.com
- 홈페이지: http://www.dev-on.com

### 라이선스 주의사항

> 소스를 변경하여 사용하는 경우 DevOn 개발담당자에게 변경된 소스 전체와 변경된 사항을 알려야 한다. 원저자의 허락없이 재배포 할 수 없으며 LG CNS 외부로의 유출을 하여서는 안 된다.

---

## 마이그레이션 분석

### 복잡도 평가

| 프로젝트 | 복잡도 | 주요 이슈 |
|----------|--------|-----------|
| COMMON | 중 | MiPlatform 3.2 의존성, DevOn Framework 의존성 |
| NPH_ECS | 하 | MiPlatform 미사용, SignGate/EDViewer 유지 필요 |
| NPH_HIS | 상 | MiPlatform 3.2/3.3 병행, DevOn Framework, 화면 수 다수 |

### Nexacro 사용 여부

**결과**: 사용되지 않음

- Nexacro 관련 파일 (.xfdl, 설정 파일): **0건**
- MiPlatform만 사용됨

---

## 참고 사항

### MiPlatform 수명 주기
- **MiPlatform 3.3**: 2025년 12월 31일 판매 종료 예정
- **권장**: Nexacro Platform으로의 마이그레이션

### 의존성 정리

```
NPH_HIS
├── COMMON (nphCom.jar)
│   ├── MiPlatform 3.2
│   └── DevOn Framework
├── MiPlatform 3.2/3.3
└── DevOn Framework

NPH_ECS
├── MiPlatform 미사용
└── SignGate (전자서명)
```

---

## 결론

1. **NPH 시스템**은 LG CNS의 DevOn Framework와 투비소프트의 MiPlatform을 기반으로 한 레거시 병원정보시스템입니다.

2. **MiPlatform 3.2와 3.3이 병행** 사용되고 있어 마이그레이션 시 두 버전을 모두 고려해야 합니다.

3. **NPH_ECS는 MiPlatform을 사용하지 않으나** SignGate/EDViewer 등 ActiveX 기반 컴포넌트가 있어 모던 브라우저 마이그레이션이 필요합니다.

4. **DevOn Framework는 LG CNS의 라이선스**를 가지므로 변경 시 통지 및 외부 유출 금지 등의 조건을 준수해야 합니다.

---

*분석 완료*
