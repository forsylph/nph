# FTP/SSH 클라이언트

> 최종 수정: 2026-03-08

---

## 1. 개요

NPH 시스템은 edtFTPj와 JSch를 사용하여 파일 전송(FTP/SFTP)을 수행한다.

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

### 5.1 용도

- SFTP 파일 전송
- SSH 원격 명령 실행
- 포트 포워딩

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
| **JSch** | 0.1.54 | SSH/SFTP 클라이언트 |

---

## 8. 관련 문서

- [README.md](./README.md)
- [B.HTTP-REST-클라이언트.md](./B.HTTP-REST-클라이언트.md)