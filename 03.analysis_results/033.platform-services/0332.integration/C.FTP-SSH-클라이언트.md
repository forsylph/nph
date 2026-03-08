# FTP/SSH 클라이언트

> 최종 수정: 2026-03-08

---

## 1. 개요

NPH 시스템은 edtFTPj를 사용한 FTP 전송 흔적이 강하게 확인된다. `JSch`는 JAR 포함에 그치지 않고 `NPH_HIS/webapp/Admin/temp/uploadFile.jsp`에서 직접 import 및 `new JSch()` 호출이 확인된다. 다만 현재 확인된 사용처는 관리자/임시 업로드 유틸 성격이며, 광범위한 운영 경로까지 확인된 것은 아니다.

## 1A. 직접 확인 근거 파일

| 구분 | 직접 확인 근거 |
|------|----------------|
| FTP 배치 | `nph/bat/sample/FtpUploadJob.java`, `FtpDownloadJob.java`, `nph/bat/hp/job/FtpUploadJob.java` |
| JSch 직접 사용 | `webapp/Admin/temp/uploadFile.jsp` |
| JAR | `edtftpj-2.0.1.jar`, `commons-net-2.0.jar`, `jsch-0.1.54.jar` |

---

## 2. JAR 파일

### 2.1 FTP

| 파일명 | 버전 | 용도 |
|--------|------|------|
| **edtftpj-2.0.1.jar** | 2.0.1 | FTP/FTPS 클라이언트 |
| **commons-net-2.0.jar** | 2.0 | Apache Commons Net (FTP 등) |

### 2.2 SSH/SFTP

| 파일명 | 버전 | 용도 |
|--------|------|------|
| **jsch-0.1.54.jar** | 0.1.54 | SSH/SFTP 클라이언트 |

---

## 3. edtFTPj

### 3.1 주요 클래스

```java
import com.enterprisedt.net.ftp.FTPClient;
import com.enterprisedt.net.ftp.FTPConnectMode;
import com.enterprisedt.net.ftp.FTPException;
```

### 3.2 FtpUploadJob.java

```java
package nph.bat.sample;

public class FtpUploadJob extends AbstractConfigUsingObject implements JobIF {
    // FTP 연결 설정
    public static final String CONNECTION_MODE_PASSIVATE = "PASSIVE";
    public static final String CONNECTION_MODE_ACTIVATE = "ACTIVE";

    // 연결 파라미터
    private String TARGET_IP;           // FTP 서버 IP
    private String TARGET_PORT;         // FTP 포트
    private String TARGET_ID;           // 계정 ID
    private String TARGET_PASSWORD;     // 계정 비밀번호
    private String TARGET_DIRECTORY;    // 대상 디렉토리

    // FTPClient 사용
    FTPClient ftp = new FTPClient(targetIp, Integer.parseInt(targetPort));
    ftp.login(targetId, targetPassword);
    ftp.put(localFile, remoteFilename);
}
```

### 3.3 FTP 사용 패턴

```mermaid
flowchart LR
    A[배치 Job] --> B[FtpUploadJob<br/>FtpDownloadJob]
    B --> C[FTPClient<br/>edtFTPj]
    C --> D[FTP 서버]
```

---

## 4. Commons Net

### 4.1 FTP 기능

Apache Commons Net은 FTP, FTPS, TFTP 등 다양한 네트워크 프로토콜을 지원한다.

### 4.2 주요 패키지

```java
import org.apache.commons.net.ftp.FTP;
import org.apache.commons.net.ftp.FTPClient;
import org.apache.commons.net.ftp.FTPSClient;
```

---

## 5. JSch (SSH/SFTP)

### 5.1 현재 확인 수준

- `jsch-0.1.54.jar` 실물은 존재한다.
- `NPH_HIS/webapp/Admin/temp/uploadFile.jsp`에서 `com.jcraft.jsch.*` import와 `new JSch()` 호출이 직접 확인된다.
- 다만 현재 직접 확인된 사용처는 관리자 업로드 JSP 1건이며, 배치/업무 서비스 전반으로 넓게 쓰인다고 단정할 단계는 아니다.

### 5.2 주요 패키지

```java
import com.jcraft.jsch.Channel;
import com.jcraft.jsch.ChannelSftp;
import com.jcraft.jsch.JSch;
import com.jcraft.jsch.Session;
```

---

## 6. 연동 사례

### 6.1 배치 FTP 전송

| Job 클래스 | 용도 |
|-----------|------|
| **FtpUploadJob** | FTP 파일 업로드 |
| **FtpDownloadJob** | FTP 파일 다운로드 |

### 6.2 연동 대상

| 구분 | 용도 |
|------|------|
| **PACS 연동** | 의료 영상 파일 전송 |
| **외부 기관** | 파일 송수신 |

---

## 7. 기술 스택

| 기술 | 버전 | 상태 |
|------|------|------|
| **edtFTPj** | 2.0.1 | FTP/FTPS 클라이언트 (주 사용) |
| **Commons Net** | 2.0 | 네트워크 유틸리티 |
| **JSch** | 0.1.54 | 관리자 업로드 JSP 기준 직접 사용 확인 |

---

## 8. 관련 문서

- [README.md](./README.md)
- [B.HTTP-REST-클라이언트.md](./B.HTTP-REST-클라이언트.md)

