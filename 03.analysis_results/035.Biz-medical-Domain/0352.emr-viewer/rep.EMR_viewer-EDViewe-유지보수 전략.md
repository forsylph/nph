# EDViewer 운영 병원 유지보수 전략

> 운영 중인 병원에서 EDViewer 소스 없이 유지보수하는 현실적인 전략

> 주의: 본 문서의 코드/URL 예시는 전략 설명용이며, 실제 운영 경로(`*.mhi`, JSP, API)는 코드베이스 기준으로 별도 확인이 필요합니다.

---

## 현재 상황

```
병원 운영 환경
├── HIS 운영 중 (중단 불가)
├── EDViewer 의존
│   └── 소스 코드 없음
├── 문서 양식 다수
│   └── 공수 부담
└── 래퍼로 버티는 중
    └── 한계 근접
```

---

## 현실적 선택지 (3단계 접근)

### Phase 1: 버티기 (당장~6개월) - 현재

```
JavaScript 래퍼 고도화
├── eViewCommon.js 개선
├── 보안 강화
├── 로깅 추가
└── 에러 핸들링 개선
```

### Phase 2: 점진적 전환 (6개월~2년)

```
신규 문서만 신규 뷰어 사용
├── 기존 문서: EDViewer 유지
├── 신규 문서: HTML5 뷰어
└── 병행 운영
```

### Phase 3: 완전 전환 (2년+)

```
기존 문서 마이그레이션
├── PDF/A 변환
├── HTML5 뷰어 통합
└── EDViewer 완전 폐기
```

---

## Phase 1 상세: 래퍼 고도화

### eViewCommon.js 개선안

```javascript
/**
 * EDViewer 래퍼 모듈
 * 기존 기능 + 신규 보안/안정성 기능
 */

var EDViewerWrapper = {

    // 설정
    config: {
        timeout: 30000,          // 30초 타임아웃
        maxRetry: 3,             // 최대 재시도
        enableLog: true,         // 로깅 활성화
        debugMode: false         // 디버그 모드
    },

    // 상태 관리
    state: {
        isLoading: false,
        retryCount: 0,
        lastError: null
    },

    /**
     * EDViewer 호출 (개선版)
     */
    open: function(params) {
        var self = this;

        // 1. 입력값 검증 강화
        if (!this.validateParams(params)) {
            this.showError('입력값 검증 실패');
            return false;
        }

        // 2. 중복 호출 방지
        if (this.state.isLoading) {
            console.warn('EDViewer 로딩 중...');
            return false;
        }

        this.state.isLoading = true;

        // 3. 로깅
        if (this.config.enableLog) {
            this.log('EDViewer 호출 시작', params);
        }

        try {
            // 4. ActiveX 객체 생성 (try-catch 강화)
            var edviewer = this.createActiveX();

            // 5. 파라미터 설정
            this.setParameters(edviewer, params);

            // 6. 문서 로드 (타임아웃 처리)
            var loadResult = this.loadWithTimeout(edviewer, params.url, this.config.timeout);

            if (loadResult) {
                this.show(edviewer);
                this.onSuccess(params);
            } else {
                throw new Error('문서 로드 타임아웃');
            }

        } catch (e) {
            this.handleError(e, params);
            return false;
        } finally {
            this.state.isLoading = false;
        }

        return true;
    },

    /**
     * 파라미터 검증
     */
    validateParams: function(params) {
        if (!params) return false;
        if (!params.pid || params.pid.trim() === '') return false;
        if (!params.chosType) return false;

        // XSS 필터링
        params.pid = this.sanitize(params.pid);

        return true;
    },

    /**
     * ActiveX 객체 생성
     */
    createActiveX: function() {
        try {
            return new ActiveXObject("EDViewer.Control");
        } catch (e) {
            throw new Error('EDViewer ActiveX 생성 실패: ' + e.message);
        }
    },

    /**
     * 타임아웃 처리된 로드
     */
    loadWithTimeout: function(edviewer, url, timeout) {
        var startTime = Date.now();
        var loaded = false;

        // 타임아웃 체크 인터벌
        var checkInterval = setInterval(function() {
            if (Date.now() - startTime > timeout) {
                clearInterval(checkInterval);
                return false;
            }
        }, 100);

        try {
            edviewer.LoadDocument(url);
            loaded = true;
        } catch (e) {
            clearInterval(checkInterval);
            throw e;
        }

        clearInterval(checkInterval);
        return loaded;
    },

    /**
     * 에러 처리
     */
    handleError: function(error, params) {
        this.state.lastError = error;
        this.log('EDViewer 에러', error);

        // 재시도 로직
        if (this.state.retryCount < this.config.maxRetry) {
            this.state.retryCount++;
            console.log('재시도 ' + this.state.retryCount + '/' + this.config.maxRetry);

            setTimeout(function() {
                EDViewerWrapper.open(params);
            }, 1000);

        } else {
            this.showError('EDViewer 실행 실패: ' + error.message);
            this.state.retryCount = 0;
        }
    },

    /**
     * 로깅
     */
    log: function(action, data) {
        if (!this.config.enableLog) return;

        var logData = {
            timestamp: new Date().toISOString(),
            action: action,
            data: data,
            user: document.one.doctorId.value || 'unknown'
        };

        // 서버로 로그 전송
        this.sendLog(logData);
    },

    /**
     * 서버에 로그 전송
     */
    sendLog: function(logData) {
        var xhr = new XMLHttpRequest();
        xhr.open('POST', '/ecs/logNavi/LogEdviewerAccess.mhi', true);
        xhr.setRequestHeader('Content-Type', 'application/json');
        xhr.send(JSON.stringify(logData));
    },

    /**
     * 에러 표시
     */
    showError: function(message) {
        alert('[EDViewer 오류]\n' + message + '\n\n관리자에게 문의하세요.');
    },

    /**
     * XSS 방지
     */
    sanitize: function(input) {
        if (!input) return '';
        return input.replace(/[<>"'&]/g, function(m) {
            return {'<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#x27;', '&': '&amp;'}[m];
        });
    }
};

// 사용
EDViewerWrapper.open({
    pid: 'P12345',
    chosType: 'OP',
    doctorId: 'D001'
});
```

---

## Phase 2 상세: 점진적 전환

### 하이브리드 방식

```
문서 생성 시점별 구분
├── 기존 문서 (2024년 이전)
│   └── EDViewer로 조회
│
└── 신규 문서 (2024년 이후)
    └── HTML5 뷰어로 조회
```

### 구현 방법

```java
// Command에서 분기
public class RetrieveEmrDataCMD extends AbstractMiplatformCommand {

    public void execute() {
        String docId = paramData.getString("docId");
        String docDate = paramData.getString("docDate");

        // 2024년 이후 문서는 신규 뷰어
        if (isNewFormat(docDate)) {
            // HTML5 뷰어용 URL 반환
            platformResponse.addDataset("ds_Result",
                getHtml5ViewerUrl(docId));
        } else {
            // EDViewer용 데이터 반환
            platformResponse.addDataset("ds_Result",
                getEdviewerData(docId));
        }
    }

    private boolean isNewFormat(String docDate) {
        return docDate.compareTo("20240101") >= 0;
    }
}
```

---

## Phase 3 상세: 마이그레이션

### PDF/A 변환 전략

```
기존 EMR 데이터
    ↓ 자동 변환
PDF/A-1b (장기 보존 형식)
    ↓
HTML5 뷰어에서 표시
```

### 변환 스크립트

```python
# 배치 변환 (서버 측)
import subprocess

def convert_emr_to_pdf(emr_id):
    """
    EDViewer를 headless로 실행하여 PDF 생성
    """
    cmd = [
        'EDViewer_Converter.exe',  # 변환용 CLI 버전
        '--input', emr_id,
        '--output', f'/converted/{emr_id}.pdf',
        '--format', 'pdf/a-1b'
    ]

    subprocess.run(cmd, timeout=30)
```

---

## 운영상 팁

### 1. 모니터링 강화

```javascript
// EDViewer 상태 모니터링
setInterval(function() {
    if (typeof edviewer !== 'undefined') {
        var status = edviewer.GetStatus();

        if (status === 'ERROR') {
            // 자동 복구 시도
            location.reload();
        }
    }
}, 5000);
```

### 2. 폴백(Fallback) 전략

```javascript
function openEmrWithFallback(docId) {
    // 1차: EDViewer 시도
    var success = tryEdviewer(docId);

    if (!success) {
        // 2차: PDF 다운로드 제공
        alert('뷰어 로드 실패. PDF로 다운로드합니다.');
        window.location = '/ecs/emrNavi/DownloadPdf.mhi?docId=' + docId;
    }
}
```

### 3. 캐싱

```java
// 서버 측 캐싱
@Cacheable("emrData")
public LMultiData retrieveEmrData(String docId) {
    // EDViewer 호출 대신 캐싱된 데이터 반환
    return emrCache.get(docId);
}
```

---

## 결론

### 현실적 로드맵

```
2024 Q1-Q2: 래퍼 고도화
├── JavaScript 개선
├── 모니터링 강화
└── 폴백 전략

2024 Q3-Q4: 하이브리드 시작
├── 신규 문서: HTML5
└── 기존 문서: EDViewer 유지

2025+: 완전 전환
├── PDF 변환
└── EDViewer 폐기
```

### 핵심 원칙

1. **운영 중단 없이** 점진적 개선
2. **신규/기존 분리**로 공수 분산
3. **래퍼 고도화**로 버티기
4. **장기적 관점**에서 완전 교체

---

*병원 운영을 해치지 않으면서 EDViewer 의존성을 줄이는 현실적인 전략입니다.*
