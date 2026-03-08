# Integration 개요

> 최종 수정: 2026-03-08

---

## 1A. 상위 연결

- 이 폴더의 기준 설명은 [../README.md](../README.md) 를 먼저 본다.
- DevOn 코어는 [../../032.framework-core/0321.overview/A.Framework-개요.md](../../032.framework-core/0321.overview/A.Framework-개요.md) 와 같이 본다.

이 문서는 외부 시스템과의 공통 연동에 사용되는 솔루션과 패키지를 정리하는 기준본이다.

---

## 1. 현재 확인된 강한 사용 근거

- HTTP/REST: `com.rest.api.*`, `com.rest.util.HttpClientUtil`, `org.apache.http.*`, `javax.ws.rs.*`, `com.sun.jersey.*`
- FTP: `nph.bat.sample.FtpUploadJob`, `FtpDownloadJob`, `nph.bat.hp.job.FtpUploadJob`, `com.enterprisedt.net.ftp.*`
- SOAP/Axis: `SavekimsCMD`, `kr.co.kimsonline.poc.MedicalInfoSoapStub`, 다수 `org.apache.axis.*` 생성/VO 코드
- SVNKit: `SvnLogPC`, `org.tmatesoft.svn.*`

추가 검증 결과 `JSch`는 `Admin/temp/uploadFile.jsp`에서 직접 사용이 확인되었고, `xldap`는 `UserMngmPC`, `EamIFUC`, `ComLoginUC`, `MenuInfoCMD` 등에서 직접 사용이 확인되었다. `JCAOS`는 애플리케이션 소스 import는 미확인이지만 `dsagent.properties`의 `crypto.type=JCAOS`, `magicsaml-sp-v1.3.3.jar`의 `JCAOSCryptoApi`, `jcaos-1.4.7.7.jar`의 `JCAOSProvider` 기준으로 런타임 구성 관여는 확인된다.

`2026-03-04` 운영 로그 기준으로는 `AsyncWorkContextListener` 초기화, `HttpClientUtil` 인증서 로드, `LST`/`POLNET` 핸들러 등록이 직접 확인된다. 즉 `0332.integration`에 정리된 외부 연동 계층은 단순 라이브러리 보유가 아니라 실제 사이트 기동 과정에 참여한다.

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
| 운영 로그 | `devonhome/logs/20260304PMinfo.log`, `devonhome/logs/20260304PMdebug.log` |

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

## 3. 기술 스택 요약

| 기술 | 상태 | 비고 |
|------|------|------|
| **Apache HttpClient** | ✅ 직접 사용 확인 | `HttpClientUtil`, `PolNetUC`, `LstUC` |
| **Jersey/JAX-RS** | ✅ 직접 사용 확인 | `com.rest.api.*`, `web.xml` |
| **edtFTPj** | ✅ 직접 사용 확인 | 배치 FTP 전송 |
| **JSch** | ✅ 직접 사용 확인 | 관리자 업로드 JSP 1건 |
| **xldap** | ✅ 직접 사용 확인 | EAM/권한/사용자 관리 |
| **Apache Axis** | ✅ 직접 사용 확인 | KIMS SOAP 연동 |
| **JCAOS** | ⚠️ 런타임 구성 관여 확인 | 앱 소스 직접 import 미확인 |
| **SVNKit** | ✅ 직접 사용 확인 | `SvnLogPC`, `SvnLogIFPC` |

---

## 4. 문서별 한 줄 요약

| 문서 | 한 줄 요약 |
|------|------------|
| [B.HTTP-REST-클라이언트.md](./B.HTTP-REST-클라이언트.md) | 내부 REST 서버와 외부 HTTP 호출 클라이언트를 함께 정리한 문서 |
| [C.FTP-SSH-클라이언트.md](./C.FTP-SSH-클라이언트.md) | 배치 FTP 전송과 관리자 JSP 기반 JSch 사용 흔적을 정리한 문서 |
| [D.LDAP-인증연동.md](./D.LDAP-인증연동.md) | xldap 직접 사용과 LDAP 설정 흔적을 기준으로 정리한 문서 |
| [E.SOAP-웹서비스.md](./E.SOAP-웹서비스.md) | KIMS 중심 SOAP/Axis 호출 패턴과 생성 프록시 사용을 정리한 문서 |
| [F.JCAOS-SVNKit.md](./F.JCAOS-SVNKit.md) | JCAOS의 런타임 구성 관여와 SVNKit 직접 사용을 함께 정리한 문서 |

---

## 5. 빠른 판단

- `HTTP/REST`, `FTP`, `SOAP/Axis`, `SVNKit`은 현재 코드 근거가 강하다.
- `JSch`는 관리자 업로드 JSP 기준 직접 사용이 확인된다.
- `JCAOS`는 설정/JAR 기준 관여는 확인되지만, 앱 소스 직접 호출은 아직 미확인이다.

---

## 6. 빌드 시스템 참고

### 6.1 Ant 사용 현황

| 파일 | 위치 | 용도 |
|------|------|------|
| `build.xml` | `NPH_BUILD/` | 배치 모듈 컴파일 |
| `build-wstest.xml` | `NPH_HIS/ant/` | JAR 패키징 |

NPH는 Ant를 사용하나 **Axis 전용 태스크는 미사용**이다. 일반 Ant 태스크(`<javac>`, `<jar>`, `<copy>`)만 사용.

### 6.2 미사용 JAR

| JAR 파일 | 원래 용도 | 미사용 사유 |
|----------|----------|-------------|
| **axis-ant.jar** | Ant용 Axis 태스크 (`axis-java2wsdl`, `axis-wsdl2java`) | NPH는 Axis 전용 태스크 미사용, 런타임에 불필요 |

**상세 분석**: [Minority Report/NPH-빌드-배포-분석.md](../../038.fact-todo-reference/0389.Minority%20Report/NPH-빌드-배포-분석.md)
