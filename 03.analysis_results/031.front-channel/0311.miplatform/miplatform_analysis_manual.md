# NPH 마이플랫폼(MiPlatform) 소스코드 분석 매뉴얼

## 1. 마이플랫폼 개요

### 1.1 MiPlatform이란?
- **개발사**: 토우소프트(現: Nexacro Platform)
- **버전**: MiPlatform 3.3 (일부 3.2)
- **특성**: X-Internet 기반의 RIA(Rich Internet Application) 플랫폼
- **기술 스택**: XML 기반 화면 정의 + JavaScript 로직

### 1.2 핵심 구성 요소
| 구성 요소 | 설명 |
|-----------|------|
| **MiPlatform Browser** | 런타임 브라우저 |
| **PID (Platform Integrated Developer)** | 전용 개발 도구 |
| **MiUpdater** | 배포 및 업데이트 관리 |
| **BSB** | PI 파일 변환 도구 |

---

## 2. NPH 프로젝트 구조

### 2.1 전체 디렉토리 구조
```
/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/
├── bin/                          # 실행 환경
│   ├── apache-tomcat-9.0.105/    # Tomcat 웹서버
│   ├── eclipse/                  # Eclipse IDE
│   ├── jdk1.7.0_80.x86/          # JDK 1.7
│   ├── jdk1.8.0_181.x86_64/      # JDK 1.8
│   └── JEUS7/                    # JEUS 웹서버
│
└── workspace/                    # 개발 소스코드
    ├── COMMON/                   # 공통 라이브러리
    ├── NPH_BAT/                  # 배치 프로젝트
    ├── NPH_BUILD/                # 빌드 설정
    ├── NPH_ECS/                  # 전자의무기록
    ├── NPH_HIS/                  # 메인 HIS 프로젝트
    ├── RemoteSystemsTempFiles/   # 임시파일
    └── Servers/                  # 서버 설정
```

### 2.2 NPH_HIS 웹앱 구조
```
NPH_HIS/webapp/
├── index330.jsp                  # 메인 진입점 (97.2%)
├── Miplatform330/install/        # 클라이언트 설치/업데이트
│   ├── Update_cfg.xml
│   ├── Update_component.xml
│   └── Update_etc.xml
├── ui/                           # 화면 정의
│   ├── NPH_start.xml             # AppGroup 설정 (핵심)
│   ├── LIBs/                     # 공통 JavaScript
│   │   ├── formCom.js            # 폼 공통 함수
│   │   ├── sysLib.js             # 시스템 라이브러리
│   │   ├── utilLib.js            # 유틸리티
│   │   └── script_*.js           # 업무별 스크립트
│   ├── AZ/                       # 관리 업무
│   ├── MD/                       # 모바일/외래
│   ├── MR/                       # 원무/수납
│   ├── SP/                       # 검사/방사선
│   ├── ER/                       # 응급
│   └── template/                 # 템플릿
├── eView/
│   └── dataMap.xml               # SQL 매핑
└── devonhome/                    # DEVON 프레임워크 설정
    ├── conf/                     # 설정 파일
    ├── navigation/               # 화면 플로우
    └── xmlquery/                 # XML 쿼리
```

---

## 3. 마이플랫폼 파일 형식

### 3.1 Form Definition 파일 (*.xml)
**경로**: `/NPH_HIS/webapp/ui/**/*.xml`

마이플랫폼 표준 확장자는 `.xfd`이나 NPH는 `.xml` 사용

**주요 구조**:
```xml
<?xml version="1.0" encoding="utf-8"?>
<Window>
  <Form Height="768" Id="FORM_ID" Title="화면명" Width="1024">
    <!-- 1. 데이터셋 정의 -->
    <Datasets>
      <Dataset DataSetType="Dataset" Id="_dsData">
        <Contents>
          <colinfo id="col1" size="50" type="STRING"/>
          <colinfo id="col2" size="10" type="INT"/>
        </Contents>
      </Dataset>
    </Datasets>

    <!-- 2. UI 컴포넌트 -->
    <Div Height="100" Id="dvTop">
      <Button Id="btnSearch" OnClick="dvTop_btnSearch_OnClick" Text="조회"/>
    </Div>

    <Grid AutoFit="TRUE" BindDataset="_dsData" Id="grList">
      <contents>
        <head>
          <cell col="0" display="text" text="컬럼1"/>
        </head>
        <body>
          <cell col="0" colid="col1" display="text"/>
        </body>
      </contents>
    </Grid>

    <!-- 3. 이벤트 스크립트 -->
    <Script><![CDATA[
      #include "LIBs::formCom.js";

      function FORM_ID_OnLoadCompleted(obj) {
        cf_OnLoadCompleted(obj);
      }
    ]]></Script>
  </Form>
</Window>
```

### 3.2 NPH_start.xml (AppGroup 설정)
**경로**: `/NPH_HIS/webapp/ui/NPH_start.xml`

**역할**:
- 애플리케이션 전역 설정
- 컴포넌트 등록
- 프로토콜 설정
- 업무별 AppGroup 정의

**주요 구조**:
```xml
<ConnectGroup
    device="Win32 (1024x768)"
    height="986"
    SessionURL="com::Login3.xml"
    Title="경찰병원의료정보시스템"
    width="1280">

  <!-- UI 컴포넌트 등록 -->
  <container Version="1000">
    <Component Id="Grid" Module="CyGrid" Name="cyGrid"/>
    <Component Id="TreeView" Module="CyTreeView" Name="cyTreeView"/>
    <Component Id="WebBrowser" Module="CyWebBrowser" Name="cyWebBrowser"/>
  </container>

  <!-- 통신 프로토콜 -->
  <protocols version="1000">
    <protocol id="http" name="cyHttpAdp" Compress="True" TimeOut="300"/>
    <protocol id="https" name="cyHttpAdp" TimeOut="300"/>
  </protocols>

  <!-- 업무별 AppGroup -->
  <AppGroups>
    <AppGroup Prefix="AZ_COM" Type="form">
      <Script Baseurl="AZ/COM/"/>
    </AppGroup>
    <AppGroup Prefix="MD_OPN" Type="form">
      <Script Baseurl="MD/OPN/"/>
    </AppGroup>
  </AppGroups>
</ConnectGroup>
```

### 3.3 파일명 패턴

| 패턴 | 의미 | 예시 |
|------|------|------|
| `XXXXX00000M.xml` | 메인 화면 | AZ_COM01001M.xml |
| `XXXXX00000P.xml` | 팝업 화면 | AZCOMF001P.xml |
| `XXXXX00000D.xml` | 다이얼로그 | - |

---

## 4. UI 컴포넌트 시스템

### 4.1 컴포넌트 분류

**입력/선택**:
- `Edit` - 텍스트 입력
- `MaskEdit` - 마스크 입력
- `Spin` - 숫자 스핀
- `Combo` - 드롭다운 선택
- `Radio` - 라디오 버튼
- `Checkbox` - 체크박스
- `Calendar` - 날짜 선택

**표시**:
- `Static` - 정적 텍스트
- `Image` - 이미지
- `TextArea` - 텍스트 영역

**데이터 표시**:
- `Grid` - 데이터 그리드
- `TreeView` - 트리뷰
- `List` - 리스트

**컨테이너**:
- `Div` - Division 컨테이너
- `Tab` - 탭 컨테이너
- `PopupDiv` - 팝업 컨테이너
- `WebBrowser` - 웹브라우저

**기능**:
- `Button` - 버튼
- `FileDialog` - 파일 다이얼로그
- `Progressbar` - 진행바
- `Chart` - 차트
- `MenuBar` - 메뉴

### 4.2 Dataset 주요 기능

**데이터 조작**:
```javascript
// 행 추가/삭제
ds.AddRow();
ds.DeleteRow(rowIdx);

// 데이터 접근
var value = ds.GetColumn(row, "colName");
ds.SetColumn(row, "colName", value);

// 정렬/필터링
ds.Sort("col1,col2", true);      // 오름차순
ds.Sort("col1:D,col2:D");        // 내림차순
ds.Filter("col1 > '1'");         // 필터 적용
ds.UnFilter();                    // 필터 해제

// 집계 함수
ds.Sum("col1");
ds.Avg("col1");
ds.Min("col1");
ds.Max("col1");
```

**트랜잭션 처리**:
```javascript
// 변경사항 저장
ds.ApplyChange();

// 행 상태 확인
var rowType = ds.GetRowType(row);  // 0:Normal, 1:Insert, 2:Update, 4:Delete

// 원본 데이터 복원
var orgValue = ds.GetOrgColumn(row, "colName");
ds.Reset();  // 원래 데이터로 되돌리기
```

---

## 5. JavaScript 라이브러리

### 5.1 공통 라이브러리 위치
**경로**: `/NPH_HIS/webapp/ui/LIBs/`

| 파일명 | 역할 |
|--------|------|
| `formCom.js` | 폼 공통 함수 |
| `formResize.js` | 화면 크기 조정 |
| `sysLib.js` | 시스템 라이브러리 |
| `utilLib.js` | 유틸리티 함수 |
| `objLib.js` | 객체 관련 함수 |
| `mdiLib.js` | MDI (다중 문서 인터페이스) |
| `script_az.js` | AZ 업무 스크립트 |
| `script_er.js` | ER 업무 스크립트 |
| `script_md.js` | MD 업무 스크립트 |
| `certLib.js` | 인증서 라이브러리 |

### 5.2 스크립트 포함 방식
```xml
<Script><![CDATA[
#include "LIBs::formCom.js";
#include "LIBs::utilLib.js";

function Form_OnLoadCompleted(obj) {
  cf_OnLoadCompleted(obj);
}
]]></Script>
```

---

## 6. 데이터 통신

### 6.1 트랜잭션 구조

**파일 위치**:
- 네비게이션: `/NPH_HIS/devonhome/navigation/`
- XML 쿼리: `/NPH_HIS/devonhome/xmlquery/`
- 데이터 매핑: `/NPH_HIS/webapp/eView/dataMap.xml`

### 6.2 통신 방식

| 방식 | 설명 |
|------|------|
| **비동기(Async)** | 콜백 함수에서 순차 처리 |
| **동기(Sync)** | Protocol Adapter 설정 또는 스크립트 전환 |

### 6.3 데이터 형식
- **XML** - 기본
- **BIN** - ZLIB 압축
- **CSV** - CSV 형식

---

## 7. 개발 환경 정보

| 항목 | 버전/설정 |
|------|-----------|
| 마이플랫폼 | 3.3 (일부 3.2) |
| JDK | 1.7, 1.8 |
| 웹서버 | Tomcat 9.0, JEUS 7 |
| IDE | Eclipse |
| 데이터베이스 | Oracle |
| 문자셋 | UTF-8, EUC-KR |

---

## 8. 소스코드 분석 체크리스트

### 8.1 Form 분석 시 확인 사항
- [ ] Form ID 및 Title 확인
- [ ] Dataset 정의 (컬럼, 타입, 크기)
- [ ] UI 컴포넌트 구조
- [ ] 이벤트 핸들러 정의
- [ ] #include 된 라이브러리
- [ ] AppGroup Prefix 확인

### 8.2 JavaScript 분석 시 확인 사항
- [ ] 공통 함수 호출 흐름
- [ ] Dataset 트랜잭션 처리
- [ ] 서버 통신 (Transaction)
- [ ] 팝업/다이얼로그 호출
- [ ] 에러 처리

### 8.3 업무 분류별 경로

| 업무 | 경로 | 설명 |
|------|------|------|
| AZ | `/ui/AZ/` | 관리/공통 |
| ER | `/ui/ER/` | 전자의무기록 |
| MD | `/ui/MD/` | 모바일/외래 |
| MR | `/ui/MR/` | 원무/수납 |
| SP | `/ui/SP/` | 검사/방사선 |

---

## 9. 참고 문서 링크

### 공식 문서
- [마이플랫폼 3.3 메인 문서](https://docs.tobesoft.com/miplatform_3_3)
- [초보자 자습서](https://docs.tobesoft.com/getting_started_miplatform_ko)
- [PID Developer 가이드](https://docs.tobesoft.com/pid_developer_guide_miplatform_ko)

### 커뮤니티/블로그
- [제타위키 - MiPlatform](https://zetawiki.com/wiki/MiPlatform)
- [Chapter 6. 컴포넌트별 주요 기능](https://capricasix.tistory.com/entry/MiPlatform-Chapter6-Component%EB%B3%84-%EC%A3%BC%EC%9A%94%EA%B8%B0%EB%8A%A52)
- [Chapter 7. 고급기능 및 Tip](https://capricasix.tistory.com/entry/MiPlatform-Chapter7-%EA%B3%A0%EA%B8%89%EA%B8%B0%EB%8A%A5-%EB%B0%8F-Tip1)

---

## 10. 특이사항 및 제약사항

1. **확장자 차이**: 표준 마이플랫폼의 `.xad`, `.xfd` 대신 `.xml` 사용
2. **인코딩**: EUC-KR과 UTF-8 혼용
3. **버전**: MiPlatform 3.3 (97.2%), 일부 3.2 (1.8%)
4. **통신**: HTTP/HTTPS, ZLIB 압축
5. **브라우저**: IE 브라우저 기반 (ActiveX)
6. **지원 종료**: 2025년 12월 31일 MiPlatform V3.3 판매 종료

---

*작성일: 2026-03-07*
*대상: /mnt/n/99.SourceCode Backup/NPH/AADEV_NPH*
