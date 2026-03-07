# EDViewer(eView) 팩트체크

> 기준 코드베이스: `N:\99.SourceCode Backup\NPH\AADEV_NPH\workspace`
> 목적: EDViewer 관련 문서에서 반복적으로 혼동된 항목을 실제 코드/리소스 기준으로 정리

---

## 1. 핵심 결론

| 항목 | 확인 결과 |
|------|-----------|
| EDViewer 소스 | 미확인. 현재 백업셋에서는 `OCX/EXE/DLL/APK` 바이너리만 확인 |
| 실제 JSP 진입점 | `NPH_HIS/webapp/eView/EdViewer.jsp`, `NPH_HIS/webapp/jsp/md_mobile/emr/EdViewer.jsp`, `NPH_HIS/webapp/jsp/md_mobile/emr/emrView.jsp` |
| 실제 서버 연동 클래스 | `NPH_ECS/src/BKSNP/EMR/ExternalDBPusher.java`, `ExternalDBPusher1.java` |
| 직접 미확인 항목 | `retrieveEmrData`, `saveEmrSignature`, `EMRDataServlet.java`, `emrViewer.jsp`, `NPH_ECS/webapp/jsp/emr/*` |
| 실제 호출 패턴 | `OBJECT classid=...` + `FV_CommonCall(...)` 흔적 확인 |
| 직접 미확인 호출 패턴 | `new ActiveXObject("EDViewer.Control")` 문자열은 현재 백업셋에서 미발견 |

---

## 2. 실제 확인된 파일

### 2.1 JSP / 화면

- `NPH_HIS/webapp/eView/EdViewer.jsp`
- `NPH_HIS/webapp/jsp/md_mobile/emr/EdViewer.jsp`
- `NPH_HIS/webapp/jsp/md_mobile/emr/emrView.jsp`

### 2.2 서버 연동 Java

- `NPH_ECS/src/BKSNP/EMR/ExternalDBPusher.java`
  - 확인 메서드: `CallExternalDB(...)`, `MakeLog(...)`
- `NPH_ECS/src/BKSNP/EMR/ExternalDBPusher1.java`
  - 확인 메서드: `CallExternalDB1(...)`

### 2.3 배포 바이너리

- `NPH_HIS/webapp/EMR_DATA/applet/EDViewer_Ocx.ocx`
- `NPH_HIS/webapp/EMR_DATA/applet/BKEDViewer.exe`
- `NPH_ECS/webapp/EMR_DATA/eMobile/EDViewer_Ocx.ocx`
- `NPH_ECS/webapp/EMR_DATA/eMobile/EDViewer_Ocx.cab`
- `NPH_ECS/webapp/EMR_DATA/eMobile/BKEDViewer.exe`
- `NPH_ECS/webapp/EMR_DATA/eMobile/HookDLL.dll`
- `NPH_ECS/webapp/EMR_DATA/eMobile/ISEDViewer(20).apk`
- `NPH_ECS/webapp/EMR_DATA/eMobile/ISEDViewer(21).apk`

---

## 3. 자주 혼동되는 항목

### 3.1 문서에는 있었지만 현재 백업셋에서 직접 못 찾은 것

- `retrieveEmrData`
- `saveEmrSignature`
- `EMRDataServlet.java`
- `emrViewer.jsp`
- `NPH_ECS/webapp/jsp/emr/*`
- `RetrieveEmrData.mhi`

이 항목들은 다른 저장소나 누락된 산출물에 있을 가능성은 남아 있지만, 현재 검토한 백업셋에서는 근거를 찾지 못했습니다.

### 3.2 문서 예시와 실제 확인값이 다른 것

- 문서 예시: `new ActiveXObject("EDViewer.Control")`
- 실제 확인: `EdViewer.jsp` 계열에서 `OBJECT classid=...` + `FV_CommonCall(...)`

- 문서 예시: `md/opn/emrView.jsp`
- 실제 확인: `jsp/md_mobile/emr/emrView.jsp`

- 문서 예시: `retrieveEmrData`, `saveEmrSignature`
- 실제 확인: `ExternalDBPusher*`의 `CallExternalDB*`

---

## 4. 사실 기반으로 보는 EDViewer 구조

```text
[NPH_HIS JSP]
  ├── eView/EdViewer.jsp
  ├── jsp/md_mobile/emr/EdViewer.jsp
  └── jsp/md_mobile/emr/emrView.jsp
            │
            ▼
     OBJECT classid / FV_CommonCall
            │
            ▼
     EDViewer_Ocx.ocx / BKEDViewer.exe
            │
            ▼
[NPH_ECS EMR Java]
  ├── ExternalDBPusher.CallExternalDB(...)
  └── ExternalDBPusher1.CallExternalDB1(...)
```

---

## 5. 문서 작성 가이드

EDViewer 관련 문서를 앞으로 쓸 때는 아래 규칙이 안전합니다.

1. `확인됨`과 `추정/예시`를 반드시 분리
2. 메서드명, JSP 경로, `.mhi` 액션명은 문자열 검색으로 실존 확인 후 기재
3. `EDViewer.Control` 같은 COM ProgID는 실제 문자열 확인 전까지 단정하지 않기
4. EDViewer 자체 동작 설명보다 `JSP -> 호출 래퍼 -> 바이너리 -> ECS 연동` 흐름을 우선 기술

---

## 6. 관련 문서

- `rep.EMR_viewer-EDViewer-overview.md`
- `rep.EMR_viewer-EDViewer-바이너리-분석.md`
- `tech-stack.md`


