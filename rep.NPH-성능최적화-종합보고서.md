# NPH 시스템 성능 최적화 종합 보고서

> NPH 병원정보시스템 성능 병목 분석 및 최적화 방안
> 분석일: 2026-03-06
> 분석 범위: MiPlatform 통신 / XML Query / 대형 화면 / EDViewer / DB 성능

---

## Executive Summary

### 핵심 발견

| 영역 | 현재 상태 | 주요 병목 | 개선 긴급도 |
|------|-----------|-----------|-------------|
| **MiPlatform 통신** | BIN 프로토콜 사용 ✅ | 이미 최적화됨 | 낮음 |
| **XML Query** | 2,023개 파일, 런타임 파싱 | 캐싱 없음, `${}` 문자열 치환 | **높음** |
| **대형 화면** | 2.4MB 화면 존재 | Dataset 163개, 컬럼 5,859개 | **높음** |
| **EDViewer** | 7.7MB OCX 바이너리 | ActiveX 초기화, 소스 없음 | **높음** |
| **DB 쿼리** | Connection Pool 30개 | 복잡 조인, 페이징 없음 | **중간** |

### 권고사항 요약

1. **즉시 적용 (P0)**: XML Query 파일 캐싱, 대형 화면 지연 로딩
2. **단기 적용 (P1)**: Connection Pool 증설, EDViewer 래퍼 개선
3. **중기 적용 (P2)**: 인덱스 추가, 스칼라 서브쿼리 제거

---

## 1. MiPlatform 통신 프로토콜 분석

### 1.1 BIN(바이너리) 프로토콜 확인

**소스 코드 증거**:

```java
// MiplatformRequest.java:67-69
// 성능테스트를 위해 잠시변경함
// this.method = PlatformRequest.XML;
this.method = PlatformRequest.BIN;  // BIN 모드 강제 적용

// MiplatformResponse.java:62-65
// 성능테스트를 위해 잠시변경함
// this(httpResponse, PlatformResponse.XML, ...);
this(httpResponse, PlatformResponse.BIN, MiplatformConverter.DEFAULT_CHARSET);
```

### 1.2 프로토콜 비교

| 프로토콜 | 특징 | 크기 | 속도 | NPH 사용 |
|----------|------|------|------|----------|
| **BIN** | 바이너리 직렬화 | 원본의 100% | 매우 빠름 | ✅ **사용 중** |
| XML | 텍스트 마크업 | 원본의 300-500% | 느림 | ❌ 주석 처리 |
| ZIP | BIN + 압축 | 원본의 30-50% | 중간 | ❌ 미사용 |

### 1.3 결론: 통신 최적화 완료 ✅

MiPlatform BIN 프로토콜은 이미 최적화된 상태입니다. 추가 개선 여지는 낮으며, **주요 병목은 통신이 아닌 다른 영역**에 있습니다.

---

## 2. DevOn XML Query 성능 분석

### 2.1 현황

- **XML Query 파일 수**: 2,023개
- **총 크기**: 약 50MB
- **실행 방식**: 런타임 파일 로드 + 파싱

### 2.2 주요 병목

#### 병목 1: XML 런타임 파싱 (Critical)

```java
// EC 클래스에서의 사용 예시
LCommonDao commDao = new LCommonDao("/sp/cel/spschrprt/retrievePtRcpnInfoRtrv", data);
LMultiData lResult = commDao.executeQuery();  // 매번 XML 파일 로드/파싱
```

**문제점**:
- XML Query가 매 요청마다 파일 시스템에서 로드됨
- 2,023개 XML 파일 (약 50MB)의 반복 파싱
- CPU 집약적 작업

#### 병목 2: PreparedStatement 미사용 (Critical)

```xml
<!-- 기존 방식: 문자열 치환 -->
WHERE PRSC_SET_CLSF_SQNO = ${prscSetClsfSqno}

<!-- 개선 방식: 파라미터 바인딩 -->
WHERE PRSC_SET_CLSF_SQNO = ?
```

**SQL Injection 취약점 존재 가능성**

#### 병목 3: 캐싱 메커니즘 부재 (High)

- XML Query 파일 캐싱: ❌ 없음
- 파싱된 SQL 캐싱: ❌ 없음
- 쿼리 결과 캐싱: ❌ 없음

### 2.3 개선 방안

#### 단기: XML Query 파일 캐싱 구현

```java
// 싱글톤 캐싱 클래스
public class XmlQueryCache {
    private static final Map<String, String> queryCache = new ConcurrentHashMap<>();

    public static String getQuery(String xmlPath, String queryName) {
        String key = xmlPath + "#" + queryName;
        return queryCache.computeIfAbsent(key, k -> loadAndParseQuery(xmlPath, queryName));
    }

    // 서버 시작 시 프리로딩
    public static void preloadAllQueries() {
        // xmlquery 디렉토리 순회하여 모든 쿼리 로드
    }
}
```

#### 중기: PreparedStatement 적용

**단계적 마이그레이션**:
1. 신규 쿼리는 `?` 파라미터 바인딩 방식 적용
2. 기존 `${}` 쿼리는 SQL Injection 필터링 강화
3. 사용 빈도 높은 쿼리 우선순위로 마이그레이션

---

## 3. 대형 화면 로딩 성능 분석

### 3.1 대형 화면 현황

| 순위 | 화면ID | 크기 | Dataset 수 | 컬럼 수 | 위험도 |
|------|--------|------|------------|---------|--------|
| 1 | **MD_ORD01001P** | **2.4MB** | **163개** | **5,859개** | 🔴 심각 |
| 2 | HP_DMS01303M | 2.3MB | 4개 | 125개 | 🟠 높음 |
| 3 | HP_DMS02204M | 1.4MB | 15개 | 90개 | 🟡 중간 |
| 4 | MD_HEA02420M | 983KB | 76개 | 2,301개 | 🟠 높음 |

### 3.2 MD_ORD01001P (처방등록) 심층 분석

**성능 병목 요소**:
- Dataset: 163개 (과도)
- 컬럼 정의: 5,859개
- JavaScript 함수: 1,034개
- 서버 통신: 142개
- Timer: 2개 (1초, 3초 간격)

**화면 구조**:
```xml
<Form Id="MD_ORD01001P" ...>
    <Datasets>
        <!-- 163개 Dataset 정의 -->
        <Dataset Id="ds_Patient" ...>...</Dataset>
        <Dataset Id="ds_Prescription" ...>...</Dataset>
        <!-- ... 161개 더 ... -->
    </Datasets>

    <!-- UI 컴포넌트 -->
    <Grid Id="Grid_Prescription" .../>       <!-- 26개 그리드 -->
    <Edit Id="ED_PatientName" .../>          <!-- 69개 입력 -->
    <Button Id="btn_Save" .../>             <!-- 118개 버튼 -->
    <Tab Id="tab_main" ...>                 <!-- 5개 TabPage -->
    <Div ...>                              <!-- 29개 Div -->
</Form>
```

### 3.3 개선 방안

#### 방안 1: Tab 기반 지연 로딩 (P0)

```javascript
// 현재: 모든 Tab 초기화
function MD_ORD01001P_OnLoadCompleted(obj) {
    initTab1(); initTab2(); initTab3(); initTab4(); initTab5();  // 전부 로드
}

// 개선: 활성 Tab만 초기화
function tab_main_OnTabChanged(obj, nIndex) {
    if (!isTabLoaded[nIndex]) {
        loadTabData(nIndex);  // 지연 로드
        isTabLoaded[nIndex] = true;
    }
}
```

#### 방안 2: Dataset 동적 생성 (P1)

```javascript
// 현재: XML에 163개 Dataset 정의
// 개선: 필수 Dataset만 XML에 정의, 나머지는 동적 생성
function createDynamicDataset(dsId, colInfos) {
    var dsObj = new Dataset();
    dsObj.id = dsId;
    for (var i = 0; i < colInfos.length; i++) {
        dsObj.addColumn(colInfos[i].id, colInfos[i].type, colInfos[i].size);
    }
    return dsObj;
}
```

**예상 효과**: MD_ORD01001P 화면 로딩 시간 **60-70% 단축**

---

## 4. EDViewer(EMR 뷰어) 성능 분석

### 4.1 현황

| 항목 | 내용 |
|------|------|
| **제품** | EDViewer / eView |
| **타입** | ActiveX / OCX (C++ 바이너리) |
| **크기** | 7.7MB (EDViewer_Ocx.ocx) |
| **플랫폼** | Windows + IE only |
| **소스코드** | **없음** (바이너리만 존재) |

### 4.2 주요 병목

**1. OCX 초기화 시간 (3-10초)**
```
사용자 클릭 → EDViewer 팝업 → OCX 로드 → 보안 확인 → 화면 표시
```

**2. EMR 데이터 로딩**
```java
// eddocument_eView.jsp 주석
// 기록지 파일을 불러오는 것은 현재의 timer로는 문제가 있다.
// DB에서 데이터 받아오는 것도 현재의 timer로도 문제가 있다. → 방안 고려
```

**3. ActiveX 의존성**
- IE 11만 지원
- Edge, Chrome, Firefox 미지원
- 기업 보안 정책과 충돌

### 4.3 개선 방안

#### 단기: JavaScript 래퍼 개선 (P1)

```javascript
// eViewCommon.js 개선
// 1. 비동기 AJAX로 변경
// 2. 데이터 캐싱 적용
// 3. 로딩 인디케이터 추가
```

#### 중장기: HTML5 EMR 뷰어 도입 (P2)

**대안 기술**:
| 기술 | 장점 | 단점 |
|------|------|------|
| **PDF.js** | 오픈소스, 모든 브라우저 | 개발 기간 필요 |
| **Fabric.js** | 서명 캔버스 지원 | 커스텀 개발 |
| **상용 EMR** | 기능 완성도 높음 | 라이선스 비용 |

---

## 5. 데이터베이스 성능 분석

### 5.1 현황

**Connection Pool 설정** (devon-framework.xml):
```xml
<jdbc-pool name="default">
    <max-active-connections>30</max-active-connections>
    <max-idle-connections>10</max-idle-connections>
</jdbc-pool>
```

### 5.2 주요 병목

**1. 복잡한 조인 쿼리**
```sql
-- md/ord/mdmdhtord.xml
-- 13개 테이블 조인, use_nl 힌트
SELECT ...
FROM MDMDHTORD M1, MDMDHCPDI M2, MDMDHSIGN M3, ... (13개)
WHERE ...
```

**2. 스칼라 서브쿼리 남용**
```sql
-- sp/tes/spsthjrpt.xml
SELECT
    (SELECT ... FROM HPPAHIDPT WHERE CHOS_NO = E.CHOS_NO),
    (SELECT ... FROM HPPAHIDPT WHERE CHOS_NO = E.CHOS_NO),
    (SELECT ... FROM HPPAHIDPT WHERE CHOS_NO = B.CHOS_NO)
```

**3. 페이징 미적용**
- 대용량 목록 조회 시 전체 데이터 로드
- ROWNUM 페이징 일부 적용되나 미흡

### 5.3 개선 방안

#### 단기: Connection Pool 증설 (P1)

```xml
<jdbc-pool name="default">
    <max-active-connections>80</max-active-connections>  <!-- 30 → 80 -->
    <max-idle-connections>30</max-idle-connections>       <!-- 10 → 30 -->
    <min-idle-connections>15</min-idle-connections>       <!-- 추가 -->
    <validation-query>SELECT 1 FROM DUAL</validation-query>
</jdbc-pool>
```

#### 중기: 인덱스 추가 (P2)

| 테이블 | 컬럼 | 이유 |
|--------|------|------|
| MDMDHTORD | (CHOS_NO, PRSC_DY, PRSC_STAT) | 처방 조회 |
| HPPAHIDPT | (CHOS_NO, AND_PRGR_STAT) | 입원환자 조회 |
| SPSTHJRPT | (PRSC_SQNO, EXMN_DEPT) | 검사접수 |

---

## 6. 종합 최적화 로드맵

### 6.1 즉시 적용 (P0, 1-2주)

| 순위 | 항목 | 대상 | 예상 효과 |
|------|------|------|-----------|
| 1 | XML Query 파일 캐싱 | 2,023개 파일 | 쿼리 실행 20% 개선 |
| 2 | 대형 화면 지연 로딩 | MD_ORD01001P | 로딩 60-70% 단축 |
| 3 | SQL Injection 필터링 | `${}` 사용 쿼리 | 보안 강화 |

### 6.2 단기 적용 (P1, 1-2개월)

| 순위 | 항목 | 대상 | 예상 효과 |
|------|------|------|-----------|
| 4 | Connection Pool 증설 | 30 → 80 | 대기 시간 감소 |
| 5 | EDViewer 래퍼 개선 | eViewCommon.js | EMR 로딩 개선 |
| 6 | 화면 캐싱 | HTTP Cache-Control | 재방문 속도 향상 |

### 6.3 중기 적용 (P2, 3-6개월)

| 순위 | 항목 | 대상 | 예상 효과 |
|------|------|------|-----------|
| 7 | 인덱스 추가 | 6개 테이블 | 조회 30-50% 향상 |
| 8 | 스칼라 서브쿼리 제거 | spsthjrpt 등 | 조회 30% 향상 |
| 9 | 페이징 적용 | 대용량 목록 | 메모리 사용 감소 |
| 10 | HTML5 EMR 프로토타입 | EDViewer 대체 | 브라우저 호환성 |

### 6.4 장기 검토 (6개월 이후)

| 순위 | 항목 | 대상 | 예상 효과 |
|------|------|------|-----------|
| 11 | MyBatis 마이그레이션 | XML Query | 유지보수성 향상 |
| 12 | HTML5 EMR 완전 전환 | EDViewer | ActiveX 의존성 제거 |

---

## 7. 구체적인 설정 변경

### 7.1 JEUS HTTP 캐싱 설정

```xml
<!-- web.xml -->
<filter>
    <filter-name>CacheFilter</filter-name>
    <filter-class>jeus.servlet.filter.CacheFilter</filter-class>
    <init-param>
        <param-name>maxAge</param-name>
        <param-value>86400</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>CacheFilter</filter-name>
    <url-pattern>*.xml</url-pattern>
</filter-mapping>
```

### 7.2 DevOn 성능 로깅 활성화

```xml
<!-- devon-framework.xml -->
<log>
    <performance-log>
        <enabled>true</enabled>
        <slow-query-threshold>1000</slow-query-threshold>
        <slow-transaction-threshold>3000</slow-transaction-threshold>
    </performance-log>
</log>
```

---

## 8. 결론

### 핵심 요약

1. **MiPlatform BIN 프로토콜**: 이미 최적화됨 ✅
2. **XML Query**: 캐싱 부재로 개선 여지 큼 ⚠️
3. **대형 화면**: MD_ORD01001P(2.4MB)가 가장 심각한 병목 🔴
4. **EDViewer**: ActiveX 의존성으로 현대화 필수 🔴
5. **DB**: Connection Pool 및 쿼리 튜닝 필요 ⚠️

### 우선순위별 조치

| 단계 | 핵심 작업 | 예상 효과 |
|------|-----------|-----------|
| **P0** | XML 캐싱 + 화면 지연 로딩 | 즉각적 성능 개선 |
| **P1** | Connection Pool + EDViewer 래퍼 | 안정성 향상 |
| **P2** | 인덱스 + 쿼리 튜닝 | 장기적 성능 |

### 최종 권고

**즉시 실행할 작업**:
1. `MD_ORD01001P` 화면에 Tab 기반 지연 로딩 적용
2. XML Query 파일 캐싱 구현
3. Connection Pool 30 → 80으로 증설

이 3가지 작업만으로도 NPH 시스템의 **전반적인 성능이 30-50% 개선**될 것으로 예상됩니다.

---

*분석 완료: 2026-03-06*
*기준 소스: `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace`*
