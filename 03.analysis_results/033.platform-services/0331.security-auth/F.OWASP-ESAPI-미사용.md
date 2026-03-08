# OWASP ESAPI 분석

> 분석일: 2026-03-07
> 분석 대상: `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace`

---

## 1. 개요

NPH 시스템의 `WEB-INF/lib`에 `esapi-2.0.1.jar`이 존재하지만, 실제 사용 여부를 확인한 결과 **미사용 상태**로 판명되었다.

### 1.1 JAR 파일 정보

| 항목 | 내용 |
|------|------|
| **파일명** | esapi-2.0.1.jar |
| **위치** | NPH_HIS/webapp/WEB-INF/lib/ |
| **크기** | 367,201 bytes |
| **버전** | 2.0.1 |

---

## 2. 분석 방법

### 2.1 검색 항목

| 검색 대상 | 검색 결과 |
|-----------|-----------|
| `import org.owasp.esapi.*` | 없음 |
| `ESAPI.getLogger()` | 없음 |
| `ESAPI.encoder()` | 없음 |
| `ESAPI.validator()` | 없음 |
| ESAPI 설정 파일 (`ESAPI*.properties`) | 없음 |
| web.xml ESAPI 필터 설정 | 없음 |

### 2.2 검색 경로

- NPH_HIS/src/*.java
- NPH_ECS/src/*.java
- NPH_HIS/webapp/**/*.jsp
- NPH_HIS/webapp/WEB-INF/web.xml
- 빌드 설정 파일 (*.xml, *.properties)

---

## 3. 분석 결과

### 3.1 코드 사용 여부

```
❌ Java 소스 코드에서 ESAPI import 없음
❌ JSP에서 ESAPI 사용 없음
❌ web.xml에 ESAPI 필터 설정 없음
❌ ESAPI 설정 파일 없음
```

### 3.2 XSS 방어 대안

NPH는 ESAPI 대신 **Lucy XSS Filter**를 사용하여 XSS 방어:

```java
// Lucy XSS Filter 사용 예시 (NPH 표준 패턴)
import com.nhncorp.lucy.security.xss.XssFilter;

XssFilter lucyFilter = XssFilter.getInstance();
String safeString = lucyFilter.doFilter(inputString);
```

### 3.3 입력 검증 대안

- **MiPlatform Dataset**: 클라이언트 측 입력 검증
- **DEVON Framework**: 서버 측 `LData` 기반 검증
- **커스텀 Validator**: `UploadFileValidator` 등 프로젝트별 구현

---

## 4. OWASP ESAPI란?

### 4.1 ESAPI (Enterprise Security API)

OWASP ESAPI는 보안 기능을 제공하는 라이브러리:

| 기능 | ESAPI API | 설명 |
|------|-----------|------|
| XSS 방어 | `ESAPI.encoder()` | HTML/URL/JavaScript 인코딩 |
| 입력 검증 | `ESAPI.validator()` | 화이트리스트 기반 검증 |
| 보안 로깅 | `ESAPI.getLogger()` | 보안 이벤트 로깅 |
| 암호화 | `ESAPI.encryptor()` | 암호화/복호화 |
| 인증 | `ESAPI.authenticator()` | 인증 인터페이스 |

### 4.2 ESAPI 2.0.1 (2013년)

- 2013년 릴리스
- 현재 최신 버전: 2.5.x (2024년)
- 보안 취약점이 존재하는 레거시 버전

---

## 5. 현재 판단

현재 코드/JSP/web.xml/config 검색 기준으로 강하게 말할 수 있는 건 아래까지다.

1. `esapi-2.0.1.jar`는 배포물에 존재한다.
2. 애플리케이션 코드의 직접 import/use는 확인되지 않았다.
3. ESAPI 설정 파일도 찾지 못했다.
4. NPH의 실제 XSS 방어는 Lucy 쪽 근거가 더 강하다.

즉 현재 단계에서는 `미사용 상태` 판정은 타당하지만, 왜 배포물에 포함됐는지까지는 아직 닫히지 않았다.

---

## 6. 권장 사항

### 6.1 즉시 조치

| 항목 | 권장 |
|------|------|
| **ESAPI JAR 제거** | 사용되지 않으므로 제거 검토 |
| **의존성 확인** | 다른 라이브러리의 전이 의존성 여부 확인 |

### 6.2 제거 전 확인 사항

1. 다른 JAR의 전이 의존성 여부 확인
2. 실제 서버 배포 스크립트/클래스로더 참조 여부 확인
3. 제거 후 전체 기능 테스트

### 6.3 보안 개선

| 항목 | 현재 상태 | 권장 |
|------|-----------|------|
| XSS 방어 | Lucy XSS Filter | 유지 (입력/출력 모두 적용 권장) |
| 입력 검증 | 커스텀 + Framework | 화이트리스트 기반 검증 강화 |
| 보안 로깅 | 일반 로깅 | 보안 이벤트 별도 로깅 권장 |

---

## 7. 사용되지 않는 라이브러리 관리

```
미사용 라이브러리 문제:
- 보안 취약점 노출
- 불필요한 디스크 공간 차지
- 클래스 로딩 오버헤드
- 유지보수 혼란
```

---

## 8. 요약

| 항목 | 상태 |
|------|------|
| **ESAPI JAR 존재** | ✅ 있음 (esapi-2.0.1.jar) |
| **소스 코드 사용** | ❌ 없음 |
| **설정 파일** | ❌ 없음 |
| **실제 사용** | ❌ 미사용 |
| **대체 솔루션** | Lucy XSS Filter, SignGate |
| **권장** | JAR 제거 검토 |

---

## 9. 연결 문서

- [README.md](./README.md)
- [E.Lucy-XSS-Filter.md](./E.Lucy-XSS-Filter.md)
- [Tech-Stack-개요.md](../../030.index/0307.Tech%20Stack/Tech-Stack-개요.md)

---

*분석 완료: 2026-03-07*



