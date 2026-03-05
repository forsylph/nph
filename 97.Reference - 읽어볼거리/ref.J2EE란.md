# J2EE(Java 2 Platform, Enterprise Edition)란?

> Java 기반 엔터프라이즈 애플리케이션 개발 표준 플랫폼

---

## 개요

| 항목 | 내용 |
|------|------|
| **정식 명칭** | Java 2 Platform, Enterprise Edition |
| **약자** | J2EE (현재는 Java EE, Jakarta EE) |
| **출시** | 1999년 |
| **개발사** | Sun Microsystems → Oracle → Eclipse Foundation |
| **목적** | 대규모 기업용 애플리케이션 개발 표준 |

---

## 왜 만들어졌나?

### 1990년대 후반 기업용 애플리케이션의 문제

**기존 방식의 어려움:**
1. **복잡한 시스템 통합** - ERP, CRM, DB, 메시지 큐 등 연결의 어려움
2. **확장성 부족** - 트래픽 증가 시 시스템 확장이 어려움
3. **이식성 문제** - 벤더별로 다른 기술 스택
4. **분산 처리** - 여러 서버에서 애플리케이션 실행의 복잡성

**필요했던 것:**
- ✅ 표준화된 기업용 개발 플랫폼
- ✅ 분산 애플리케이션 지원
- ✅ 트랜잭션 관리
- ✅ 보안
- ✅ 확장성

---

## J2EE의 핵심 구성요소

```
┌─────────────────────────────────────────────────────────┐
│                    J2EE Platform                        │
├─────────────────────────────────────────────────────────┤
│  Presentation Layer (Web)                               │
│  ├── Servlet (HTTP 요청 처리)                           │
│  ├── JSP (JavaServer Pages - 화면 표현)                 │
│  └── JavaBeans (컴포넌트)                               │
├─────────────────────────────────────────────────────────┤
│  Business Layer (EJB)                                   │
│  ├── Session Bean (비즈니스 로직)                       │
│  ├── Entity Bean (데이터 persistence)                    │
│  └── Message-Driven Bean (비동기 메시지 처리)             │
├─────────────────────────────────────────────────────────┤
│  Data Access Layer                                      │
│  ├── JDBC (데이터베이스 연결)                           │
│  ├── JTA/JTS (트랜잭션 관리)                            │
│  └── JNDI (네이밍 서비스)                              │
├─────────────────────────────────────────────────────────┤
│  Messaging & Integration                                  │
│  ├── JMS (Java Message Service)                         │
│  ├── JavaMail (이메일)                                  │
│  └── JCA (Connector Architecture)                       │
├─────────────────────────────────────────────────────────┤
│  Web Services                                             │
│  ├── JAX-RPC (XML Web Services)                         │
│  └── JAXR (Registry)                                    │
└─────────────────────────────────────────────────────────┘
```

---

## J2EE 핵심 기술 상세

### 1. Servlet (서블릿)

**역할:** HTTP 요청을 받아 처리하는 Java 클래스

```java
// Servlet 예시
public class HelloServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse res) {
        res.setContentType("text/html");
        PrintWriter out = res.getWriter();
        out.println("<h1>Hello, J2EE!</h1>");
    }
}
```

**특징:**
- HTTP 요청/응답 처리
- 스레드 기반 (각 요청별 스레드 생성)
- 플랫폼 독립적

---

### 2. JSP (JavaServer Pages)

**역할:** HTML에 Java 코드를 삽입하여 동적 웹 페이지 생성

```jsp
<%@ page language="java" %>
<html>
<body>
    <h1>Welcome, <%= request.getParameter("username") %></h1>

    <%-- Java 코드 삽입 --%>
    <% for (int i = 0; i < 5; i++) { %>
        <p>Count: <%= i %></p>
    <% } %>
</body>
</html>
```

**JSP vs Servlet:**
| JSP | Servlet |
|-----|---------|
| HTML 중심 | Java 코드 중심 |
| 디자이너 친화적 | 개발자 중심 |
| 내부적으로 Servlet으로 변환 | 직접 HTTP 처리 |

---

### 3. EJB (Enterprise JavaBeans)

**역할:** 분산 환경에서의 비즈니스 로직 컴포넌트

**EJB 종류:**

| 종류 | 용도 | 예시 |
|------|------|------|
| **Session Bean** | 비즈니스 로직 | 주문 처리, 계산 |
| **Entity Bean** | 데이터 persistence | 고객 정보, 상품 정보 |
| **Message-Driven Bean** | 비동기 메시지 처리 | 메일 발송, 배치 작업 |

**EJB의 문제:**
- 설정이 복잡함 (ejb-jar.xml)
- 무겁고 느림
- 테스트 어려움
- **Spring이 대체하게 됨**

---

### 4. JDBC (Java Database Connectivity)

**역할:** Java에서 DB 연결 표준 API

```java
// JDBC 예시
Connection conn = DriverManager.getConnection(
    "jdbc:oracle:thin:@localhost:1521:ORCL", "user", "pass");

Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM users");

while (rs.next()) {
    System.out.println(rs.getString("username"));
}
```

---

### 5. JMS (Java Message Service)

**역할:** 비동기 메시지 처리 표준

```java
// JMS 예시
QueueConnectionFactory factory =
    (QueueConnectionFactory) ctx.lookup("QueueConnectionFactory");
QueueConnection conn = factory.createQueueConnection();
QueueSession session = conn.createQueueSession(false, Session.AUTO_ACKNOWLEDGE);

QueueSender sender = session.createSender(queue);
TextMessage message = session.createTextMessage("Hello, JMS!");
sender.send(message);
```

---

### 6. JNDI (Java Naming and Directory Interface)

**역할:** 리소스 이름으로 객체 찾기

```java
// JNDI 예시
Context ctx = new InitialContext();
DataSource ds = (DataSource) ctx.lookup("java:comp/env/jdbc/MyDB");
Connection conn = ds.getConnection();
```

---

## J2EE 아키텍처

### 전형적인 J2EE 애플리케이션 구조

```
┌────────────────────────────────────────────────────┐
│                   Client Tier                        │
│  ├── Web Browser (HTML/JavaScript)                 │
│  └── Java Application (Swing/AWT)                    │
└────────────────────┬───────────────────────────────┘
                     │ HTTP / RMI
┌────────────────────▼───────────────────────────────┐
│                 Web Tier                           │
│  ┌─────────────┐      ┌─────────────┐              │
│  │   Servlet   │──────▶│     JSP     │              │
│  └─────────────┘      └─────────────┘              │
│        │                                            │
│  ┌─────▼──────────┐                               │
│  │  JavaBeans     │                                │
│  │  (Form/Model)  │                                │
│  └────────────────┘                               │
└────────────────────┬───────────────────────────────┘
                     │
┌────────────────────▼───────────────────────────────┐
│              Business Tier                         │
│  ┌─────────────────────────────────────────────┐  │
│  │         EJB Container                        │  │
│  │  ┌─────────────┐    ┌─────────────┐         │  │
│  │  │ Session Bean│───▶│  Entity Bean│         │  │
│  │  │(Business   │    │  (Data)     │         │  │
│  │  │ Logic)     │    │             │         │  │
│  │  └─────────────┘    └─────────────┘         │  │
│  │         │                                    │  │
│  │  ┌──────▼──────┐                            │  │
│  │  │ Message-Driven│                          │  │
│  │  │ Bean (JMS)  │                            │  │
│  │  └─────────────┘                            │  │
│  └─────────────────────────────────────────────┘  │
└────────────────────┬───────────────────────────────┘
                     │ JDBC / JTA
┌────────────────────▼───────────────────────────────┐
│                EIS Tier                            │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐             │
│  │Database │  │  MQ     │  │  ERP    │             │
│  │(Oracle) │  │(Queue)  │  │(SAP)   │             │
│  └─────────┘  └─────────┘  └─────────┘             │
└────────────────────────────────────────────────────┘
```

---

## J2EE와 DevOn/Struts의 관계

### Struts 1.x가 J2EE 위에서 동작

```
┌────────────────────────────────────────────────┐
│              Presentation Layer                │
│  ┌────────────────────────────────────────┐   │
│  │          Struts 1.x                    │   │
│  │  ┌──────────┐      ┌────────────┐       │   │
│  │  │ ActionServlet│───▶│   Action   │       │   │
│  │  │ (Controller)│    │ (Command)  │       │   │
│  │  └──────────┘      └────────────┘       │   │
│  │        │                                  │   │
│  │  ┌─────▼──────┐                          │   │
│  │  │ ActionForm │                          │   │
│  │  │ (Model)    │                          │   │
│  │  └────────────┘                          │   │
│  └────────────────────────────────────────┘   │
│                    │                          │
│              ┌─────▼─────┐                   │
│              │    JSP    │ (View)            │
│              └───────────┘                   │
└────────────────────────────────────────────────┘
                     │
┌────────────────────▼─────────────────────────────┐
│              J2EE Platform                       │
│  ├── Servlet Container (Tomcat, JEUS)          │
│  ├── JSP Engine                                  │
│  ├── EJB Container (선택적)                      │
│  └── JNDI, JDBC, JTA 등                         │
└────────────────────────────────────────────────┘
```

**DevOn의 구조:**
```
DevOn Framework (Struts 1.x 기반)
        │
        ├── MiplatformServlet (ActionServlet 역할)
        ├── Command (Action 역할)
        ├── Navigation (struts-config.xml 역할)
        └── PC (Session Bean 역할 대체)
                │
                ▼
        J2EE Platform (Servlet, JSP, JDBC)
```

---

## J2EE의 변천사

### 이름 변경

| 시기 | 이름 | 주요 특징 |
|------|------|----------|
| **1999~2006** | J2EE | Java 2 Platform, Enterprise Edition |
| **2006~2017** | Java EE | Java Platform, Enterprise Edition (Java 5부터) |
| **2017~현재** | Jakarta EE | Oracle → Eclipse Foundation 이관 후 |

### 버전별 특징

| 버전 | 연도 | 주요 특징 |
|------|------|----------|
| **J2EE 1.2** | 1999 | 초기 버전, EJB 1.0 |
| **J2EE 1.3** | 2001 | EJB 2.0, JMS |
| **J2EE 1.4** | 2003 | Web Services 지원 |
| **Java EE 5** | 2006 | 어노테이션 도입 (@Entity) |
| **Java EE 6** | 2009 | CDI, JAX-RS (REST) |
| **Java EE 7** | 2013 | WebSocket, JSON-P |
| **Java EE 8** | 2017 | 마지막 Oracle 버전 |
| **Jakarta EE 8** | 2019 | Eclipse Foundation 이관 |
| **Jakarta EE 9** | 2020 | javax → jakarta 패키지 변경 |
| **Jakarta EE 10** | 2022 | 최신 버전 |

---

## J2EE vs Spring

### J2EE의 문제와 Spring의 등장

| 문제 | J2EE 방식 | Spring 방식 |
|------|-----------|-------------|
| **복잡성** | EJB가 복잡함 | POJO 기반 단순함 |
| **테스트** | EJB 컨테이너 필요 | 단위 테스트 용이 |
| **설정** | XML이 많음 (ejb-jar.xml) | 어노테이션 선호 |
| **의존성** | JNDI lookup | DI (Dependency Injection) |
| **트랜잭션** | EJB에서 선언적 | AOP로 선언적 |

### Spring이 대체한 부분

```
Before (J2EE만 사용):
JSP + Servlet + EJB + JNDI

After (Spring 사용):
JSP/Thymeleaf + Spring MVC + Spring + JDBC/JPA
```

| J2EE 기술 | Spring 대안 |
|-----------|-------------|
| EJB Session Bean | Spring Service (@Service) |
| EJB Entity Bean | JPA / Spring Data |
| JNDI lookup | DI (Dependency Injection) |
| JTA (트랜잭션) | @Transactional |
| Servlet/JSP | Spring MVC / Spring Boot |

---

## DevOn과 J2EE

### DevOn이 J2EE를 사용하는 방식

**DevOn Framework = Struts 1.x (MVC) + J2EE (Servlet/JDBC)**

| 계층 | 기술 |
|------|------|
| **View** | MiPlatform (JSP 대체) |
| **Controller** | Servlet (MiplatformServlet) |
| **Business** | Command + PC (EJB 대체) |
| **Data Access** | XML Query + JDBC |
| **Container** | JEUS (J2EE WAS) |

**J2EE 기술 사용 현황:**
- ✅ **Servlet**: MiplatformServlet, GeneralServlet
- ✅ **JDBC**: LCommonDao, LQueryService
- ✅ **JNDI**: DataSource lookup
- ✅ **JTA**: 트랜잭션 관리
- ❌ **JSP**: MiPlatform으로 대체
- ❌ **EJB**: PC (Process Component)로 대체
- ❌ **JMS**: 내부 메시지 큐 사용 (일부)

---

## 요약

### J2EE란?

> **Java 기반 엔터프라이즈 애플리케이션 개발 표준 플랫폼**

- **1999년** Sun Microsystems에서 출시
- **Servlet, JSP, EJB, JDBC** 등의 기술 집합
- 대규모 기업 시스템 개발을 위한 표준
- 현재는 **Jakarta EE**로 계속 발전 중

### DevOn 사용자에게

**DevOn Framework는 J2EE 플랫폼 위에서 동작합니다:**

1. **JEUS 6.0** (J2EE WAS)에서 실행
2. **Servlet/JDBC** (J2EE 기술) 사용
3. **Struts 1.x** (J2EE 기반 프레임워크) 기반

**J2EE의 현재:**
- EJB는 거의 사용되지 않음 (Spring 대체)
- Servlet/JSP는 여전히 기반 기술
- Jakarta EE로 계속 발전 중
- Spring Boot가 대세

**마이그레이션 시 고려사항:**
- J2EE → Spring Boot
- DevOn → Spring Framework
- MiPlatform → Nexacro/웹 표준

---

*"J2EE는 Java 엔터프라이즈 개발의 역사이자 기초입니다."*

**참고자료:**
- [Jakarta EE Official](https://jakarta.ee/)
- [Java EE Tutorial (Oracle)](https://docs.oracle.com/javaee/7/tutorial/)
- [J2EE - Wikipedia](https://en.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition)
