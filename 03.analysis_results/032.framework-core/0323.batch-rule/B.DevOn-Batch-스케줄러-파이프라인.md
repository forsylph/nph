# DevOn Batch 스케줄러 파이프라인

## 1. 목적

이 문서는 `devon-batch-scheduler.xml`을 기준으로 NPH 배치 스케줄러가 어떤 실행 위치, Reader, Parser, 파라미터 처리 구성을 갖는지 정리한다.

약어/용어는 [../../030.index/0303.약어-용어집/약어-용어집.md](../../030.index/0303.%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91/%EC%95%BD%EC%96%B4-%EC%9A%A9%EC%96%B4%EC%A7%91.md)를 먼저 보면 빠르다.

## 2. 핵심 결론

- `devon-batch-scheduler.xml`은 배치 스케줄러의 실행 위치와 파라미터 처리 파이프라인을 정의한다.
- 현재 확인된 기본 실행 위치는 `embedded` executor 하나다.
- Reader/Parser 구성이 꽤 풍부해서, 파일·리소스·JDBC·JSON·CSV·고정길이 등 다양한 입력 형식을 수용한다.
- 이 문서의 중심은 `Rule`이 아니라 `배치가 어떤 형태의 입력을 어떤 파이프라인으로 처리하느냐`다.

## 3. 스케줄러 설정의 중심축

### 3.1 실행 위치

확인된 설정:
- `scheduler-enable=false`
- `executor/location-list/embedded/id=SCHEDULER-EMBEDDED`
- `enabled=true`

해석:
- 스케줄러 UI/기능은 전역적으로 꺼져 있을 수 있지만,
- executor 목록에는 embedded 실행자가 남아 있다.
- 즉 현재 백업 기준으로는 내장형 실행 모델이 기본이다.

### 3.2 스케줄러 DB spec

확인된 설정:
- `database/devon-spec-name=default`
- timestamp / schedule-date-format 정의

해석:
- 배치 스케줄러도 DevOn spec 이름 `default`를 본다.
- 별도 Rule 엔진 DB와 동일한 이야기가 아니다.

## 4. 파라미터 파이프라인

```mermaid
flowchart LR
    input --> reader
    reader --> parser
    parser --> job
```

### 4.1 Reader 계층

실제 설정에서 보이는 대표 Reader:
- `DirectStringReader`
- `FileParameterReader`
- `ResourceFileParameterReader`
- `ResourceDirFileParameterReader`
- `JdbcCursorReader`
- `RelativeFileParameterReader`
- `EmptyReader`

해석:
- 배치 입력은 단일 형태가 아니다.
- 파일, 리소스, DB 커서, 직접 입력까지 흡수한다.
- 특히 `JdbcCursorReader`가 들어 있다는 점은 배치가 DB를 직접 소스로 삼는 흐름을 공식적으로 갖고 있다는 의미다.

### 4.2 Parser 계층

실제 설정에서 보이는 대표 Parser:
- `JSON2LDataParser`
- `CSV2LDataParser`
- `LData2CSVParser`
- `FixedLength2LDataParser`
- `LData2FixedLengthParser`

해석:
- DevOn Batch는 단순 배치 스케줄러가 아니라, 데이터 형태 변환 파이프라인까지 프레임워크 차원에서 제공한다.
- 따라서 `batch-rule`을 단순히 Rule 엔진 문서로만 채우면, 이 중요한 파이프라인 특성이 가려진다.

## 5. 이 문서와 InnoRules 문서의 경계

### 5.1 여기서 설명하는 것

- 배치 스케줄러 설정
- 입력 소스
- 파싱 변환
- embedded executor
- batch spec / batch logging

### 5.2 여기서 설명하지 않는 것

- InnoRules 저장소 `rule`
- IRClient / IRSession API 세부
- Rule 코드/Rule 정의 방식
- Rule 엔진 초기화 순서 자체

위 내용은 [../../033.platform-services/0334.InnoRules](../../033.platform-services/0334.InnoRules)에서 다룬다.

## 6. 해석

`0323.batch-rule`은 이름 때문에 Rule 엔진 문서로 읽히기 쉽지만, 실제로는 `배치 실행 틀`을 설명하는 자리가 더 맞다.

NPH 배치를 유지보수할 때 먼저 알아야 하는 것은:
- 어느 executor가 실행되는가
- 입력은 어디서 오나
- Reader/Parser가 어떻게 붙나
- DB 커서를 쓰는가
- 로그와 상태는 어디에 남나

이지, Rule 엔진 상세가 아니다.

## 7. 다음에 읽을 문서

- [C.Rule-Engine-연동지점.md](./C.Rule-Engine-%EC%97%B0%EB%8F%99%EC%A7%80%EC%A0%90.md)
- [../../033.platform-services/0334.InnoRules/InnoRules-Batch-연동.md](../../033.platform-services/0334.InnoRules/InnoRules-Batch-%EC%97%B0%EB%8F%99.md)



