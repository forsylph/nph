# EdiMngmPC 분기구조 분석

## 1. 목적

약어/용어는 [03.약어-용어집.md](../0310.index/03.%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91.md) 를 먼저 보면 빠르다.

이 문서는 `EdiMngmPC`가 왜 두꺼운 분기형 PC로 보이는지, 그리고 왜 당시 프레임워크 구조가 실제로 필요했을 가능성이 큰지 코드 기준으로 정리한 trace 문서다.

## 2. 상위 구조에서 이 문서를 읽는 위치

- 이 문서는 [../0311.overview/03.Architecture-overview.md](../0311.overview/03.Architecture-overview.md)의 EDI/분기형 사례다.
- command 흐름은 [../0312.front-channel/02.Command-Navigation-Dispatch.md](../0312.front-channel/02.Command-Navigation-Dispatch.md), `LCommonDao/LQueryMaker`는 [../0313.data-access/02.LCommonDao-LQueryMaker.md](../0313.data-access/02.LCommonDao-LQueryMaker.md)와 같이 보는 것이 좋다.

## 3. 대표 진입 경로

```mermaid
flowchart LR
    UI --> clamNavi
    clamNavi --> RetrieveEdiRecvRcpnCMD
    RetrieveEdiRecvRcpnCMD --> EdiMngmPC
    EdiMngmPC --> RecvEdiFileEC
    RecvEdiFileEC --> LCommonDao
```

- 화면 URL:
  - `/hp/dms/clamNavi/RetrieveEdiRecvRcpn.mhi`
  - `/hp/dms/clamNavi/SaveEdiRecvRcpn.mhi`
- navigation: `clamNavi.xml`
- command:
  - `RetrieveEdiRecvRcpnCMD`
  - `SaveEdiRecvRcpnCMD`

## 4. command / PC / UC / EC

### 4.1 command

- `RetrieveEdiRecvRcpnCMD` -> `TxServiceUtil.getNTxService("hp.dms.EdiMngmPC")`
- `SaveEdiRecvRcpnCMD` -> `TxServiceUtil.getTxService("hp.dms.EdiMngmPC")`

### 4.2 PC

`EdiMngmPC`
- `samFileId + version`에 따라 `sQuery`를 조립한다
- 복잡한 분기를 화면이나 command가 아니라 PC에 몰아두는 구조다

### 4.3 UC

- 현재 이 trace 범위에서는 핵심 UC보다 `PC -> EC`와 query family 선택이 중심이다
- 따라서 이 문서는 UC보다 `EdiMngmPC -> RecvEdiFileEC`를 기준 체인으로 본다

### 4.4 EC

`RecvEdiFileEC`
- `recvEdiFileEC.retrieveEdiRecvRcpn(data, sQuery, version)`
- 최종 실행은 `new LCommonDao(sQuery + "/retrieveEdiRecvRcpn" + version, data)`로 수렴한다

## 5. query path -> xmlquery

대표 예:

- `/hp/dms/hpdmhf201/retrieveEdiRecvRcpn073`
- `/hp/dms/hpdmhf402/retrieveEdiRecvRcpn071`
- `/hp/dms/hpdmhf604/retrieveEdiRecvRcpn089`
- `/hp/dms/hpdmhi010/retrieveEdiRecvRcpn070`
- `/hp/dms/hpdmht201/retrieveEdiRecvRcpn`

대표 xmlquery file family:

- `hpdmhf201.xml`, `hpdmhf402.xml`, `hpdmhf604.xml`
- `hpdmhi010.xml`
- `hpdmht201.xml`, `hpdmht204.xml`

## 6. 해석

- `EdiMngmPC`는 보기 싫게 두꺼운 PC가 맞다.
- 다만 이유 없는 두꺼움은 아니다.
- 이 PC는 파일유형 차이, 버전 차이, 후처리 차이, command 분리, query family 선택을 한곳으로 모아둔 구조다.
- 즉 보기 좋은 구조는 아니지만, 당시 입력 포맷 다양성을 화면 밖으로 밀어내기 위한 타협으로 보는 편이 더 맞다.

## 7. 다시 올라갈 문서

- 개요로 돌아가려면
  - [../0311.overview/01.Framework-개요.md](../0311.overview/01.Framework-%EA%B0%9C%EC%9A%94.md)
- command 구조로 돌아가려면
  - [../0312.front-channel/02.Command-Navigation-Dispatch.md](../0312.front-channel/02.Command-Navigation-Dispatch.md)
- `LCommonDao/LQueryMaker`로 연결하려면
  - [../0313.data-access/02.LCommonDao-LQueryMaker.md](../0313.data-access/02.LCommonDao-LQueryMaker.md)
- 설계 평가와 연결하려면
  - [../0315.design-review/02.설계평가-상세.md](../0315.design-review/02.%EC%84%A4%EA%B3%84%ED%8F%89%EA%B0%80-%EC%83%81%EC%84%B8.md)
