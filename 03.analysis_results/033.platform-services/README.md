# README

## 1. 목적

이 폴더는 DevOn 자체가 아닌 외부 솔루션, 패키지, 플랫폼 서비스를 정리하는 기준 폴더다.

운영 기준:
- DevOn 내부 구조면 `032.framework-core`
- DevOn 외부 솔루션/패키지면 `033.platform-services`

## 2. 하위 구획

- `0331.security-auth`
  - SSO, SAML, 전자서명, XSS 방어 등
- `0332.integration`
  - SOAP/REST 연동, HTTP/FTP/SFTP, 외부 시스템 연계
- `0333.shared-solutions`
  - 공통 솔루션, 외부 라이브러리, 플랫폼 공통 서비스
- `0334.InnoRules`
  - 외부 의료 Rule 엔진 솔루션과 Java/Batch 연동 문서

## 3. 참고

- 의료 특화 솔루션의 기준본은 `035.Biz-medical-Domain`에 둔다.
- 예: `EDViewer`는 여기서 언급될 수 있지만 기준 분석 문서는 `035`에 둔다.




## 4. 같이 볼 문서

- 화면 진입은 [../031.front-channel/0313.ui-entry/A.Front-Channel-개요.md](../031.front-channel/0313.ui-entry/A.Front-Channel-개요.md)
- DevOn 코어는 [../032.framework-core/0321.overview/A.Framework-개요.md](../032.framework-core/0321.overview/A.Framework-개요.md)
- Data Access는 [../032.framework-core/0322.data-access/A.Data-Access-개요.md](../032.framework-core/0322.data-access/A.Data-Access-개요.md)
- 의료업무 맥락은 [../035.Biz-medical-Domain](../035.Biz-medical-Domain)
- 실제 사례는 [../037.runtime-trace/A.트레이스-읽는순서.md](../037.runtime-trace/A.트레이스-읽는순서.md)
- 사실/미확인은 [../038.fact-todo-reference/0382.fact-check/00.fact-check.md](../038.fact-todo-reference/0382.fact-check/00.fact-check.md)
