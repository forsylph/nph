# security-auth 개요

## 1. 목적

이 문서는 `033.platform-services` 중 보안/인증 계열 솔루션을 정리하는 기준본이다.

## 2. 현재 확인된 주요 솔루션

- `MagicSSO`
- `MagicSAML`
- `OpenSAML`
- `GPKI`
- `SignGate`
- `Lucy XSS Filter`
- `OWASP ESAPI` (JAR 존재, 실제 사용은 별도 확인 필요)
- `Bouncy Castle`

## 3. 분류 기준

- 인증/인가, SSO, 인증서, SAML, 전자서명, XSS 방어는 이 하위 폴더에서 관리한다.
- DevOn 내부 interceptor나 command 구조 설명은 `032.framework-core`에 둔다.

## 4. 다음 단계

- 기술별 실제 코드 사용 위치 정리
- 인증 흐름과 사용자 로그인 흐름 연결
- `035.Biz-medical-Domain`의 의료업무 인증 시나리오와 링크
