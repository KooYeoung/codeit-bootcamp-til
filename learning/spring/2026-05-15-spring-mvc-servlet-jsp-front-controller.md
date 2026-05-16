# Spring MVC 1편: Servlet, JSP, MVC 패턴과 Front Controller 정리

## 학습 날짜

2026-05-15

## 학습 주제

- 웹 애플리케이션 기본 구조
- 서블릿
- `HttpServletRequest`
- `HttpServletResponse`
- JSP
- MVC 패턴
- Front Controller 패턴
- 직접 MVC 프레임워크 만들기
- Spring MVC 구조
- `DispatcherServlet`
- `HandlerMapping`
- `HandlerAdapter`
- `ViewResolver`

---

## 1. 웹 애플리케이션 이해

웹 애플리케이션은 클라이언트가 HTTP 요청을 보내고, 서버가 요청을 처리한 뒤 HTTP 응답을 반환하는 구조이다.

웹 브라우저에서 URL을 입력하거나 버튼을 클릭하면 서버로 HTTP 요청이 전달된다.

서버는 요청을 분석하고 필요한 로직을 수행한 뒤 HTML, JSON, 파일 등 다양한 형태의 응답을 반환한다.

Java 백엔드 웹 개발에서는 이 HTTP 요청과 응답을 어떻게 편리하게 다룰 것인지가 중요하다.

처음에는 서블릿을 통해 HTTP 요청과 응답을 직접 다루고, 이후 JSP와 MVC 패턴을 통해 역할을 분리하며, 최종적으로 Spring MVC가 이런 구조를 체계적으로 제공한다고 이해할 수 있다.

---

## 2. 서블릿이란

서블릿은 Java에서 HTTP 요청과 응답을 다루기 위한 표준 기술이다.

개발자가 HTTP 요청 메시지를 직접 문자열로 파싱해서 처리하면 매우 불편하다.

서블릿은 HTTP 요청 메시지를 분석해서 `HttpServletRequest` 객체에 담아주고, 응답을 쉽게 만들 수 있도록 `HttpServletResponse` 객체를 제공한다.

서블릿을 사용하면 다음과 같은 작업을 할 수 있다.

- 요청 URL 확인
- HTTP 메서드 확인
- 쿼리 파라미터 조회
- HTML Form 데이터 조회
- HTTP 메시지 바디 읽기
- 응답 상태 코드 설정
- 응답 헤더 설정
- HTML 또는 JSON 응답 생성

서블릿은 HTTP를 Java 객체로 편하게 다룰 수 있게 해주는 기반 기술이라고 볼 수 있다.

---

## 3. HttpServletRequest

`HttpServletRequest`는 HTTP 요청 메시지를 편리하게 조회할 수 있게 해주는 객체이다.

HTTP 요청 메시지에는 다음 정보가 포함된다.

- HTTP 메서드
- 요청 URL
- 쿼리 스트링
- 프로토콜
- 헤더
- 쿠키
- 메시지 바디

예를 들어 다음과 같은 요청이 있다고 하자.

```http
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded

username=kim&age=20
```

이 요청을 직접 문자열로 파싱하지 않아도 `HttpServletRequest`를 사용하면 필요한 값을 쉽게 꺼낼 수 있다.

```java
String username = request.getParameter("username");
String age = request.getParameter("age");
```

또한 request 객체는 요청이 시작되고 끝날 때까지 유지되는 임시 저장소 역할도 할 수 있다.

```java
request.setAttribute("member", member);
Member member = (Member) request.getAttribute("member");
```

MVC 패턴에서는 이 request 저장소를 Model처럼 사용하기도 한다.

---

## 4. HttpServletResponse

`HttpServletResponse`는 HTTP 응답 메시지를 생성할 때 사용한다.

응답 메시지에는 다음 정보가 포함된다.

- HTTP 상태 코드
- 응답 헤더
- 응답 바디

HTML을 반환할 때는 다음처럼 Content-Type을 지정할 수 있다.

```java
response.setContentType("text/html");
response.setCharacterEncoding("utf-8");
```

그리고 `PrintWriter`를 통해 HTML을 직접 작성할 수 있다.

```java
PrintWriter writer = response.getWriter();
writer.println("<html>");
writer.println("<body>");
writer.println("<div>안녕?</div>");
writer.println("</body>");
writer.println("</html>");
```

JSON을 반환할 때는 `application/json`을 사용한다.

```java
response.setHeader("content-type", "application/json");
```

객체를 JSON 문자열로 바꾸기 위해 Jackson의 `ObjectMapper`를 사용할 수 있다.

```java
String result = objectMapper.writeValueAsString(data);
response.getWriter().write(result);
```

서블릿은 HTTP 응답을 직접 제어할 수 있다는 장점이 있지만, HTML을 Java 코드로 직접 작성하면 코드가 매우 복잡해진다.

---

## 5. 서블릿으로 회원 관리 웹 애플리케이션 만들기

회원 관리 예제에서는 다음 요구사항을 다룬다.

- 회원 저장
- 회원 목록 조회

회원 도메인은 `id`, `username`, `age`를 가진다.

회원 저장소는 메모리 기반으로 구성하고, 싱글톤 패턴을 적용했다.

```java
private static final MemberRepository instance = new MemberRepository();

public static MemberRepository getInstance() {
    return instance;
}

private MemberRepository() {
}
```

스프링을 사용하면 스프링 빈으로 객체를 관리할 수 있지만, 순수 서블릿 예제에서는 직접 싱글톤 패턴을 적용했다.

서블릿으로 회원 등록 폼을 만들면 Java 코드 안에서 HTML을 직접 작성해야 한다.

```java
w.write("<form action=\"/servlet/members/save\" method=\"post\">");
w.write("username: <input type=\"text\" name=\"username\" />");
w.write("age: <input type=\"text\" name=\"age\" />");
w.write("</form>");
```

이 방식은 동작은 하지만, HTML이 길어질수록 유지보수가 매우 어렵다.

화면 수정이 필요할 때 Java 코드를 수정해야 하고, 디자이너나 퍼블리셔가 작업하기도 어렵다.

---

## 6. JSP가 필요한 이유

JSP는 HTML 안에 Java 코드를 넣어 동적인 화면을 만들 수 있게 해준다.

서블릿처럼 Java 코드 안에서 HTML 문자열을 만드는 것보다 JSP가 화면 작성에는 더 편하다.

하지만 JSP에 회원 저장, 회원 조회 같은 비즈니스 로직이 들어가면 문제가 생긴다.

JSP 하나가 다음 역할을 모두 가지게 된다.

- 요청 파라미터 조회
- 비즈니스 로직 호출
- Model 데이터 준비
- HTML 렌더링

이렇게 되면 화면 코드와 Java 로직이 섞여 유지보수가 어려워진다.

즉, JSP는 View 역할에는 적합하지만 Controller 역할까지 담당하면 코드가 복잡해진다.

---

## 7. MVC 패턴

MVC 패턴은 애플리케이션을 세 가지 역할로 나누는 구조이다.

- Model
- View
- Controller

## Controller

Controller는 HTTP 요청을 받고 필요한 비즈니스 로직을 호출한다.

그리고 View에 전달할 데이터를 Model에 담는다.

서블릿은 MVC 패턴에서 Controller 역할을 할 수 있다.

## Model

Model은 View에 전달할 데이터를 담는 역할을 한다.

순수 서블릿 기반 MVC 예제에서는 `HttpServletRequest`의 저장소 기능을 Model처럼 사용한다.

```java
request.setAttribute("member", member);
```

## View

View는 화면을 렌더링하는 역할을 한다.

JSP는 MVC 패턴에서 View 역할을 담당한다.

MVC 패턴을 적용하면 Controller는 요청 처리와 흐름 제어에 집중하고, JSP는 화면 출력에 집중할 수 있다.

---

## 8. forward와 redirect

MVC 패턴을 이해하려면 `forward`와 `redirect` 차이를 알아야 한다.

## forward

`forward`는 서버 내부에서 다른 서블릿이나 JSP로 이동하는 방식이다.

```java
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```

클라이언트는 서버 내부에서 이동이 발생한 것을 알 수 없다.

브라우저 URL도 변경되지 않는다.

Controller에서 JSP로 이동할 때 주로 사용한다.

## redirect

`redirect`는 서버가 클라이언트에게 다시 요청하라고 응답하는 방식이다.

클라이언트는 redirect 경로로 새로운 요청을 보낸다.

브라우저 URL도 변경된다.

등록, 수정, 삭제 같은 POST 요청 이후 새로고침 문제를 막기 위해 자주 사용한다.

---

## 9. /WEB-INF 경로

JSP를 `/WEB-INF` 아래에 두면 외부에서 직접 JSP를 호출할 수 없다.

```text
/WEB-INF/views/new-form.jsp
```

이렇게 하면 반드시 Controller를 통해서만 JSP가 호출된다.

이 구조는 MVC 패턴에서 중요하다.

사용자가 View를 직접 호출하지 못하게 하고, 항상 Controller를 거쳐 필요한 Model 데이터를 준비한 뒤 View를 렌더링하도록 만들 수 있다.

---

## 10. MVC 패턴의 한계

MVC 패턴을 적용하면 서블릿과 JSP만 사용하는 것보다 구조가 좋아진다.

하지만 여전히 한계가 있다.

각 Controller마다 다음 코드가 반복된다.

- View로 forward하는 코드
- View 경로 작성
- request, response 사용
- 공통 처리 로직

예를 들어 각 Controller에서 다음 코드가 반복될 수 있다.

```java
String viewPath = "/WEB-INF/views/members.jsp";
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```

컨트롤러가 많아질수록 중복 코드가 늘어난다.

이 문제를 해결하기 위해 Front Controller 패턴을 도입한다.

---

## 11. Front Controller 패턴

Front Controller 패턴은 모든 요청을 하나의 컨트롤러에서 먼저 받는 구조이다.

기존 구조에서는 각 URL마다 개별 Controller가 직접 호출되었다.

Front Controller를 도입하면 클라이언트 요청은 먼저 Front Controller로 들어오고, Front Controller가 요청 URL에 맞는 실제 Controller를 찾아 호출한다.

Front Controller의 장점은 다음과 같다.

- 요청의 입구를 하나로 만들 수 있다.
- 공통 처리를 한 곳에 모을 수 있다.
- 개별 Controller는 서블릿을 직접 사용하지 않아도 된다.
- 컨트롤러 호출 방식을 일관되게 만들 수 있다.

Spring MVC의 핵심인 `DispatcherServlet`도 Front Controller 패턴으로 구현되어 있다.

---

## 12. 직접 MVC 프레임워크 만들기 흐름

직접 MVC 프레임워크를 만드는 과정은 v1부터 v5까지 단계적으로 발전한다.

## v1: Front Controller 도입

Front Controller가 요청을 받고, URL에 맞는 Controller를 찾아 호출한다.

각 Controller는 서블릿과 비슷하게 `HttpServletRequest`, `HttpServletResponse`를 직접 사용한다.

## v2: View 분리

각 Controller에서 반복되던 View 이동 로직을 별도의 View 객체로 분리한다.

이렇게 하면 Controller는 View 이름이나 View 객체를 반환하고, 실제 렌더링은 View가 담당하게 된다.

## v3: Model 추가

Controller가 서블릿 기술에 직접 의존하지 않도록 Model을 별도로 만든다.

request 객체 대신 ModelView 같은 객체를 사용해 View 이름과 Model 데이터를 함께 반환한다.

## v4: 실용적인 Controller

Controller가 ModelView를 직접 생성하지 않고, Model만 전달받아 View 이름을 반환하도록 단순화한다.

개발자가 작성해야 할 코드가 줄어든다.

## v5: HandlerAdapter 도입

서로 다른 방식의 Controller를 유연하게 처리하기 위해 Adapter를 도입한다.

HandlerAdapter가 있으면 Front Controller는 다양한 컨트롤러 호출 방식을 통일된 구조로 처리할 수 있다.

---

## 13. Spring MVC 구조

직접 만든 MVC 프레임워크와 Spring MVC는 다음처럼 대응된다.

| 직접 만든 MVC 프레임워크 | Spring MVC |
| --- | --- |
| FrontController | DispatcherServlet |
| handlerMappingMap | HandlerMapping |
| MyHandlerAdapter | HandlerAdapter |
| ModelView | ModelAndView |
| viewResolver | ViewResolver |
| MyView | View |

Spring MVC도 Front Controller 패턴을 기반으로 한다.

Spring MVC의 Front Controller가 바로 `DispatcherServlet`이다.

---

## 14. DispatcherServlet

`DispatcherServlet`은 Spring MVC의 핵심이다.

스프링 부트는 `DispatcherServlet`을 자동으로 서블릿에 등록하고, 기본적으로 모든 경로에 대해 매핑한다.

요청이 들어오면 `DispatcherServlet`은 다음 순서로 동작한다.

1. 요청 URL에 맞는 핸들러를 조회한다.
2. 핸들러를 실행할 수 있는 핸들러 어댑터를 찾는다.
3. 핸들러 어댑터를 실행한다.
4. 핸들러 어댑터가 실제 컨트롤러를 호출한다.
5. 컨트롤러 실행 결과로 `ModelAndView`를 반환한다.
6. ViewResolver를 통해 View를 찾는다.
7. View를 반환받는다.
8. View를 렌더링한다.

이 흐름을 이해하면 Spring MVC가 내부에서 어떤 방식으로 컨트롤러를 호출하고 View를 렌더링하는지 파악할 수 있다.

---

## 15. HandlerMapping과 HandlerAdapter

## HandlerMapping

`HandlerMapping`은 요청 URL에 맞는 핸들러, 즉 컨트롤러를 찾아주는 역할을 한다.

예를 들어 `/springmvc/members/new-form` 요청이 들어오면 이 요청을 처리할 컨트롤러를 찾는다.

## HandlerAdapter

`HandlerAdapter`는 찾은 핸들러를 실제로 실행할 수 있게 해주는 어댑터이다.

Spring MVC에는 다양한 방식의 컨트롤러가 존재할 수 있다.

핸들러 어댑터는 이 다양한 컨트롤러 호출 방식을 통일해서 `DispatcherServlet`이 처리할 수 있게 해준다.

즉, `DispatcherServlet`은 직접 컨트롤러를 호출하지 않고, HandlerAdapter를 통해 컨트롤러를 실행한다.

---

## 16. ViewResolver

컨트롤러가 논리 View 이름을 반환하면, ViewResolver가 실제 View 경로를 찾아준다.

예를 들어 컨트롤러가 다음과 같이 반환한다고 하자.

```java
return "members";
```

ViewResolver는 prefix와 suffix를 붙여 실제 경로를 만든다.

```text
/WEB-INF/views/members.jsp
```

이렇게 하면 컨트롤러는 실제 JSP 경로를 몰라도 되고, 논리적인 View 이름만 반환하면 된다.

---

## 17. 오늘 정리

오늘 학습한 핵심 흐름은 다음과 같다.

처음에는 서블릿으로 HTTP 요청과 응답을 직접 처리했다.

하지만 서블릿만 사용하면 HTML을 Java 코드로 직접 작성해야 해서 불편했다.

JSP를 사용하면 화면 작성은 편해졌지만, JSP 안에 비즈니스 로직이 섞이는 문제가 생겼다.

MVC 패턴은 이 문제를 해결하기 위해 Controller, Model, View의 역할을 나누었다.

하지만 Controller마다 공통 코드가 반복되었고, 이를 해결하기 위해 Front Controller 패턴이 등장했다.

Spring MVC는 이 Front Controller 패턴을 `DispatcherServlet`으로 구현했다.

따라서 Spring MVC를 제대로 이해하려면 단순히 애노테이션 사용법만 외우는 것이 아니라, 서블릿 → JSP → MVC → Front Controller → DispatcherServlet로 이어지는 발전 흐름을 이해하는 것이 중요하다.
