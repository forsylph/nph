# JCAOS/SVNKit

> 최종 수정: 2026-03-08

---

## 1. 개요

NPH 시스템은 JCAOS(한국형 연동 솔루션)와 SVNKit을 사용한다.

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

### 3.1 개요

JCAOS(Java Common Architecture for Open Systems)는 한국형 시스템 연동 솔루션이다.

### 3.2 주요 용도

| 구분 | 용도 |
|------|------|
| **행정기관 연동** | 정부 시스템 연동 |
| **표준 프레임워크** | eGovFrame 연동 |
| **인터페이스** | 시스템 간 표준 인터페이스 |

### 3.3 특징

- 한국형 행정전산망 연동 표준
- XML 기반 메시지 교환
- 보안 통신 지원

---

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
| **JCAOS** | 1.4.7.7 | 한국형 연동 솔루션 |
| **SVNKit** | 1.8.14 | SVN 클라이언트 |

---

## 6. 관련 문서

- [README.md](./README.md)