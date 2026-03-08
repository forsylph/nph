# AZ_SYS03100M SVN 로그조회 실행체인

## 1. 목적

약어/용어는 [약어-용어집.md](../030.index/0303.약어-용어집/약어-용어집.md) 를 먼저 보면 빠르다.

이 문서는 `AZ_SYS03100M.xml` 화면에서 `SVN 로그 조회`가 어떻게 이어지는지 `화면 -> navigation -> command -> PC -> SVN 저장소` 기준으로 정리한 trace 문서다.

## 2. 상위 구조에서 이 문서를 읽는 위치

- 이 문서는 [../032.framework-core/0325.Version Control and Deployment/Draft.버전관리-배포관리-드래프트.md](../032.framework-core/0325.Version%20Control%20and%20Deployment/Draft.%EB%B2%84%EC%A0%84%EA%B4%80%EB%A6%AC-%EB%B0%B0%ED%8F%AC%EA%B4%80%EB%A6%AC-%EB%93%9C%EB%9E%98%ED%94%84%ED%8A%B8.md)의 `SvnLog` 사례를 실제 화면 기준으로 분리한 문서다.
- front dispatch는 [../031.front-channel/0312.navigation-command/A.Command-Navigation-Dispatch.md](../031.front-channel/0312.navigation-command/A.Command-Navigation-Dispatch.md)를 같이 본다.
- DevOn 코어 흐름은 [../032.framework-core/0321.overview/A.Framework-개요.md](../032.framework-core/0321.overview/A.Framework-%EA%B0%9C%EC%9A%94.md)를 같이 본다.

## 3. 대표 진입 경로

```mermaid
flowchart LR
    UI[AZ_SYS03100M.xml]
    MHI["/az/com/comnNavi/RetrieveSvnLog.mhi"]
    NAVI[comnNavi.xml]
    CMD[RetrieveSvnLogCMD]
    PC[SvnLogPC]
    SVN[svn.nph.go.kr]

    UI --> MHI --> NAVI --> CMD --> PC --> SVN
```

- 화면 파일:
  - [`AZ_SYS03100M.xml`](N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/webapp/ui/AZ/SYS/AZ_SYS03100M.xml)
- 화면 함수:
  - [`AZ_SYS03100M.xml:965`](N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/webapp/ui/AZ/SYS/AZ_SYS03100M.xml:965) `fRetrieveMaster2()`
- 호출 URL:
  - [`AZ_SYS03100M.xml:975`](N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/webapp/ui/AZ/SYS/AZ_SYS03100M.xml:975) `/az/com/comnNavi/RetrieveSvnLog.mhi`
- callback:
  - [`AZ_SYS03100M.xml:678`](N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/webapp/ui/AZ/SYS/AZ_SYS03100M.xml:678) `case "RetrieveSvnLog"`

## 4. command / PC / UC / EC

### 4.1 command

- navigation action:
  - [`comnNavi.xml:149`](N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/devonhome/navigation/mhi/az/com/comnNavi.xml:149)
  - `action name="RetrieveSvnLog"`
- command:
  - [`RetrieveSvnLogCMD.java:25`](N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/src/nph/his/az/com/comn/cmd/RetrieveSvnLogCMD.java:25)
  - `TxServiceUtil.getNTxService("az.comn.SvnLogPC")`

### 4.2 PC

- PC 구현:
  - [`SvnLogPC.java`](N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/src/nph/his/az/com/comn/pc/SvnLogPC.java)
- instance 매핑:
  - [`instance_az.properties:215`](N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/devonhome/instance/instance_az.properties:215)
  - `az.comn.SvnLogPC = nph.his.az.com.comn.pc.SvnLogPC`
- 인터페이스:
  - [`SvnLogIFPC.java`](N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/src/nph/his/az/com/comn/pc/SvnLogIFPC.java)

### 4.3 UC

- 이 trace 범위에서는 중심이 아니다.

### 4.4 EC

- 이 사례는 일반 `EC -> xmlquery -> DB` 체인보다 `PC -> SVN 저장소` 체인이 핵심이다.
- 즉 DevOn command/PC 구조를 사용하지만, 최종 접근 대상은 DB가 아니라 SVN 저장소다.

## 5. query path -> xmlquery

- 이 사례는 `xmlquery` 중심 trace가 아니다.
- `RetrieveSvnLogCMD`는 `SvnLogPC`를 호출하고, `SvnLogPC`는 SVNKit API를 통해 저장소에 직접 접근한다.
- 확인 근거:
  - [`SvnLogPC.java:10`](N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/src/nph/his/az/com/comn/pc/SvnLogPC.java:10) `org.tmatesoft.svn.*`
  - [`SvnLogPC.java:39`](N:/99.SourceCode%20Backup/NPH/AADEV_NPH/workspace/NPH_HIS/src/nph/his/az/com/comn/pc/SvnLogPC.java:39) `svn://svn.nph.go.kr/nph/trunk/NPH_HIS/`

## 6. 해석

- 이 사례는 일반 업무 화면과 다르게 `DB 조회`가 아니라 `SVN 저장소 조회`를 수행한다.
- 따라서 버전관리/배포관리 문서를 읽을 때 매우 좋은 증거가 된다.
- 또한 `instance_*.properties`가 단순 설정이 아니라, 운영 화면에서 호출되는 PC wiring이라는 점도 같이 보여준다.

## 7. 다시 올라갈 문서

- 버전관리/배포 draft:
  - [../032.framework-core/0325.Version Control and Deployment/Draft.버전관리-배포관리-드래프트.md](../032.framework-core/0325.Version%20Control%20and%20Deployment/Draft.%EB%B2%84%EC%A0%84%EA%B4%80%EB%A6%AC-%EB%B0%B0%ED%8F%AC%EA%B4%80%EB%A6%AC-%EB%93%9C%EB%9E%98%ED%94%84%ED%8A%B8.md)
- front-channel:
  - [../031.front-channel/0313.ui-entry/A.Front-Channel-개요.md](../031.front-channel/0313.ui-entry/A.Front-Channel-%EA%B0%9C%EC%9A%94.md)
- framework-core:
  - [../032.framework-core/0321.overview/A.Framework-개요.md](../032.framework-core/0321.overview/A.Framework-%EA%B0%9C%EC%9A%94.md)
