# JCAOS/SVNKit

> 최종 수정: 2026-03-08

---

## 1. 개요

NPH 시스템은 `SVNKit`의 직접 사용 흔적이 확인된다. `JCAOS`는 JAR과 라이선스 파일 존재 수준을 넘어서 SSO/암호화 구성 관여가 확인된다. 다만 현재 애플리케이션 소스에서 `com.dreamsecurity.jcaos.*` 직접 import는 확인되지 않았다.

## 1A. 직접 확인 근거 파일

| 구분 | 직접 확인 근거 |
|------|----------------|
| JCAOS 설정/런타임 | `webapp/WEB-INF/homepath/cfg/dsagent.properties`, `magicsaml-sp-v1.3.3.jar`, `jcaos-1.4.7.7.jar` |
| SVNKit 직접 사용 | `SvnLogPC.java`, `SvnLogIFPC.java`, `devonhome/instance/instance_az.properties` |

---

## 2. JAR 파일

### 2.1 JCAOS

| 파일명 | 버전 | 용도 |
|--------|------|------|
| **jcaos-1.4.7.7.jar** | 1.4.7.7 | 한국형 시스템 연동 솔루션 |
| **jcaos.lic** | - | JCAOS 라이선스 파일 |

### 2.2 SVNKit

| 파일명 | 버전 | 용도 |
|--------|------|------|
| **svnkit-1.8.14.jar** | 1.8.14 | SVN 클라이언트 라이브러리 |

---

## 3. JCAOS

### 3.1 현재 확인 수준

- `jcaos-1.4.7.7.jar`, `jcaos.lic` 실물은 존재한다.
- `dsagent.properties`의 `crypto.type=JCAOS`, `magicsaml-sp-v1.3.3.jar`의 `com.dreamsecurity.crypto.api.JCAOSCryptoApi`, `jcaos-1.4.7.7.jar`의 `com.dreamsecurity.JCAOSProvider`가 확인된다.
- 따라서 현재 가장 안전한 결론은 `JCAOS가 SSO/암호화 런타임 구성에 관여한다`는 수준이며, 앱 소스 직접 호출 여부는 추가 확인이 필요하다는 것이다.

## 4. SVNKit

### 4.1 개요

SVNKit은 Subversion(SVN) 저장소에 접근하는 순수 Java 라이브러리다.

### 4.2 주요 패키지

```java
import org.tmatesoft.svn.core.SVNException;
import org.tmatesoft.svn.core.SVNURL;
import org.tmatesoft.svn.core.wc.SVNClientManager;
import org.tmatesoft.svn.core.wc.SVNRevision;
```

### 4.3 용도

| 구분 | 용도 |
|------|------|
| **소스 버전 관리** | SVN 저장소 접근 |
| **배포 관리** | 소스 체크아웃/커밋 |
| **버전 추적** | 리비전 관리 |

---

## 5. 기술 스택

| 기술 | 버전 | 상태 |
|------|------|------|
| **JCAOS** | 1.4.7.7 | 런타임 구성 관여 확인, 앱 소스 직접 import 미확인 |
| **SVNKit** | 1.8.14 | SvnLogPC 기준 직접 사용 확인 |

---

## 6. 관련 문서

- [README.md](./README.md)

