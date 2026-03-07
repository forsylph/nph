# Solutions 개요

> 최종 수정: 2026-03-07

---

## 1A. 상위 연결

- 이 폴더의 기준 설명은 [../README.md](../README.md) 를 먼저 본다.
- DevOn 코어는 [../../032.framework-core/0321.overview/A.Framework-개요.md](../../032.framework-core/0321.overview/A.Framework-개요.md) 와 같이 본다.
- 의료업무 맥락은 [../../035.Biz-medical-Domain](../../035.Biz-medical-Domain) 으로 이어진다.
- 실제 사례는 [../../037.runtime-trace/A.트레이스-읽는순서.md](../../037.runtime-trace/A.트레이스-읽는순서.md) 를 본다.

이 문서는 DevOn 바깥에서 동작하는 공통 솔루션과 플랫폼 공통 패키지를 정리하는 기준본이다.

---

## 2. 분석 현황

| 솔루션 | 상태 | 문서 |
|--------|------|------|
| **Rexpert** | ✅ 분석 완료 | [B.Rexpert-리포트엔진.md](./B.Rexpert-리포트엔진.md) |
| **Quartz** | ✅ 분석 완료 | [C.Quartz-스케줄러.md](./C.Quartz-스케줄러.md) |
| **TPR Report** | ✅ 분석 완료 | [D.TPR-Report.md](./D.TPR-Report.md) |
| **Excel 라이브러리** | ✅ 분석 완료 | [E.Excel-라이브러리.md](./E.Excel-라이브러리.md) |
| **iText XML Worker** | ✅ 분석 완료 | [F.iText-PDF.md](./F.iText-PDF.md) |

---

## 3. 기술 스택 요약

### 3.1 리포트 엔진

| 기술 | 버전 | 공급사 | 비고 |
|------|------|--------|------|
| **Rexpert** | 3.0 | (주)렉스퍼트 | 한국형 리포트 엔진 |
| **Rexpert Viewer** | 1.0.0.57 | (주)렉스퍼트 | ActiveX/Plugin |
| **TPR Report** | - | 자체 | EMR 뷰어 리포트 |

### 3.2 리포트 파일 형식

| 형식 | 설명 |
|------|------|
| `.reb` | Rexpert 리포트 템플릿 (REX3 바이너리) |
| `.oof` | OOF (Object Oriented Format) 데이터 |

### 3.3 스케줄러

| 기술 | 버전 | 비고 |
|------|------|------|
| **Quartz** | 1.6.1 | JAR 포함, 직접 사용 안함 |
| **DEVON Batch Scheduler** | - | 실제 사용 구현체 |

### 3.4 Excel 처리

| 기술 | 버전 | 비고 |
|------|------|------|
| **Apache POI** | 3.2 (2008) | 엑셀 읽기/쓰기 |
| **JExcelApi** | - | 엑셀 읽기/쓰기 (경량) |

### 3.5 PDF 처리

| 기술 | 버전 | 비고 |
|------|------|------|
| **iText XML Worker** | 1.2.0 | HTML to PDF 변환 |

---

## 4. 아키텍처 위치

```mermaid
flowchart TB
    subgraph Client["MiPlatform Client"]
        XML[XML 화면]
        JS[JavaScript 라이브러리]
        ActiveX[RexCtl ActiveX]
    end

    subgraph Server["Server Side"]
        JSP[JSP Services]
        RexJar[Rexpert.jar]
        Templates[리포트 템플릿]
        Batch[Batch Scheduler]
    end

    subgraph Shared["0333.Solutions"]
        Rexpert[Rexpert 3.0]
        Quartz[Quartz 1.6.1]
    end

    XML --> JS
    JS --> ActiveX
    ActiveX --> JSP
    JSP --> RexJar
    RexJar --> Templates
    Templates --> Rexpert
    Batch --> Quartz
```

---

## 5. 솔루션 사용 현황

### 5.1 Rexpert

**사용 패턴:**
```
화면 이벤트 → JavaScript 호출 → Dataset 변환 → OOF 생성 → ActiveX 렌더링
```

**주요 함수:**

| 함수 | 설명 |
|------|------|
| `cf_ViewReport()` | 리포트 미리보기 팝업 |
| `cf_printReport()` | 바로출력 |
| `cf_PreviewReport()` | 화면 내장 뷰어 |
| `cf_DataSettoXML()` | Dataset → XML 변환 |

**사용 화면:**
- MD/ORD - 처방전, 처방 내역
- MD/HEA - 건강검진 리포트
- MR/RCH - 접수증 출력
- ER/CSR - 응급 환자 리포트
- SP/LAB - 검사 결과 리포트
- HP/PAT - 환자 라벨 출력

### 5.2 DEVON Batch Scheduler

**사용 패턴:**
```
DB 설정 → JobIF 구현체 → 스케줄러 실행 → 로그 기록
```

**주요 Job 클래스:**

| 클래스 | 용도 |
|--------|------|
| `SmsAutoSendJob` | SMS 자동 발송 |
| `EisMartDailyJob` | EIS 마트 일배치 |
| `NedisAutoJob` | NEDIS 자동 처리 |

**DB 테이블:**
- `olb_job_group_info` - Job 그룹 정보
- `olb_job_info` - Job 상세 정보
- `olb_job_exec_tx` - 실행 이력

---

## 6. 파일 구조

```
0333.Solutions/
├── README.md
├── A.Solutions-개요.md              # 이 문서
├── B.Rexpert-리포트엔진.md          # ✅ 분석 완료
├── C.Quartz-스케줄러.md             # ✅ 분석 완료
├── D.TPR-Report.md                  # ✅ 분석 완료
├── E.Excel-라이브러리.md            # ✅ 분석 완료
└── F.iText-PDF.md                   # ✅ 분석 완료
```

---

## 7. 분류 기준

- DevOn 자체 실행 구조가 아닌 외부 솔루션/패키지 설명은 여기서 관리한다.
- 단, 배치/룰의 DevOn 실행 구조 설명은 `032.framework-core`에 둔다.
- 의료 특화 솔루션의 기준본은 `035.Biz-medical-Domain`에 둔다.

---

## 8. 분석 완료

모든 주요 솔루션 분석이 완료되었습니다.

1. ~~Rexpert 리포트 엔진~~ ✅ 완료
2. ~~Quartz 스케줄러~~ ✅ 완료
3. ~~TPR Report~~ ✅ 완료
4. ~~Excel 라이브러리 (POI/JXL)~~ ✅ 완료
5. ~~iText XML Worker~~ ✅ 완료

---

## 9. 관련 문서

- [B.Rexpert-리포트엔진.md](./B.Rexpert-리포트엔진.md) - Rexpert 상세 분석
- [C.Quartz-스케줄러.md](./C.Quartz-스케줄러.md) - Quartz/스케줄러 상세 분석
- [D.TPR-Report.md](./D.TPR-Report.md) - TPR Report 상세 분석
- [E.Excel-라이브러리.md](./E.Excel-라이브러리.md) - Apache POI/JExcelApi 분석
- [F.iText-PDF.md](./F.iText-PDF.md) - iText XML Worker 분석
- [Tech-Stack-개요.md](../../030.index/0307.Tech%20Stack/Tech-Stack-개요.md)