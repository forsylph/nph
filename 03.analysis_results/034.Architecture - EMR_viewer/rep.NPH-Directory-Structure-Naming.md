# NPH 프로젝트 디렉토리 구조 및 네이밍 룰

> 국립의료원 병원정보시스템(NPH) 프로젝트 구조 분석
> 분석 일자: 2026-03-07

---

## 1. 최상위 디렉토리 구조

### 1.1 프로젝트 모듈

| 디렉토리 | 용도 |
|----------|------|
| **NPH_HIS** | 병원정보시스템 (Hospital Information System) - 메인 애플리케이션 |
| **NPH_ECS** | 전자동의시스템 (Electronic Chart System) - EMR/동의 연동 |
| **NPH_BAT** | 배치 처리 모듈 |
| **NPH_BUILD** | 빌드 설정 및 리소스 |
| **COMMON** | 공통 라이브러리 (devonx 프레임워크 확장) |
| **Servers** | 서버 설정 파일 |

### 1.2 프로젝트 네이밍 규칙

```
패턴: NPH_<모듈명>
예시: NPH_HIS, NPH_ECS, NPH_BAT, NPH_BUILD

공통 모듈: COMMON (대문자, 단일 단어)
```

---

## 2. 소스코드 디렉토리 구조

### 2.1 Java 소스 구조

```
src/
├── nph/                        # 메인 NPH 애플리케이션
│   ├── app/                    # 애플리케이션 계층
│   │   ├── emp/               # 직원 모듈
│   │   └── pat/               # 환자 모듈
│   ├── bat/                    # 배치 처리
│   │   ├── az/                # 입원 배치
│   │   │   ├── biz/
│   │   │   ├── cmd/
│   │   │   ├── job/
│   │   │   └── reader/
│   │   ├── common/
│   │   ├── er/                # 응급 배치
│   │   ├── hp/                # 병원 배치
│   │   ├── md/                # 진료 배치
│   │   ├── mr/                # 의무기록 배치
│   │   └── sp/                # 특수 배치
│   └── his/                    # HIS 비즈니스 로직
│       ├── az/                # 입원 (Admission Zone)
│       ├── er/                # 응급 (Emergency Room)
│       ├── hp/                # 병원 (Hospital)
│       ├── md/                # 진료 (Medical)
│       ├── mr/                # 의무기록 (Medical Record)
│       └── sp/                # 특수 (Special)
├── com/                        # REST API, 유틸리티
└── BKSNP/                      # 레거시 ECS 코드 (NPH_ECS)
    └── EMR/
```

### 2.2 패키지 네이밍 규칙

| 하위 패키지 | 용도 |
|-----------|------|
| `cmd` | Command 클래스 (컨트롤러) |
| `pc` | Presentation Component |
| `ec` | Enterprise Component |
| `vo` | Value Object |
| `util` | 유틸리티 클래스 |
| `common` / `comn` | 공통 코드 |
| `constants` | 상수 정의 |
| `job` | 배치 작업 |
| `reader` | 데이터 리더 |
| `parser` | 데이터 파서 |
| `biz` | 비즈니스 로직 |

### 2.3 비즈니스 모듈 약어

| 약어 | 한글명 | 영문명 |
|------|--------|--------|
| `az` | 입원 | Admission Zone |
| `er` | 응급 | Emergency Room |
| `hp` | 병원 | Hospital |
| `md` | 진료 | Medical |
| `mr` | 의무기록 | Medical Record |
| `sp` | 특수 | Special |

---

## 3. 파일 네이밍 규칙

### 3.1 Java 파일

#### Command 클래스 (CMD 접미사)

```
패턴: <액션><엔티티>CMD.java

예시:
- CheckLoginNewCMD.java        # 로그인 확인
- RetrieveActvListCMD.java    # 활성 목록 조회
- SaveActvBdgtMngmListCMD.java # 활동 예산 저장
- DeleteMntrRprtCMD.java      # 모니터링 보고서 삭제
- GetUserInfoCMD.java         # 사용자 정보 조회
```

#### Presentation 클래스 (PC 접미사)

```
패턴: <엔티티>PC.java 또는 <엔티티>IFPC.java

예시:
- LoginPC.java          # 로그인 프레젠테이션
- LoginIFPC.java        # 로그인 인터페이스 (IF = Interface)
- HomePagePC.java       # 홈페이지 프레젠테이션
- MdclQiMngmPC.java     # 의료 품질 관리
```

#### Enterprise 클래스 (EC 접미사)

```
패턴: <엔티티>EC.java

예시:
- LoginEC.java          # 로그인 엔터프라이즈
- HomePageEC.java       # 홈페이지 엔터프라이즈
```

#### Utility 클래스

```
패턴: <목적>UTIL.java 또는 <목적>Util.java

예시:
- DateUTIL.java         # 날짜 유틸리티
- EmailUTIL.java        # 이메일 유틸리티
- StringUtil.java       # 문자열 유틸리티
- TxServiceUtil.java    # 트랜잭션 서비스 유틸리티
```

#### 레거시 ECS 클래스 (NPH_ECS)

```
패턴: BK_<목적>.java 또는 <목적>_<액션>.java

예시:
- BK_logMsg.java           # 로그 메시지
- BK_util.java             # 유틸리티
- BK_dbConnection.java     # DB 연결
- DB_CONN.java             # DB 연결 (상수형 네이밍)
- TBL_PROC_ocsapabh.java   # 테이블 프로시저
```

### 3.2 JSP 파일

#### 페이지 네이밍

```
패턴: <목적>[_<서브타입>].jsp

예시:
- index.jsp            # 진입 페이지
- index-new.jsp        # 진입 페이지 (변형)
- batch.jsp            # 배치 페이지
- opEbb.jsp            # 수술 게시판
- refreshsql.jsp       # SQL 새로고침
- FaxLogin.jsp         # 팩스 로그인 (PascalCase)
```

#### 부분 페이지 (Partial)

```
패턴: <엔티티>_<액션>.part.jsp

예시:
- job_exec_detail_log.part.jsp      # 작업 실행 상세 로그
- job_group_list.part.jsp           # 작업 그룹 목록
- job_editor_content.part.jsp       # 작업 에디터 내용
```

#### 모듈별 페이지

```
패턴: <모듈>_<엔티티>_<액션>.jsp

예시:
- Admin_AjaxCommon.jsp    # 관리자 Ajax 공통
- Admin_Common.jsp        # 관리자 공통
```

### 3.3 JavaScript 파일

#### 라이브러리 파일

```
패턴: <라이브러리>[-<버전>].js 또는 <라이브러리>.min.js

예시:
- jquery-1.3.2.min.js           # jQuery 라이브러리
- jquery-ui-1.7.custom.min.js   # jQuery UI
- prototype.js                   # Prototype.js
- json2.js                      # JSON 라이브러리
- common.js                     # 공통 함수
```

#### 모듈별 스크립트

```
패턴: <모듈>_<기능>.js 또는 <기능>_view.js

예시:
- scheduler.js               # 스케줄러
- exec_log_view.js          # 실행 로그 뷰
- job_group_management.js   # 작업 그룹 관리
```

#### DHTMLX 그리드 확장

```
패턴: dhtmlxgrid_excell_<타입>.js

예시:
- dhtmlxgrid_excell_calendar.js   # 달력 셀
- dhtmlxgrid_excell_combo.js      # 콤보박스 셀
```

### 3.4 설정 파일

#### XML 설정

```
패턴: <목적>.xml 또는 <프레임워크>.xml

예시:
- web.xml                  # Java EE 표준
- log4j.xml               # 로깅 설정
- config.xml               # 애플리케이션 설정
- sql.xml                  # SQL 쿼리 정의
- sqlParameter.xml        # SQL 파라미터
- jeus-web-dd.xml          # JEAS 배포 설정
- devon-core.xml          # 프레임워크 설정
- his.xml                  # HIS 모듈 설정
```

#### Properties 파일

```
패턴: <목적>.properties 또는 instance_<모듈>.properties

예시:
- config.properties        # 설정
- message.properties      # 메시지
- instance_app.properties # 앱 인스턴스
- instance_az.properties  # 입원 인스턴스
- DataSource.properties   # 데이터소스
- DataConnection.properties # DB 연결
```

---

## 4. 웹 리소스 구조

### 4.1 webapp 디렉토리 구조

```
webapp/
├── WEB-INF/
│   ├── classes/           # 컴파일된 클래스
│   ├── conf/              # 설정 파일
│   ├── homepath/          # 홈 경로 설정
│   ├── keys/              # 보안 키
│   ├── lib/               # JAR 라이브러리
│   ├── web.xml            # 웹 배포 설정
│   ├── sql.xml            # SQL 정의
│   └── config.properties
│
├── Admin/                 # 관리자 모듈
│   ├── jsp/
│   │   ├── common/
│   │   ├── doc/
│   │   ├── img/
│   │   ├── mapper/
│   │   ├── system/
│   │   └── term/
│   └── dhtmlx/            # DHTMLX 라이브러리
│
├── batchMgr/              # 배치 관리자
│   ├── js/
│   └── jsp/
│
├── eView/                 # 전자 뷰어 (EMR)
│   ├── EdViewer.jsp
│   ├── RecordViewer.jsp
│   ├── common/
│   └── popup/
│
├── images/                # 정적 이미지
│
├── jsp/                   # 메인 JSP
│   ├── anstRecViewer/
│   ├── emrtest/
│   ├── include/
│   ├── ir/               # 인터페이스
│   ├── lst/              # 목록
│   ├── md_mobile/        # 모바일
│   ├── report/
│   ├── sp_ray/
│   └── sys/
│
├── tpr/                   # TPR 리포트
│
├── ui/                    # Miplatform UI
│   ├── AZ/, ER/, HP/, MD/, MR/, SP/  # 모듈 UI
│   ├── MAIN/, ROOT/, TEST/
│   ├── LIBs/
│   └── *.xml, *.res       # 리소스
│
├── Miplatform320/         # Miplatform 3.2
├── Miplatform330/         # Miplatform 3.3
│
├── sga/                   # SGA 모듈
├── sso/                   # Single Sign-On
├── itsm/                  # IT 서비스 관리
└── api/                   # REST API
```

### 4.2 UI 리소스 구조

```
ui/
├── AZ/                    # 입원 UI
├── ER/                    # 응급 UI
├── HP/                    # 병원 UI
├── MD/                    # 진료 UI
├── MR/                    # 의무기록 UI
├── SP/                    # 특수 UI
├── MAIN/                  # 메인 UI
├── ROOT/                  # 루트 UI
├── TEST/                  # 테스트 UI
├── LIBs/                  # UI 라이브러리
├── *.xml                  # UI 정의 파일
└── NPH.res                # 리소스 파일
```

---

## 5. 특수 디렉토리

### 5.1 설정 디렉토리

| 디렉토리 | 용도 |
|----------|------|
| `devonhome/conf/` | Devon 프레임워크 설정 |
| `devonhome/conf/project/` | 프로젝트별 설정 (his.xml, nph_bat.xml) |
| `devonhome/conf/repConf/` | 리포트 설정 |
| `devonhome/instance/` | 인스턴스별 설정 |
| `devonhome/message/` | 메시지 리소스 |
| `devonhome/navigation/` | 네비게이션 설정 |
| `devonhome/validation/` | 유효성 검사 규칙 |
| `devonhome/xmlquery/` | XML 쿼리 정의 |
| `webapp/WEB-INF/conf/` | 웹 애플리케이션 설정 |
| `webapp/WEB-INF/homepath/` | 홈 경로 설정 |

### 5.2 라이브러리

| 위치 | 용도 |
|------|------|
| `webapp/WEB-INF/lib/` | 웹 애플리케이션 라이브러리 (JAR) |
| `NPH_BUILD/libs/` | 빌드 타임 라이브러리 |
| `ui/LIBs/` | UI 라이브러리 |

### 5.3 빌드 출력

| 디렉토리 | 용도 |
|----------|------|
| `NPH_ECS/build/classes/` | 컴파일된 클래스 |
| `NPH_HIS/bin/` | 바이너리 출력 |
| `NPH_BUILD/bin/` | 빌드 바이너리 |
| `NPH_BAT/bin/` | 배치 바이너리 |
| `.settings/` | Eclipse 설정 |
| `reports/` | 빌드 리포트 (PMD) |

---

## 6. NPH_HIS vs NPH_ECS 비교

| 항목 | NPH_HIS | NPH_ECS |
|------|---------|---------|
| **패키지 구조** | `nph.his.<모듈>` | `BKSNP.EMR` |
| **프레임워크** | Devon 프레임워크 | 레거시/서블릿 기반 |
| **네이밍 스타일** | 구조화됨, 모듈 기반 | 평면적, 절차적 |
| **Java 네이밍** | 접미사 (CMD, PC, EC) | 접두사 (BK_), 대문자 |
| **설정** | 다수의 XML 설정 | 단순 DB 연결 |
| **UI** | Miplatform UI | ActiveX/HTML |

---

## 7. 클래스 타입 접미사 요약

| 접미사 | 용도 | 예시 |
|--------|------|------|
| `CMD` | Command (컨트롤러) | `CheckLoginNewCMD.java` |
| `PC` | Presentation Component | `LoginPC.java` |
| `EC` | Enterprise Component | `LoginEC.java` |
| `IF` | Interface | `LoginIFPC.java` |
| `VO` | Value Object | `UserVO.java` |
| `UTIL` | Utility | `DateUTIL.java` |

---

## 8. 네이밍 규칙 요약

### 8.1 디렉토리 네이밍

```
프로젝트: NPH_<모듈명> (대문자, 언더스코어 구분)
패키지: 소문자, 계층 구조 (nph.his.<모듈>)
기능 디렉토리: 소문자 (cmd, pc, ec, vo, util, biz, job)
```

### 8.2 파일 네이밍

```
Java 클래스: PascalCase + 접미사 (CMD, PC, EC, VO, UTIL)
JSP: snake_case 또는 camelCase (목적_서브타입.jsp)
JavaScript: camelCase 또는 library-version.min.js
XML: 소문자, 목적 기반 (config.xml, sql.xml)
Properties: 소문자, 목적 기반 (message.properties)
```

### 8.3 한글 업무 용어 약어

| 약어 | 한글명 | 비즈니스 모듈 예시 |
|------|--------|-------------------|
| `mdcr` | 진료 | 진료 관련 |
| `mdcrsupt` | 진료지원 | 진료 지원 |
| `ptaf` | 환자업무 | 환자 관련 |
| `prscmstrmngm` | 처방마스터관리 | 처방 관리 |
| `ptsafemngm` | 환자안전관리 | 환자 안전 |
| `mdclqimngm` | 의료질향상관리 | 의료 품질 |
| `schdmngm` | 일정관리 | 스케줄 관리 |
| `ptmngm` | 환자관리 | 환자 관리 |

---

*NPH 프로젝트는 Devon 프레임워크 기반의 구조화된 네이밍 규칙을 따르며, HIS와 ECS는 서로 다른 아키텍처 스타일을 가집니다.*