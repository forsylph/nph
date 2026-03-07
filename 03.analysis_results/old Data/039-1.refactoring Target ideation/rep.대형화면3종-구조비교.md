# NPH 대형 화면 3종 구조 비교

> MD_ORD01001P vs HP_DMS01303M vs HP_DMS02204M
> 분석일: 2026-03-07

---

## 1. 문서 목적

이 문서는 기존 `rep.대형화면3종-비교분석.md`의 성능 중심 비교와 분리해, 실제 유지보수 관점에서 `화면 -> navigation -> CMD -> PC -> EC -> xmlquery` 구조를 비교하기 위한 문서다.

## 2. 한눈 요약

| 항목 | MD_ORD01001P | HP_DMS01303M | HP_DMS02204M |
|------|--------------|--------------|--------------|
| 대표 navigation | `ptmdcrNavi` | `clamNavi` | `drgNavi` |
| 대표 command | `SavePtOrderCMD`, `UpdateDurtCMD`, `RetrievePtOrder*` | `RetrieveEdiRecvRcpnCMD`, `SaveEdiRecvRcpnCMD` | `RetrieveDrgRevwPtListCMD` |
| 대표 PC | `PrscMngmPC` | `EdiMngmPC` | `DrgPostRevwMngmPC` |
| 대표 EC | `CommonptPrscEC` | `RecvEdiFileEC` | `PostRevwEC` |
| xmlquery 성격 | `scninfo.xml` + `mdmdhtord.xml` | `hpdmhf* / hpdmhi* / hpdmht*` | `hpdmhdmbs.xml` |
| 구조 난점 | 시나리오 과적재 | 분기형 PC | 도메인 묶음 과밀 |

## 3. 화면별 구조 해석

### 3.1 MD_ORD01001P

- 기본주문 조회, 전처방 조회, 자동복사, 저장, DUR 보정이 한 화면에 함께 있다.
- 즉 화면이 큰 것만이 아니라, 서로 다른 사용자 시나리오가 한 화면 안에 과도하게 겹쳐 있다.
- `ptmdcrNavi -> CMD -> PrscMngmPC -> PrscSaveUC / PrscCheckDurUC -> CommonptPrscEC -> scninfo.xml / mdmdhtord.xml`

### 3.2 HP_DMS01303M

- 화면은 단순 조회형처럼 보이지만, 서버 쪽 `EdiMngmPC`가 `samFileId + version` 분기를 크게 품고 있다.
- 복잡도가 UI보다는 `query family 선택`에 몰려 있다.
- `clamNavi -> RetrieveEdiRecvRcpnCMD -> EdiMngmPC -> RecvEdiFileEC -> hpdmhf* / hpdmhi* / hpdmht*`

### 3.3 HP_DMS02204M

- 조회 화면처럼 보이지만 `DrgPostRevwMngmPC`, `PostRevwEC`, `hpdmhdmbs.xml`이 심사 후처리까지 함께 품고 있다.
- 즉 단일 조회 화면이라기보다 심사 도메인 묶음의 일부다.
- `drgNavi -> RetrieveDrgRevwPtListCMD -> DrgPostRevwMngmPC -> PostRevwEC -> hpdmhdmbs.xml`

## 4. 지금 관점에서의 판단

- `MD_ORD01001P`
  - 문제의 본체는 화면 과밀 + 시나리오 과적재다.
- `HP_DMS01303M`
  - 문제의 본체는 분기형 PC와 EDI 포맷 계열 다양성이다.
- `HP_DMS02204M`
  - 문제의 본체는 심사 도메인 로직이 조회 흐름과 같은 파일군에 묶여 있는 점이다.

즉 셋 다 "프레임워크가 무겁다"로 뭉뚱그리면 틀린다. 셋은 각각 다른 이유로 무겁다.

## 5. 연결 문서

- `rep.MD_ORD01001P-실행체인.md`
- `rep.EdiMngmPC-분기구조-분석.md`
- `rep.HP_DMS02204M-실행체인.md`
- `rep.HP_DMS01303M-심층분석.md`
- `rep.HP_DMS02204M-심층분석.md`
- `rep.대형화면3종-비교분석.md`
