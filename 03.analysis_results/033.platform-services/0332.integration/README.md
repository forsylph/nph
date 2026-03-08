# Integration 개요

> 최종 수정: 2026-03-08

---

## 1A. 상위 연결

- 이 폴더의 기준 설명은 [../README.md](../README.md) 를 먼저 본다.
- DevOn 코어는 [../../032.framework-core/0321.overview/A.Framework-개요.md](../../032.framework-core/0321.overview/A.Framework-개요.md) 와 같이 본다.
- 의료업무 맥락은 [../../035.Biz-medical-Domain](../../035.Biz-medical-Domain) 으로 이어진다.
- 실제 사례는 [../../037.runtime-trace/트레이스-읽는순서.md](../../037.runtime-trace/트레이스-읽는순서.md) 를 본다.

이 문서는 외부 시스템과의 공통 연동에 사용되는 솔루션과 패키지를 정리하는 기준본이다.

---

## 1. 현재 확인된 강한 사용 근거

- HTTP/REST: `com.rest.api.*`, `com.rest.util.HttpClientUtil`, `org.apache.http.*`, `javax.ws.rs.*`, `com.sun.jersey.*`
- FTP: `nph.bat.sample.FtpUploadJob`, `FtpDownloadJob`, `nph.bat.hp.job.FtpUploadJob`, `com.enterprisedt.net.ftp.*`
- SOAP/Axis: `SavekimsCMD`, `kr.co.kimsonline.poc.MedicalInfoSoapStub`, 다수 `org.apache.axis.*` 생성/VO 코드
- SVNKit: `SvnLogPC`, `org.tmatesoft.svn.*`

추가 검증 결과 `JSch`는 `Admin/temp/uploadFile.jsp`에서 직접 사용이 확인되었고, `xldap`는 `UserMngmPC`, `EamIFUC`, `ComLoginUC`, `MenuInfoCMD` 등에서 직접 사용이 확인되었다. `JCAOS`는 애플리케이션 소스 import는 미확인이지만 `dsagent.properties`의 `crypto.type=JCAOS`, `magicsaml-sp-v1.3.3.jar`의 `JCAOSCryptoApi`, `jcaos-1.4.7.7.jar`의 `JCAOSProvider` 기준으로 런타임 구성 관여는 확인된다.

---

## 1B. 직접 확인 근거 파일

| 기술 | 직접 확인 근거 |
|------|----------------|
| HTTP/REST | `com/rest/api/*`, `com/rest/util/HttpClientUtil.java`, `PolNetUC.java`, `LstUC.java`, `web.xml`의 Jersey servlet |
| FTP | `nph/bat/sample/FtpUploadJob.java`, `FtpDownloadJob.java`, `nph/bat/hp/job/FtpUploadJob.java` |
| JSch | `webapp/Admin/temp/uploadFile.jsp` |
| xldap | `UserMngmPC.java`, `EamIFUC.java`, `ComLoginUC.java`, `MenuInfoCMD.java`, `ReturnSessionCMD.java` |
| SOAP/Axis | `SavekimsCMD.java`, `SelectkimsCMD.java`, `KIMSMngmPC.java`, `MedicalInfoSoapStub.java`, `log4j.properties` |
| JCAOS | `webapp/WEB-INF/homepath/cfg/dsagent.properties`, `magicsaml-sp-v1.3.3.jar`, `jcaos-1.4.7.7.jar` |
| SVNKit | `SvnLogPC.java`, `SvnLogIFPC.java`, `instance_az.properties` |

---

## 2. 하위 문서

| 솔루션 | 상태 | 문서 |
|--------|------|------|
| **HTTP/REST 클라이언트** | ✅ 분석 완료 | [B.HTTP-REST-클라이언트.md](./B.HTTP-REST-클라이언트.md) |
| **FTP/SSH 클라이언트** | ✅ 분석 완료 | [C.FTP-SSH-클라이언트.md](./C.FTP-SSH-클라이언트.md) |
| **LDAP/인증 연동** | ✅ 분석 완료 | [D.LDAP-인증연동.md](./D.LDAP-인증연동.md) |
| **SOAP 웹서비스** | ✅ 분석 완료 | [E.SOAP-웹서비스.md](./E.SOAP-웹서비스.md) |
| **JCAOS/SVNKit** | ✅ 분석 완료 | [F.JCAOS-SVNKit.md](./F.JCAOS-SVNKit.md) |

---

## 3. 빠른 판단

- `HTTP/REST`, `FTP`, `SOAP/Axis`, `SVNKit`은 현재 코드 근거가 강하다.
- `JSch`는 관리자 업로드 JSP 기준 직접 사용이 확인된다.

