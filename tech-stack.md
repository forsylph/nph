# NPH 기술 스택 리스트

> 분석일: 2026-03-05
> 분석 대상: `/mnt/n/99.SourceCode Backup/NPH/AADEV_NPH/workspace`

---

## 1. UI/프론트엔드

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| RIA 플랫폼 | **MiPlatform** | 3.2 / 3.3 | 투비소프트 | ActiveX 기반 |
| EMR 뷰어 | **eView** | - | - | EMR 문서 뷰어 |
| 서명패드 | **SignPad** | - | - | Java Applet 기반 전자서명 |
| 차트 | **ChartController** | - | - | 차트 컨트롤러 |
| 가계도 | **PedigreeChart** | - | - | 유전질환 가계도 차트 |
| 이미지 입력 | **ImageInputController** | - | - | 이미지 입력 컨트롤러 |
| 캡차 | **jj-simplecaptcha** | - | - | CAPTCHA 생성 |

---

## 2. 백엔드 프레임워크

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| 프레임워크 | **DevOn Framework** | 4.0.0 | LG CNS | Struts 1.x 기반 |
| 배치 프레임워크 | **DevOn Batch** | 1.1.0 | LG CNS | 배치 처리 |
| DB 쿼리 | **DevOn XML Query** | - | LG CNS | XML 기반 SQL (LQueryService) |

---

## 3. API/웹서비스

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| REST API | **Jersey** | 1.19.4 | Eclipse | JAX-RS 구현체 |
| SOAP | **Apache Axis** | 1.x | Apache | 웹서비스 |
| HTTP 클라이언트 | **Apache HttpClient** | 4.5.3 | Apache | HTTP 통신 |

---

## 4. 리포팅

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| 리포트 엔진 | **Rexpert** | - | - | 엔터프라이즈 리포팅 |
| 의료 리포트 | **TPR Report** | - | - | TPR 차트 리포트 |
| Excel 처리 | **Apache POI** | 3.2 | Apache | Excel 읽기/쓰기 |
| Excel 처리 | **JExcelApi** | - | - | Excel 처리 |
| PDF 생성 | **iText XML Worker** | 1.2.0 | iText | PDF 생성 |

---

## 5. 비즈니스 규칙

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| 규칙 엔진 | **InnoRule** | - | 이노리저(InnoExpert) | 비즈니스 규칙 엔진 |
| 입력 검증 | **Requisite** | - | - | 입력값 검증 프레임워크 |

---

## 6. 스케줄러

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| 작업 스케줄러 | **Quartz** | 1.6.1 | Terracotta | 스케줄링 |

---

## 7. 보안/인증

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| SSO | **MagicSSO** | 3.5 | 드림시큐리티 | Single Sign-On |
| 전자서명 | **SignGate** | - | KIS | 전자서명 솔루션 |
| 공인인증 | **GPKI** | - | KISA | 공인인증서 기반 인증 |
| SAML SP | **MagicSAML** | 1.3.3 | 드림시큐리티 | SAML Service Provider |
| SAML | **OpenSAML** | 2.6.4 | Shibboleth | SAML 라이브러리 |
| XSS 방어 | **Lucy XSS Filter** | 1.1.2 | 네이버 | XSS 필터 (**미사용**: JAR만 존재, 설정 없음) |
| 보안 API | **OWASP ESAPI** | 2.0.1 | OWASP | 보안 API |
| 암호화 | **Bouncy Castle** | 1.51 | - | 암호화 라이브러리 |
| SSL | **not-yet-commons-ssl** | 0.3.9 | - | SSL 유틸리티 |

---

## 8. 의료/EMR

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| EMR 뷰어 | **eView** | - | - | EMR 문서 뷰어 |
| 건강보험 연동 | **WSNhic** | - | - | 국민건강보험공단 연동 |
| 서명패드 | **SignPad** | - | - | 전자서명 입력 |
| 가계도 | **PedigreeChart** | - | - | 유전질환 가계도 |
| DSToolkit | **DSToolkit** | 3.4.2.0 | 드림시큐리티 | SSO 구성요소 |
| EMR 유틸 | **emr_Utils** | - | - | EMR 유틸리티 |
| EzIssuer | **EzIssuer** | - | - | 의료 관련 |

---

## 9. 연동/통신

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| FTP | **edtFTPj** | 2.0.1 | EnterpriseDT | FTP 클라이언트 |
| SFTP/SSH | **JSch** | 0.1.54 | JCraft | SSH/SFTP |
| FTP/SMTP | **Commons Net** | 2.0 | Apache | 네트워크 유틸리티 |
| LDAP | **ldapjdk** | - | - | LDAP 연동 |
| LDAP | **JCAOS** | 1.4.7.7 | - | LDAP/인증 |
| SVN | **SVNKit** | 1.8.14 | TMate | SVN 클라이언트 |

---

## 10. 데이터베이스

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| Oracle JDBC | **ojdbc6** | - | Oracle | Oracle 연결 |
| Tibero JDBC | **tibero5-jdbc** | - | 티맥스 | Tibero 연결 |
| Cache DB | **cachedb / cachejdbc** | - | - | 캐시 DB |

---

## 11. 로깅

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| 로깅 | **Log4j** | 1.2.8 / 1.2.16 | Apache | 로깅 |
| 로깅 | **Logback** | 1.1.3 / 1.1.7 | QOS.ch | 로깅 |
| 로깅 facade | **SLF4J** | 1.7.13 / 1.7.21 | QOS.ch | 로깅 추상화 |

---

## 12. 유틸리티

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| 문서 파싱 | **Apache Tika** | 1.6 | Apache | 문서 메타데이터 추출 |
| JSON | **JSON-lib** | 1.0 | - | JSON 처리 |
| JSON | **Jettison** | 1.5.4 | Codehaus | JSON 처리 |
| RSS/Atom | **Rome** | 1.0 | Rome | RSS/Atom 피드 |
| 날짜/시간 | **Joda-Time** | 2.2 | Joda.org | 날짜 처리 |
| 한글 처리 | **changekor** | - | - | 한글 변환 |
| XML | **JDOM** | - | JDOM | XML 처리 |
| XML | **Jaxen** | 1.1.6 | Jaxen | XPath |
| 파일 업로드 | **Commons FileUpload** | 1.2.1 | Apache | 파일 업로드 |
| 문자열 | **Commons Lang** | 2.6 | Apache | 문자열 유틸리티 |
| 컬렉션 | **Commons Collections** | 3.2 | Apache | 컬렉션 유틸리티 |
| 이메일 | **JavaMail** | - | Oracle | 이메일 |
| 마이그레이션 | **Wiseone Migration** | 3.3.0 | Wiseone | HR 데이터 마이그레이션 |

---

## 13. 라이브러리 (Apache Commons)

| 구분 | 기술 | 버전 | 용도 |
|------|------|------|------|
| Commons CLI | 1.2 | 커맨드라인 파싱 |
| Commons Codec | 1.3 / 1.10 | 인코딩/디코딩 |
| Commons Discovery | 0.2 | 서비스 발견 |
| Commons HTTP Client | 3.1 | HTTP 클라이언트 |
| Commons IO | 1.2 | I/O 유틸리티 |
| Commons Logging | 1.1 | 로깅 추상화 |

---

## 14. 기타

| 구분 | 기술 | 버전 | 용도 |
|------|------|------|------|
| ASM | 3.1 | 바이트코드 조작 |
| CGLIB | 2.2 | 프록시 생성 |
| Backport Concurrent | 3.1 | 동시성 유틸리티 |
| Atom | 1.1 / 3.4 | XLog |
| COS | - | 파일 업로드 |
| JSTL | - | JSP 태그 라이브러리 |
| Standard | - | JSP 표준 태그 |
| Xerces | - | XML 파싱 |
| Xalan | - | XSLT |

---

## 15. 인프라

| 구분 | 기술 | 버전 | 공급사 | 비고 |
|------|------|------|--------|------|
| WAS | **JEUS** | 6.0 | 티맥스소프트 | 웹애플리케이션서버 |
| JDK | **Java** | 1.7.0_80 | Oracle | 개발키트 |
| DB | **Oracle** | - | Oracle | 메인 DB |
| DB | **Tibero** | 5 | 티맥스 | 메인 DB |

---

## 16. 데이터소스

| 데이터소스명 | 용도 |
|-------------|------|
| jdbc/NPHDB | 메인 DB |
| jdbc/NPHENCDB | 암호화 DB |
| jdbc/NPHSMSDB | SMS DB |
| jdbc/NPHDIFDB | DIF DB |
| jdbc/NPHBKDB | 백업 DB |
| jdbc/NPHQCDB | QC DB |
| jdbc/NPHINFDB | INF DB |
| jdbc/NPHGW | 게이트웨이 |

---

## 17. 빌드/테스트 전용

| 구분 | 기술 | 버전 | 비고 |
|------|------|------|------|
| ORM | **MyBatis** | 3.4.1 | NPH_BUILD 프로젝트에서만 사용 (테스트/마이그레이션 용도) |

> **참고**: MyBatis는 메인 시스템(NPH_HIS, NPH_ECS, COMMON)에서 사용하지 않습니다.
> DevOn Framework는 자체 **XML 기반 쿼리 시스템 (LQueryService)** 을 사용합니다.
> 경로: `devonhome/xmlquery/**/*.xml`

---

## 종합 요약

### 카테고리별 솔루션 수

| 카테고리 | 솔루션 수 |
|----------|----------|
| UI/프론트엔드 | 7 |
| 백엔드 프레임워크 | 3 |
| API/웹서비스 | 3 |
| 리포팅 | 5 |
| 비즈니스 규칙 | 2 |
| 스케줄러 | 1 |
| 보안/인증 | 9 |
| 의료/EMR | 7 |
| 연동/통신 | 7 |
| 데이터베이스 | 3 |
| 로깅 | 3 |
| 유틸리티 | 14 |
| 인프라 | 4 |

### 주요 공급사

| 공급사 | 솔루션 |
|--------|--------|
| **LG CNS** | DevOn Framework, DevOn Batch |
| **투비소프트** | MiPlatform |
| **드림시큐리티** | MagicSSO, MagicSAML, DSToolkit |
| **KIS** | SignGate |
| **이노리저** | InnoRule |
| **티맥스소프트** | JEUS, Tibero |
| **Apache** | Axis, POI, Commons, HttpClient, Tika |
| **네이버** | Lucy XSS Filter |

---

*분석 완료*