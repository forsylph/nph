# Apache Struts 1.x 역사와 배경

> Java 웹 개발의 전성기, 그리고 레거시가 된 이유
> 참고: 본 문서는 일반 Struts 레퍼런스이며 NPH 실코드 경로/클래스와 1:1 대응 문서가 아닙니다.

---

## 목차

1. [Struts 1.x 탄생 배경](#1-struts-1x-탄생-배경)
2. [만들어진 이유](#2-만들어진-이유)
3. [Struts 1.x의 혁신](#3-struts-1x의-혁신)
4. [전성기 (2000~2008)](#4-전성기-20002008)
5. [왜 레거시가 되었나?](#5-왜-레거시가-되었나)
6. [Struts 1.x의 유산](#6-struts-1x의-유산)
7. [타임라인](#7-타임라인)

---

## 1. Struts 1.x 탄생 배경

### 기본 정보

| 항목 | 내용 |
|------|------|
| **만든 사람** | Craig McClanahan (Sun Microsystems) |
| **만든 시기** | 2000년 Memorial Day 주말 (3일 만에) |
| **Apache 기부** | 2000년 5월 |
| **1.0 릴리스** | 2001년 6월 |
| **EOL (지원 종료)** | 2013년 4월 |

### Craig McClanahan은 누구?

- **Sun Microsystems** 소속 개발자
- **Tomcat**의 Servlet 컨테이너(Catalina) 설계
- **Servlet 2.2, 2.3** 및 **JSP 1.1, 1.2** 사양 참여
- Java 웹 기술의 아버지 중 한 명

---

## 2. 만들어진 이유

### 2000년 초반 Java 웹 개발의 현실

**문제 1: Servlet/JSP 코드의 혼란**

```java
// 😱 기존 Servlet 개발 (2000년 초반)
public class OrderServlet extends HttpServlet {
    public void doPost(HttpServletRequest req, HttpServletResponse res) {
        // 1. 파라미터 수동 추출 (반복적인 코드)
        String customerId = req.getParameter("customerId");
        String productCode = req.getParameter("productCode");
        String quantity = req.getParameter("quantity");

        // 2. 검증 코드 (if문 덕지덕지)
        if (customerId == null || customerId.trim().equals("")) {
            req.setAttribute("error", "고객ID는 필수입니다");
            RequestDispatcher dispatcher =
                req.getRequestDispatcher("/error.jsp");
            dispatcher.forward(req, res);
            return;
        }

        if (productCode == null || productCode.trim().equals("")) {
            req.setAttribute("error", "상품코드는 필수입니다");
            // ... 반복
        }

        // 3. 형변환 (수동)
        int qty = 0;
        try {
            qty = Integer.parseInt(quantity);
        } catch (NumberFormatException e) {
            req.setAttribute("error", "수량은 숫자여야 합니다");
            // ...
            return;
        }

        // 4. 비즈니스 로직 + DB 접속 (Servlet에 다 있음)
        OrderService service = new OrderService();
        Order order = service.createOrder(customerId, productCode, qty);

        // 5. 결과를 request에 담아 JSP로 포워딩
        req.setAttribute("order", order);
        RequestDispatcher dispatcher =
            req.getRequestDispatcher("/orderResult.jsp");
        dispatcher.forward(req, res);
    }
}
```

**문제점:**
- ❌ 비즈니스 로직 + 화면 로직이 뒤섞임
- ❌ 파라미터 처리가 반복적이고 지루함
- ❌ 검증 코드가 Servlet에 가득
- ❌ 유지보수 어려움 (한 파일에 다 있음)
- ❌ 팀 개발이 어려움 (역할 분리 안됨)

---

### Struts 1.x가 해결한 것

**MVC 패턴 강제 적용:**

```java
// ✅ Struts 1.x (2000년대)
// 1. ActionServlet이 중앙에서 모든 요청 처리
// 2. ActionForm이 자동으로 파라미터 바인딩
// 3. Action이 비즈니스 로직만 처리

public class OrderAction extends Action {
    public ActionForward execute(ActionMapping mapping,
                                  ActionForm form,
                                  HttpServletRequest req, ...) {
        // Form이 자동으로 채워짐!
        OrderForm orderForm = (OrderForm) form;
        String customerId = orderForm.getCustomerId(); // 자동 바인딩
        int quantity = orderForm.getQuantity();        // 자동 형변환

        // 비즈니스 로직만 집중
        OrderService service = new OrderService();
        Order order = service.createOrder(orderForm);

        // 선언적 포워딩 (XML에 정의됨)
        return mapping.findForward("success");
    }
}
```

```xml
<!-- struts-config.xml -->
<form-bean name="orderForm" type="com.example.OrderForm"/>

<action path="/order" type="com.example.OrderAction" name="orderForm"
       validate="true" input="/orderForm.jsp">
    <forward name="success" path="/orderSuccess.jsp"/>
    <forward name="failure" path="/orderFailed.jsp"/>
</action>
```

**혜택:**
- ✅ Controller: ActionServlet이 중앙집중 처리
- ✅ Model: ActionForm이 데이터 바인딩 자동화
- ✅ View: JSP에서 Java 코드 분리 (태그 라이브러리)
- ✅ 설정: XML로 선언적 관리

---

## 3. Struts 1.x의 혁신

### 3.1 Model 2 아키텍처 표준화

```
Model 1 (기존): JSP <-> Servlet <-> Java Bean (복잡한 연결)
Model 2 (Struts): MVC 강제
        ┌─ Controller: ActionServlet
        ├─ View: JSP
        └─ Model: Action + Service
```

### 3.2 주요 혁신 기술

| 기술 | 설명 | 혁신성 |
|------|------|--------|
| **ActionServlet** | 모든 요청을 하나의 Servlet이 받아 분배 | 중앙집중 컨트롤러 |
| **ActionForm** | HTTP 파라미터 자동 바인딩 | 반복 코드 제거 |
| **Action** | 비즈니스 로직 전용 클래스 | 역할 분리 |
| **struts-config.xml** | XML 기반 선언적 설정 | 코드와 설정 분리 |
| **태그 라이브러리** | JSP에서 Java 코드 제거 | 뷰와 로직 분리 |
| **Tiles** | 화면 레이아웃 템플릿 | 재사용성 향상 |

### 3.3 기존 vs Struts 비교

| 작업 | 기존 Servlet/JSP | Struts 1.x |
|------|------------------|------------|
| **파라미터 추출** | `request.getParameter()` 반복 | `ActionForm` 자동 바인딩 |
| **형변환** | 수동 `Integer.parseInt()` | 자동 변환 |
| **검증** | Servlet에 if문 덕지덕지 | `validate()` 메소드 |
| **포워딩** | `RequestDispatcher` 직접 호출 | `mapping.findForward()` 선언적 |
| **설정** | Java 코드 하드코딩 | XML 중앙 관리 |

### 3.4 태그 라이브러리 혁명

```jsp
<%-- 😱 기존 JSP (Java 코드 뒤섞임) --%
<%
    Order order = (Order) request.getAttribute("order");
    if (order != null) {
        out.println("Order ID: " + order.getId());
        out.println("Customer: " + order.getCustomerName());
    }

    List<Item> items = order.getItems();
    for (Item item : items) {
        out.println("Item: " + item.getName() +
                   " Price: " + item.getPrice());
    }
%

<%-- ✅ Struts 태그 (선언적) --%
<%@ taglib prefix="bean" uri="http://struts.apache.org/tags-bean" %>
<%@ taglib prefix="logic" uri="http://struts.apache.org/tags-logic" %>

<bean:write name="order" property="id"/>
<bean:write name="order" property="customerName"/>

<logic:iterate id="item" name="order" property="items">
    Item: <bean:write name="item" property="name"/>
    Price: <bean:write name="item" property="price"/>
</logic:iterate>
```

---

## 4. 전성기 (2000~2008)

### 4.1 킬러 앱으로 등장

**당시 상황:**
- Java 웹 개발의 **사실상 표준**
- **"If you know Struts, you're Golden"**
- Java 웹 개발자 **90% 이상**이 사용
- JavaOne 컨퍼런스 주요 주제

**채용 시장:**
> "Struts 경험 필수" - 2000년대 중반 Java 웹 개발자 채용 공고

### 4.2 대기업 표준 채택

| 기업/기관 | 적용 사례 |
|-----------|-----------|
| **LG CNS** | DevOn Framework (Struts 기반) |
| **삼성 SDS** | 내부 시스템 |
| **국내 대형 SI** | 대부분 Struts 표준 |
| **전 세계 엔터프라이즈** | 수많은 금융, 공공 시스템 |

### 4.3 경쟁 프레임워크와의 비교

| 프레임워크 | 시대 | 결과 |
|------------|------|------|
| **Struts 1.x** | 2000~2008 | ✅ **승리** (사실상 표준) |
| **Apache Tapestry** | 2000년대 | ❌ 소수 사용 |
| **Apache Cocoon** | 2000년대 | ❌ 복잡성 문제 |
| **JavaServer Faces (JSF)** | 2004년~ | ❌ 표준이지만 실패 |
| **Spring MVC** | 2005년~ | ⏳ Struts 대항마로 성장 중 |

### 4.4 Struts 1.x의 영향

**후계자들:**
- **Spring MVC**: Struts의 아이디어를 개선
- **WebWork**: Struts 2로 흡수됨
- **JSF**: Java EE가 Struts 대항마로 실패

---

## 5. 왜 레거시가 되었나?

### 5.1 기술의 변화

| 시대 | 새로운 기술 | Struts의 한계 |
|------|-------------|---------------|
| **2005년** | 어노테이션 등장 | XML이 장황해짐 |
| **2006년** | Spring Framework 부상 | DI, AOP 지원 없음 |
| **2008년** | RESTful API 유행 | SOAP/XML 중심 설계 |
| **2010년** | Spring MVC 성숙 | 더 유연한 대안 |
| **2013년** | Spring Boot 등장 | Convention over XML |

### 5.2 주요 문제점

**1. XML 설정의 복잡성**
```xml
<!-- struts-config.xml이 프로젝트 규모에 따라 거대해짐 -->
<struts-config>
    <form-beans>
        <form-bean name="userForm" type="..."/>
        <form-bean name="orderForm" type="..."/>
        <!-- 수십~수백개의 Form 정의 -->
    </form-beans>

    <action-mappings>
        <action path="/user/*" ...>
            <forward name="list" path="..."/>
            <forward name="detail" path="..."/>
            <!-- 수십~수백개의 Action 정의 -->
        </action>
    </action-mappings>
</struts-config>
```

**2. 유연성 부족**
- 강제적인 MVC 구조가 오히려 제약
- 상속 기반 설계 (Action 클래스 상속 필수)
- 테스트 어려움 (Servlet 컨테이너 의존)

**3. 새로운 패러다임 등장**
```
Struts 1.x: XML 중심, 상속 기반, Servlet 의존
    ↓
Spring MVC: 어노테이션, POJO, DI
    ↓
Spring Boot: Convention, Auto-config, Embedded
```

### 5.3 EOL (End of Life)

| 날짜 | 이벤트 |
|------|--------|
| **2008년 12월** | Struts 1.3.10 (최종 릴리스) |
| **2013년 4월** | 공식 EOL 선언 |
| **현재** | 보안 패치 중단, 유지보수 모드 |

---

## 6. Struts 1.x의 유산

### 6.1 현재도 남아있는 이유

| 사례 | 설명 |
|------|------|
| **DevOn Framework** | LG CNS의 Struts 기반 프레임워크 |
| **엔터프라이즈 시스템** | 수많은 구축 시스템이 아직도 Struts 1.x로 운영 중 |
| **교육 목적** | MVC 패턴 이해의 교과서 |
| **마이그레이션** | Struts → Spring Boot 전환 프로젝트 진행 중 |

### 6.2 DevOn이 Struts 기반인 이유

**시대적 배경:**
- **2000년대 중반** LG CNS가 엔터프라이즈 표준으로 Struts 채택
- 당시에는 **최선의 선택**이었음
- Struts의 구조적 일관성이 대규모 프로젝트에 적합
- MiPlatform 연동을 위해 Struts 기반으로 DevOn 개발

### 6.3 현재 개발자에게

> **"Struts 1.x는 왜 이렇게 만들었나요?"**
>
> → "그 시절에는 이게 **최선**이었습니다..."

**배워야 할 점:**
- ✅ MVC 패턴의 이해
- ✅ XML 기반 설정의 장단점
- ✅ 프레임워크 발전의 역사

**하지 말아야 할 것:**
- ❌ 새 프로젝트에 Struts 1.x 사용
- ❌ DevOn으로 신규 개발

---

## 7. 타임라인

```
2000년
├── Memorial Day 주말: Craig McClanahan이 Struts 개발 (3일)
└── 5월: Apache Software Foundation에 기부

2001년
└── 6월: Struts 1.0 공식 릴리스

2002년
└── 1월: Struts 1.1 릴리스

2004년
├── 6월: JavaServer Faces (JSF) 1.0 릴리스 (대항마 실패)
└── 12월: Struts 1.2 릴리스

2005년
├── Spring 2.0 릴리스 (Spring MVC 강화)
└── Struts 인기 절정

2006년
└── WebWork 2.2 릴리스 (나중에 Struts 2로)

2007년
└── 2월: Apache Struts 2.0 릴리스 (WebWork 기반)

2008년
├── 12월: Struts 1.3.10 (최종 릴리스)
└── Spring Framework 3.0 (어노테이션 지원)

2013년
└── 4월: Struts 1.x 공식 EOL (End of Life) 선언

현재 (2024년)
├── 레거시 시스템 유지보수 중
├── DevOn Framework (Struts 기반)도 레거시
└── Struts → Spring Boot 마이그레이션 진행 중
```

---

## 요약

| 항목 | 내용 |
|------|------|
| **만든 이유** | Servlet/JSP의 혼란을 MVC로 정리 |
| **혁신** | 자동 Form 바인딩, 선언적 설정, 태그 라이브러리 |
| **전성기** | 2000~2008년 (Java 웹 표준) |
| **인기 이유** | 구조적 일관성, 대규모 프로젝트 적합, 생산성 |
| **몰락** | Spring 등장, 어노테이션 선호, XML 복잡성 |
| **현재** | 2013년 EOL, 레거시 시스템 유지보수 중 |

### 핵심 메시지

Struts 1.x는 **2000년대 Java 웹 개발의 혁신**이었습니다.

- 당시에는 **최고의 프레임워크**
- MVC 패턴을 Java 웹에 **대중화**
- 수많은 기업의 **표준**
- 현재는 **레거시**이지만, 그 역사는 계속됨

**DevOn Framework 사용자에게:**
> DevOn이 Struts 1.x 기반인 이유는 단순합니다.
> 2000년대 중반에는 **Struts가 정답**이었기 때문입니다.
> 시간이 흘러 더 나은 기술이 등장했고, 이제는 **마이그레이션**의 시간입니다.

---

*"모든 기술에는 수명이 있다. 중요한 것은 그 기술이 남긴 교훈이다."*

**참고자료:**
- [Apache Struts 1 Wikipedia](https://en.wikipedia.org/wiki/Apache_Struts_1)
- [Struts 1 Reaches End of Life - InfoQ](https://www.infoq.com/news/2013/04/struts1-eol/)
- [Origin and Architecture of Struts](https://www.java-samples.com/showtutorial.php?tutorialid=622)
- [Craig McClanahan - Wikipedia](https://en.wikipedia.org/wiki/Craig_McClanahan)
