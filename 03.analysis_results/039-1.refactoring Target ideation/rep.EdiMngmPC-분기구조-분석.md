# EdiMngmPC 분기구조 분석

## 1. 문서 목적

이 문서는 `EdiMngmPC`가 왜 두꺼운 분기형 PC로 보이는지, 그리고 왜 당시 프레임워크 구조가 실제로 필요했을 가능성이 큰지 코드 기준으로 정리한 문서다.

## 2. 핵심 결론

`EdiMngmPC`는 단순 조회 PC가 아니다.

- command는 얇다
- 실제 복잡도는 `EdiMngmPC`의 `samFileId + version` 분기에 있다
- 최종 query 실행은 `RecvEdiFileEC`의 `LCommonDao(sQuery + "/retrieveEdiRecvRcpn" + version, data)`로 수렴한다

즉 이 구조는 `괜히 복잡하게 만든 것`이라기보다, 파일유형/버전별 EDI 수신 포맷 차이를 화면 밖으로 밀어내기 위한 구조에 가깝다.

## 3. 진입 체인

- 화면 URL:
  - `/hp/dms/clamNavi/RetrieveEdiRecvRcpn.mhi`
  - `/hp/dms/clamNavi/SaveEdiRecvRcpn.mhi`
- navigation:
  - `clamNavi.xml`
- command:
  - `RetrieveEdiRecvRcpnCMD` -> `TxServiceUtil.getNTxService("hp.dms.EdiMngmPC")`
  - `SaveEdiRecvRcpnCMD` -> `TxServiceUtil.getTxService("hp.dms.EdiMngmPC")`
- PC:
  - `EdiMngmPC`
- EC:
  - `RecvEdiFileEC`

## 4. 실제 분기 규칙

기본 접두는 아래처럼 시작한다.

- `String sQuery = "/hp/dms/";`

그 다음 `samFileId`와 `version`에 따라 실제 query 파일군이 결정된다.

### 4.1 대표 분기 예

- `F020_1`
  - 기본 파일군: `hpdmhf201`
  - 버전 분기: `073`, `074`, `079`, `082`, `083`, `085`, `088`
- `F040_2`
  - 기본 파일군: `hpdmhf402`
  - 버전 분기: `071`, `072`, `073`, `074`, `083`, `085`, `079`, `088`
- `F060_4`
  - 기본 파일군: `hpdmhf604`
  - 버전 분기: `074`, `079`, `083`, `085`, `089`, `091`
- `I010`
  - 기본 파일군: `hpdmhi010`
  - 버전 분기: `old`, `070`
- `N020_1`
  - 기본 파일군: `hpdmht201`

## 5. PC -> EC -> query path

`EdiMngmPC.retrieveEdiRecvRcpn(data)`는 내부에서 `sQuery`를 조립한 뒤 아래 형태로 EC를 호출한다.

- `recvEdiFileEC.retrieveEdiRecvRcpn(data, sQuery, version)`

`RecvEdiFileEC` 쪽 실제 실행은 아래로 수렴한다.

- `new LCommonDao(sQuery + "/retrieveEdiRecvRcpn" + version, data)`

즉 최종 query path는 이런 식으로 만들어진다.

- `/hp/dms/hpdmhf201/retrieveEdiRecvRcpn073`
- `/hp/dms/hpdmhf402/retrieveEdiRecvRcpn071`
- `/hp/dms/hpdmhf604/retrieveEdiRecvRcpn089`
- `/hp/dms/hpdmhi010/retrieveEdiRecvRcpn070`
- `/hp/dms/hpdmht201/retrieveEdiRecvRcpn`

## 6. xmlquery 파일군

실제 `xmlquery`에는 동일한 statement family가 매우 넓게 퍼져 있다.

- `hpdmhf201.xml`
- `hpdmhf202.xml`
- `hpdmhf204.xml`
- `hpdmhf402.xml`
- `hpdmhf403.xml`
- `hpdmhf604.xml`
- `hpdmhi010.xml`
- `hpdmht201.xml`
- `hpdmht204.xml`
- 그 외 다수

그리고 각 파일 안에는 아래 statement가 반복된다.

- `retrieveEdiRecvRcpn`
- `retrieveEdiRecvRcpn071`
- `retrieveEdiRecvRcpn073`
- `retrieveEdiRecvRcpn074`
- `retrieveEdiRecvRcpn079`
- `retrieveEdiRecvRcpn083`
- `retrieveEdiRecvRcpn085`
- `retrieveEdiRecvRcpn088`
- `retrieveEdiRecvRcpn089`
- `retrieveEdiRecvRcpn091`

## 7. 해석

이 코드를 보면 `EdiMngmPC`는 보기 싫게 두꺼운 PC가 맞다. 다만 이유 없는 두꺼움은 아니다.

이 PC가 흡수하고 있는 것은 다음이다.

- EDI 파일유형 차이
- 수신 포맷 버전 차이
- 일부 파일군의 후처리 차이
- 저장/조회 command 분리
- 동일한 `LCommonDao` 표면 API로의 수렴

즉 이 구조가 없었다면 복잡도는 아래 중 하나로 퍼졌을 가능성이 높다.

- 화면 스크립트
- 여러 command 클래스
- EC 레벨의 난잡한 if 분기
- query 파일명을 직접 조합하는 중복 코드

## 8. 당시 왜 필요했을 가능성이 큰가

이 사례는 `LCommonDao/LQueryMaker` 구조가 과시용이 아니라는 점을 보여준다.

- 화면은 얇게 유지하고
- command는 표준화하고
- 복잡한 분기 선택은 PC에 모으고
- 최종 SQL 실행은 `LCommonDao`로 통일

이런 방향은 병원 SI처럼 파일유형, 버전, 기관 규칙이 많은 환경에서는 충분히 이해 가능한 선택이다.

지금 관점에서는 추적 비용이 크고 답답하지만, 당시에는 `복잡도를 한곳에 몰아두기 위한 타협`이었을 가능성이 높다.

## 9. 결론

`EdiMngmPC`는 기술부채 성격이 분명히 있다. 하지만 출발점이 무능해서 생긴 구조라고 보긴 어렵다.

더 정확히는:

- 문제를 숨겨서 무겁게 만든 구조
- 동시에, 실제 업무 분기를 화면 밖으로 밀어낸 구조

이다. 즉 보기 좋은 구조는 아니지만, 당시 입력 포맷 다양성과 query 파일군 규모를 보면 존재 이유는 분명히 있었다.
