# NPH 문서-코드베이스 전수점검 보고서

> 점검 기준 코드베이스: `N:\99.SourceCode Backup\NPH\AADEV_NPH\workspace`
> 점검 범위: `NPH` 하위 모든 `.md`, `.mmd`
> 점검일: 2026-03-06

---

## 1. 점검 대상

- Markdown 문서: 23개
- Mermaid 문서: 4개
- 총 점검 대상: 27개

---

## 2. 전체 결론

1. EDViewer 문서군은 현재 코드베이스 기준으로 상당 부분 정리되었으며, 별도 팩트체크 문서가 가장 신뢰도가 높다.
2. Framework/Architecture 문서군은 실코드 기반 요소와 개념 복원 요소가 섞여 있어, 현재는 `아키텍처 해설 문서`로 보는 것이 안전하다.
3. Patient Journey / MiPlatform Tree 문서는 예시성 서술이 많아 실운영 경로/클래스와 1:1 대응 문서로 사용하면 위험하다.
4. 성능 문서는 MiPlatform 프로토콜, 캐시, Connection Pool 수치 관련 단정이 일부 과했고, 현재는 상당 부분 보정되었다.
5. 대형 화면 심층분석 문서는 방향성은 유효하지만, 파일 크기/라인 수는 실제 측정값과 소폭 차이가 있었다.

---

## 3. 크리티컬 Findings

### 3.1 `LQueryService`를 실구현처럼 적은 문서들

현재 백업셋에서는 `LQueryService.java` 및 직접 참조를 확인하지 못했다.
반면 실제 업무 코드에서는 `LCommonDao(...).executeQuery()` 패턴이 광범위하게 관찰된다.

영향 문서:
- `README.md`
- `detailed-analysis.md`
- `03.analysis_results/031.Architecture - Framework/00.rep.DevOn- vs struts1 (뭘 만든거니).md`
- `03.analysis_results/031.Architecture - Framework/03.rep.DevOn_IO Layer-XML-Query-내부동작.md`

판단:
- `LQueryService`는 현재 코드베이스 기준 실존 확인 실패
- 완전 허구라고 단정할 수는 없으나, 현 백업셋 기준으로는 `개념적 XML Query 계층명`으로 취급해야 안전
- 실프로젝트의 표면 실행 진입점은 `LCommonDao` 쪽이 더 직접적임

### 3.2 Patient Journey 문서의 실경로/실클래스 오인 위험

`patient-journey-simulation.md`는 이미 상단에 개념 문서 경고가 있으나, 본문에 실제 구현처럼 보이는 `RegisterOutpatient.mhi`, `RegisterOutpatientCMD`, `ReceiptPC`, `receipt.xml` 예시가 남아 있다.

대표 예시:
- `/az/comn/receiptNavi/RegisterOutpatient.mhi`
- `RegisterOutpatientCMD.java`
- `ReceiptPC`

현재 점검 기준:
- `LoginUserCMD.java`는 실존 확인
- 위 예시들은 현재 백업셋에서 1:1 실파일 확인 실패

판단:
- 개념 설명 문서로는 유효
- 실운영 transaction map 문서로 사용하면 위험

### 3.3 `ref.MiPlatform-전체구조-트리.md`의 대표 예시가 실파일처럼 읽히는 문제

문서 상단과 Patient Journey 구간에 예시 표기가 들어가 있지만, 본문 상단 트리의 다수 화면/클래스/파일명이 현재 코드와 1:1 대응처럼 읽힐 수 있다.

특히 미확인 비중이 높은 항목:
- `SavePatientCMD.java`
- `PatientPC.java`
- `SaveOpnChartCMD.java`
- `LabPC.java`
- `PharmacyPC.java`
- `DischargeReceiptCMD.java`

판단:
- 트리 문서는 `구조적 이해용 대표 예시`로는 유효
- 코드 탐색 시작점 문서로 사용하면 오탐 유도 가능

---

## 4. 높은 우선순위 Findings

### 4.1 README의 XML Query 실행 설명

`README.md`는 프레임워크 개요 설명으로는 좋지만, 다음 구간은 현재 코드베이스 기준 과도하게 단정적이었다.
- `7.1 LQueryService를 통한 쿼리 실행`
- `/az/bizcom/cmcdNavi/SaveDetail.mhi`

판단:
- README는 입문 문서이므로, 가장 먼저 `실제 확인됨 / 개념 설명 / 미확인` 구분이 필요함

### 4.2 `detailed-analysis.md`의 실제 흐름 예시

`CheckLoginUser.mhi`, `SaveDetail.mhi`, `convertToLMultiDataWithJobType(Dataset ds)` 형태 등 일부 예시가 실구현과 다르다.

실제 확인:
- `MiplatformServlet`의 XML 응답 생성 코드 확인
- `MiplatformConverter.convertToLMultiDataWithJobType`는 `Dataset, sessionDs` 시그니처 확인

판단:
- 흐름 설명 자체는 유효
- 예시 코드가 실제 시그니처/액션명과 어긋남

### 4.3 대형 화면 심층분석 수치 오차

실측값:
- `MD_ORD01001P.xml`: 2.34MB / 55,211 lines
- `HP_DMS01303M.xml`: 2.20MB / 29,640 lines
- `HP_DMS02204M.xml`: 1.39MB / 13,537 lines

기존 문서 기재값:
- `MD_ORD01001P`: 2.4MB / 57,025 lines
- `HP_DMS01303M`: 2.3MB / 29,711 lines
- `HP_DMS02204M`: 1.4MB / 13,570 lines

판단:
- 크기 순위와 대형 화면 성격 자체는 대체로 유효
- 라인 수/크기 수치는 최신 실측으로 교체함

### 4.4 성능 문서의 Connection Pool 예시값

`rep.NPH-성능최적화-종합보고서.md`에는 `30/10` 예시가 남아 있었다.

실제 확인된 설정:
- `Servers/Tomcat v9.0 Server at localhost-config/context.xml`: `maxTotal="20"`
- `NPH_HIS/devonhome/conf/ruleEngine/jndi.xml`: `maxActive="10"`
- `NPH_HIS/devonhome/conf/product/devon-framework.xml`: `jdbc-pool` 정의 존재

판단:
- `30/10`은 확정값으로 보기 어려움
- 문서의 예시값을 실제 확인값 기준으로 낮춤

---

## 5. 신뢰도 높은 문서

### 5.1 EDViewer 문서군

가장 신뢰도 높은 문서:
- `rep.EMR_viewer-EDViewer-팩트체크.md`

이유:
- 실제 확인된 JSP
- 실제 확인된 ECS 연동 클래스
- 실제 확인된 배포 바이너리
- 미확인 항목을 미확인으로 명시

### 5.2 MiplatformConverter 문서

- `03.analysis_results/031.Architecture - Framework/03.rep.DevOn_IO Layer-.MiplatformConverter.java.md`

이유:
- 실제 `MiplatformConverter.java` 실파일 기반
- 주요 메서드 실존 확인
- 실행 흐름의 DB 접근 계층은 일반화해서 보는 것이 안전

---

## 6. 문서별 상태 요약

| 문서 | 상태 | 비고 |
|------|------|------|
| `rep.EMR_viewer-EDViewer-팩트체크.md` | 높음 | 사실 기반 요약본 |
| `rep.EMR_viewer-EDViewer-overview.md` | 중상 | 팩트체크 참조 전제 시 유효 |
| `rep.EMR_viewer-EDViewer-바이너리-분석.md` | 중상 | 예시 코드와 실구현 구분 필요 |
| `tech-stack.md` | 높음 | 주요 jar/코드 기준 재검증 반영 |
| `rep.NPH-성능분석-및-최적화방안.md` | 중상 | 프로토콜/캐시 표현 보정됨 |
| `rep.NPH-성능최적화-종합보고서.md` | 중 | 일부 수치 예시는 여전히 가정값 포함 |
| `patient-journey-simulation.md` | 중하 | 개념 문서로만 사용 권장 |
| `ref.MiPlatform-전체구조-트리.md` | 중하 | 대표 예시 비중 큼 |
| `README.md` | 중하 | 입문 문서지만 XML Query 단정 수정 필요 |
| `detailed-analysis.md` | 중하 | 흐름 설명은 유효, 예시 시그니처/액션 오차 존재 |
| `03.analysis_results/031.Architecture - Framework/*` | 중 | 해설 문서로는 유효, 실구현 문서로는 주의 |
| 대형화면 심층분석 3종 | 중상 | 방향성 유효, 실측 수치 갱신 반영 |
| `97.Reference - 읽어볼거리/*` | 참고용 | 코드베이스 1:1 문서 아님 |

---

## 7. 다음 우선순위 권고

1. `README.md`와 `detailed-analysis.md`에 `LCommonDao` 중심 XML Query 설명을 더 직접적인 실코드 예시로 보강
2. `03.analysis_results`의 나머지 하위 폴더도 동일 기준으로 전수 보정
3. `patient-journey-simulation.md`의 대표 예시 블록을 실제 검증된 transaction/command 사례로 일부 치환

---

## 8. 실측 근거

- `COMMON/src/devonx/nph/miplatform/MiplatformConverter.java`
- `COMMON/src/devonx/nph/miplatform/MiplatformRequest.java`
- `COMMON/src/devonx/nph/miplatform/MiplatformResponse.java`
- `COMMON/src/devonx/nph/system/servlet/MiplatformServlet.java`
- `COMMON/src/devonx/nph/system/servlet/GeneralServlet.java`
- `COMMON/src/devonx/nph/system/cmd/AbstractMiplatformCommand.java`
- `COMMON/src/devonx/nph/util/TxServiceUtil.java`
- `NPH_HIS/devonhome/conf/product/devon-framework.xml`
- `NPH_HIS/devonhome/conf/requisite.xml`
- `NPH_HIS/webapp/ui/MD/ORD/MD_ORD01001P.xml`
- `NPH_HIS/webapp/ui/HP/DMS/HP_DMS01303M.xml`
- `NPH_HIS/webapp/ui/HP/DMS/HP_DMS02204M.xml`
- `NPH_HIS/src/nph/his/core/interceptor/LoginCheckInterceptor.java`
- `NPH_HIS/src/nph/his/core/interceptor/FileUploadInterceptor.java`
- `NPH_HIS/src/nph/his/core/interceptor/UrlPrivCheckInterceptor.java`

---

*본 보고서는 기존 문서를 최대한 보존하고, 전수 점검 결과만 별도 산출물로 추가한 것이다.*
