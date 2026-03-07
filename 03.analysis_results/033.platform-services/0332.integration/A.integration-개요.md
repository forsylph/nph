# integration 개요

## 1. 목적

이 문서는 외부 시스템과의 공통 연동에 사용되는 솔루션과 패키지를 정리하는 기준본이다.

## 2. 현재 확인된 주요 대상

- `Apache Axis`
- `Jersey/JAX-RS` 중 DevOn 외부 경계 서비스 성격의 문서
- `Apache HttpClient`
- `edtFTPj`
- `JSch`
- `Commons Net`
- `ldapjdk`
- `JCAOS`
- `SVNKit`

## 3. 분류 기준

- 외부 시스템 연계, 파일 전송, HTTP/SOAP/REST 클라이언트, 인증 연동 패키지는 여기서 관리한다.
- 의료 특화 연계의 업무 맥락은 `035.Biz-medical-Domain`과 같이 본다.

## 4. 다음 단계

- 실제 연동 사례를 기술별로 분리
- 공통 연동과 의료업무 연동의 경계 정리

