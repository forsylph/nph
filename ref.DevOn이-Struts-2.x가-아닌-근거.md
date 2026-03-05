# DevOn이 Struts 2.x가 아닌 근거

> DevOn Framework는 Struts 1.x 기반이며, Struts 2.x와는 완전히 다른 아키텍처입니다.

---

## 핵심 결론

**DevOn = Struts 1.x + MiPlatform 연동**

DevOn Framework는 **Struts 1.x의 아키텍처를 그대로 따르고 있으며**, Struts 2.x와는 다음의 결정적인 차이점이 있습니다:

| 구분 | Struts 1.x / DevOn | Struts 2.x |
|------|-------------------|------------|
| **출시 시기** | 2001년 (Struts 1.x) | 2007년 2월 |
| **기반 기술** | Servlet 기반 | **Filter 기반** (WebWork) |
| **Front Controller** | **Servlet** (`ActionServlet` / `MiplatformServlet`) | **Filter** (`FilterDispatcher`) |
| **Action/Command** | **상속 필수** (`extends Action` / `extends AbstractMiplatformCommand`) | **POJO** (상속 불필요) |
| **설정 방식** | **XML 중심** | 어노테이션 + XML |
| **타입 변환** | 수동 (`ActionForm`) | 자동 (OGNL) |
| **의존성 주입** | 없음 | **내장 DI** |

---

## 근거 1: Front Controller의 종류가 다름

### Struts 1.x / DevOn: Servlet 기반

```java
// web.xml - Struts 1.x
<servlet>
    <servlet-name>action</servlet-name>
    <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>action</servlet-name>
    <url-pattern>*.do</url-pattern>
</servlet-mapping>

// web.xml - DevOn
<servlet>
    <servlet-name>MiPlatformChannelServlet</servlet-name>
    <servlet-class>devonx.nph.system.servlet.MiplatformServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>MiPlatformChannelServlet</servlet-name>
    <url-pattern>*.mhi</url-pattern>
</servlet-mapping>
```

**특징:**
- `HttpServlet`을 상속받은 **Servlet**이 모든 요청의 입구
- `doPost()` / `doGet()` 메소드로 요청 처리
- J2EE 표준 Servlet API 사용

### Struts 2.x: Filter 기반 (결정적 차이)

```java
// web.xml - Struts 2.x
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

**특징:**
- **`Filter`**를 사용하여 요청 처리
- Servlet이 아닌 Filter Chain에서 동작
- 더 유연한 요청 처리 (이미지, 정적 리소스 등)

**근거:**
> DevOn의 `MiplatformServlet`은 **Servlet**입니다. Struts 2.x는 **Filter** 기반입니다. 이는 완전히 다른 아키텍처입니다.

---

## 근거 2: Action/Command의 상속 구조

### Struts 1.x / DevOn: 상속 필수

```java
// Struts 1.x
public class LoginAction extends Action {
    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form, ...) {
        // 비즈니스 로직
        return mapping.findForward("success");
    }
}

// DevOn
public class LoginUserCMD extends AbstractMiplatformCommand {
    @Override
    public void execute() throws Exception {
        // 비즈니스 로직
        platformResponse.addDataset("ds_Result", result);
    }
}
```

**특징:**
- **반드시 특정 클래스를 상속**해야 함
- 프레임워크가 제공하는 라이프사이클 메소드 사용
- 테스트 어려움 (컨테이너 의존)

### Struts 2.x: POJO (Plain Old Java Object)

```java
// Struts 2.x - 상속 불필요!
public class LoginAction {
    private String username;
    private String password;

    // POJO - 아무 클래스나 가능
    public String execute() {
        // 비즈니스 로직
        return "success";
    }

    // Getter/Setter만 있으면 됨
    public void setUsername(String username) { this.username = username; }
    public String getUsername() { return username; }
}
```

**특징:**
- **어떤 클래스도 Action이 될 수 있음**
- 상속 불필요
- 단위 테스트 용이

**근거:**
> DevOn의 Command는 **`AbstractMiplatformCommand`를 반드시 상속**합니다. 이는 Struts 1.x의 `extends Action`과 동일한 패턴입니다.

---

## 근거 3: 설정 방식의 차이

### Struts 1.x / DevOn: XML 전용

```xml
<!-- Struts 1.x - struts-config.xml -->
<action path="/login" type="com.example.LoginAction" name="loginForm">
    <forward name="success" path="/welcome.jsp"/>
</action>

<!-- DevOn - navigation/*.xml -->
<action name="LoginUser">
    <command>nph.his.az.auth.cmd.LoginUserCMD</command>
    <interceptor>defaultStack</interceptor>
</action>
```

**특징:**
- **XML에 모든 설정**을 정의
- 어노테이션 사용 없음
- 설정 파일이 방대해짐

### Struts 2.x: 어노테이션 지원

```java
// Struts 2.x - 어노테이션 사용 가능
@Namespace("/user")
@Action("login")
@Result(name="success", location="/welcome.jsp")
public class LoginAction {
    @RequiredFieldValidator
    private String username;

    public String execute() {
        return "success";
    }
}
```

**특징:**
- **어노테이션**으로 설정 가능
- Convention over Configuration
- XML 없이도 동작 가능

**근거:**
> DevOn은 **어노테이션을 전혀 사용하지 않고** XML로만 설정합니다. Struts 2.x는 어노테이션을 기본적으로 지원합니다.

---

## 근거 4: 탄생 시기와 역사적 맥락

### 타임라인 비교

```
2000년: Craig McClanahan이 Struts 1.x 개발 (3일)
       ↓
2001년: Struts 1.0 릴리스
       ↓
2000년대 중반: LG CNS가 DevOn Framework 개발 시작
       │        (Struts 1.x를 기반으로 MiPlatform 연동 추가)
       ↓
2006년: WebWork 2.2 릴리스 (Struts 2의 기반)
       ↓
2007년 2월: **Struts 2.0 릴리스** (WebWork 기반)
       ↓
2008년: DevOn Framework 3.x ~ 4.x 사용 (NPH 프로젝트)
       ↓
2013년: Struts 1.x EOL (지원 종료)
```

**시간적 모순:**
- DevOn은 **2000년대 중반**에 개발 시작
- Struts 2.x는 **2007년**에 출시
- DevOn 개발 시점에 Struts 2.x는 **존재하지 않았음**

**근거:**
> DevOn이 개발될 때(2000년대 중반) Struts 2.x는 **아직 출시되지 않았습니다**. Struts 2.x는 2007년에 WebWork를 기반으로 새로 개발되었습니다.

---

## 근거 5: WebWork 기반 vs Servlet 기반

### Struts 2.x는 WebWork의 후계자

Struts 2.x는 **Struts 1.x를 계승한 것이 아니라**, 완전히 다른 프레임워크인 **WebWork 2.2**를 기반으로 개발되었습니다.

```
Struts 1.x (Apache) ──→ 2013년 EOL
       │
       │ (완전히 다른 계열)
       │
WebWork (OpenSymphony) ──→ 2006년 WebWork 2.2
       │
       ↓ (흡수)
Struts 2.x (Apache) ──→ 2007년 출시
```

**WebWork의 특징:**
- Filter 기반 아키텍처
- OGNL (Object-Graph Navigation Language) 사용
- Interceptor 패턴
- POJO Action

**DevOn의 특징:**
- Servlet 기반 아키텍처
- OGNL 사용 없음
- Interceptor Chain 있음 (Struts 1.x의 RequestProcessor와 유사)
- 상속 기반 Command

**근거:**
> DevOn은 **WebWork와 무관**합니다. DevOn은 Servlet 기반의 전통적인 MVC 패턴을 따르며, Struts 1.x의 구조를 그대로 재현하고 있습니다.

---

## 근거 6: ActionForm vs ModelDriven

### Struts 1.x / DevOn: Form 데이터 바인딩

```java
// Struts 1.x - ActionForm
public class LoginForm extends ActionForm {
    private String username;
    private String password;
    // Getter/Setter
}

// DevOn - Dataset (MiPlatform)
public class LoginUserCMD extends AbstractMiplatformCommand {
    public void execute() {
        LData input = getDatasetWithJobType("ds_Input"); // Form 데이터
    }
}
```

**특징:**
- 별도의 Form 객체가 데이터를 담음
- 수동 데이터 바인딩

### Struts 2.x: ModelDriven

```java
// Struts 2.x - ModelDriven
public class LoginAction implements ModelDriven<User> {
    private User user = new User();

    @Override
    public User getModel() {
        return user;
    }

    // OGNL이 자동으로 파라미터 바인딩
    public String execute() {
        // user.username, user.password가 자동 채워짐
        return "success";
    }
}
```

**특징:**
- OGNL을 통한 자동 데이터 바인딩
- ModelDriven 인터페이스

**근거:**
> DevOn은 **OGNL을 사용하지 않습니다**. Struts 2.x의 핵심인 OGNL 기반 데이터 바인딩이 DevOn에는 없습니다.

---

## 근거 7: 결과 반환 방식

### Struts 1.x / DevOn: Forward/Response 기반

```java
// Struts 1.x
return mapping.findForward("success"); // ActionForward

// DevOn
platformResponse.addDataset("ds_Result", result); // Dataset에 담기
```

**특징:**
- 결과를 반환하거나 Response에 담음
- 페이지 이동 개념

### Struts 2.x: Result 타입

```java
// Struts 2.x - 다양한 Result 타입
@Result(name="success", type="dispatcher", location="/welcome.jsp")
@Result(name="json", type="json")
@Result(name="redirect", type="redirect", location="/home")
```

**특징:**
- Result 타입 개념 (dispatcher, redirect, json, stream 등)
- 더 유연한 결과 처리

**근거:**
> DevOn은 **ActionForward나 Result 타입이 아닌**, MiPlatform의 Dataset에 직접 결과를 담는 방식을 사용합니다. 이는 Struts 1.x의 단순한 포워딩 개념과 유사합니다.

---

## 요약: 판단 근거

| 근거 | Struts 1.x / DevOn | Struts 2.x |
|------|-------------------|------------|
| **Front Controller** | ✅ **Servlet** (`MiplatformServlet`) | Filter (`FilterDispatcher`) |
| **Action 상속** | ✅ **필수** (`extends AbstractMiplatformCommand`) | 불필요 (POJO) |
| **설정 방식** | ✅ **XML 전용** | 어노테이션 + XML |
| **출시 시기** | ✅ DevOn 개발 시 **존재하지 않음** | 2007년 출시 |
| **기반 기술** | ✅ **Servlet API** | WebWork (Filter) |
| **데이터 바인딩** | ✅ 수동 (Dataset) | OGNL 자동 바인딩 |
| **View 기술** | ✅ MiPlatform (XML) | JSP, FreeMarker, Velocity |

---

## 핵심 메시지

### DevOn은 Struts 1.x 시대의 프레임워크

```
DevOn Framework
├── 기반: Apache Struts 1.x 아키텍처
├── 특징: Servlet 기반, XML 설정, 상속 필수
├── 개발 시기: 2000년대 중반 (Struts 2.x 이전)
└── MiPlatform 연동을 위해 Struts 1.x 구조 채택

Struts 2.x
├── 기반: WebWork 2.2 (Struts 1.x와 무관)
├── 특징: Filter 기반, 어노테이션, POJO
├── 출시: 2007년 (DevOn 개발 이후)
└── 완전히 다른 아키텍처
```

**결론:**
> DevOn Framework는 **Struts 2.x가 아닌 Struts 1.x의 아키텍처를 정확하게 따르고 있습니다**. DevOn이 개발될 때 Struts 2.x는 아직 존재하지 않았으며, Struts 1.x의 구조가 엔터프라이즈 시스템 개발에 적합하다고 판단하여 이를 채택한 것입니다.

---

*참고: Struts 1.x는 2013년 EOL(End of Life)되었으며, DevOn도 유사한 레거시 프레임워크로 분류됩니다.*
