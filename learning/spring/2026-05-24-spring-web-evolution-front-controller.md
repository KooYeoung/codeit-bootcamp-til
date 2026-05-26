# Spring Web 변천사와 Front Controller 패턴 정리

## 학습 날짜

2026-05-24

## 학습 주제

- Servlet 방식
- JSP와 Model 1 방식
- Model 2 MVC 방식
- Front Controller 패턴
- Spring MVC로 발전한 구조

---

## 1. Servlet 방식

Servlet 방식은 Java 클래스로 HTTP 요청과 응답을 직접 처리하는 초기 웹 개발 방식이다.

Servlet은 Servlet Container 안에서 실행되며, 요청이 들어오면 `service()` 메서드를 통해 HTTP 메서드에 맞는 처리 로직이 실행된다.

초기 Servlet 방식에서는 Java 코드 안에서 비즈니스 로직뿐 아니라 HTML 응답까지 직접 작성했다.

```java
response.getWriter().println("<html><body>");
response.getWriter().println("<h1>Hello</h1>");
response.getWriter().println("</body></html>");
```

이 방식은 요청과 응답을 세밀하게 제어할 수 있다는 장점이 있다.

하지만 HTML 코드와 Java 로직이 한 클래스 안에 섞이기 때문에 코드가 복잡해지고 유지보수가 어려워진다.

---

## 2. JSP와 Model 1 방식

JSP는 HTML 안에 Java 코드를 작성할 수 있게 만든 기술이다.

Servlet 방식에서는 Java 코드 안에 HTML을 작성했다면, JSP는 반대로 HTML 안에 Java 코드를 넣는 방식이다.

Model 1 방식에서는 요청이 JSP로 바로 들어오고, JSP가 화면 출력과 요청 처리를 모두 담당한다.

```jsp
<%
String name = request.getParameter("name");
%>
<h1>Hello, <%= name %></h1>
```

JSP는 화면 작성은 편하게 해주었지만, 여전히 화면 코드와 비즈니스 로직이 섞이는 문제가 있었다.

즉, Servlet 방식은 Java 안에 HTML이 섞였고, JSP Model 1 방식은 HTML 안에 Java가 섞였다.

두 방식 모두 역할 분리가 부족했다.

---

## 3. Model 2 MVC 방식

Model 2 방식은 Servlet과 JSP를 함께 사용해 역할을 분리한다.

- Servlet: Controller 역할
- JSP: View 역할
- Java Bean 또는 Model 객체: 데이터 역할

요청은 Servlet으로 들어오고, Servlet은 필요한 비즈니스 로직을 처리한 뒤 결과를 Model에 담는다.

이후 JSP로 forward하여 화면을 렌더링한다.

```java
request.setAttribute("message", message);
request.getRequestDispatcher("/WEB-INF/jsp/hello.jsp")
       .forward(request, response);
```

JSP를 `WEB-INF` 아래에 두면 사용자가 URL로 JSP에 직접 접근할 수 없다.

이렇게 하면 요청 진입점은 Controller인 Servlet이 되고, JSP는 View 역할에 집중할 수 있다.

---

## 4. Model 2 방식의 한계

Model 2 방식은 Servlet과 JSP의 역할을 분리했다는 점에서 발전된 구조이다.

하지만 여러 Controller Servlet이 존재하면 공통 기능이 반복될 수 있다.

예를 들어 다음 기능은 대부분의 Controller에서 공통으로 필요할 수 있다.

- 인증 확인
- 권한 확인
- 요청 로그
- 예외 처리
- 인코딩 처리
- 공통 Model 처리

각 Controller가 이런 공통 기능을 직접 처리하면 중복 코드가 많아지고 수정도 어려워진다.

이 문제를 해결하기 위해 Front Controller 패턴이 등장한다.

---

## 5. Front Controller 패턴

Front Controller 패턴은 모든 요청을 하나의 공통 진입점으로 모으는 구조이다.

```text
Client → FrontController → Controller → Model → View
```

Front Controller는 요청을 먼저 받고, 공통 기능을 처리한 뒤 실제 요청을 처리할 Controller로 위임한다.

장점은 다음과 같다.

- 모든 요청의 진입점이 하나로 통일된다.
- 공통 기능을 중앙에서 처리할 수 있다.
- Controller마다 반복되던 중복 코드가 줄어든다.
- 요청 처리 흐름을 일관성 있게 관리할 수 있다.
- 새로운 Controller를 추가하더라도 공통 처리 구조는 유지된다.

---

## 6. Spring MVC와 Front Controller

Spring MVC는 Front Controller 패턴을 기반으로 만들어졌다.

Spring MVC에서 Front Controller 역할을 하는 핵심 객체는 `DispatcherServlet`이다.

```text
Client → DispatcherServlet → HandlerMapping → HandlerAdapter → Controller
```

DispatcherServlet은 모든 요청을 직접 처리하지 않는다.

대신 여러 전략 객체와 협력한다.

- HandlerMapping: 요청에 맞는 Handler를 찾는다.
- HandlerAdapter: 찾은 Handler를 실행한다.
- ViewResolver: 논리 View 이름을 실제 View로 변환한다.
- View: 최종 화면을 렌더링한다.

즉, DispatcherServlet은 요청 처리 전체 흐름을 조정하는 중앙 관리자 역할을 한다.

---

## 7. 보충 정리

Servlet, JSP, Model 2, Spring MVC의 차이는 역할 분리 정도로 이해하면 좋다.

| 방식 | 요청 처리 | 화면 처리 | 문제점 |
|---|---|---|---|
| Servlet | Java 클래스 | Java 코드에서 HTML 직접 작성 | Java와 HTML 혼재 |
| JSP Model 1 | JSP | JSP | HTML과 Java 로직 혼재 |
| Model 2 MVC | Servlet | JSP | Controller마다 공통 기능 중복 |
| Spring MVC | DispatcherServlet + Controller | View | 구조 이해 필요 |

Spring MVC는 단순히 Controller 애노테이션을 편하게 쓰는 기술이 아니다.

Front Controller 패턴을 기반으로 요청 처리, 공통 기능, View 렌더링을 체계적으로 분리한 프레임워크이다.

---

## 8. 오늘 정리

자바 웹 기술은 Servlet에서 시작해 JSP, Model 2 MVC, Front Controller, Spring MVC 구조로 발전했다.

이 흐름의 핵심은 역할 분리와 중복 제거이다.

중요한 정리는 다음과 같다.

- Servlet은 Java 코드에서 HTTP 요청과 응답을 직접 처리한다.
- JSP는 HTML 안에서 Java 코드를 작성할 수 있게 해주었다.
- Model 1 방식은 JSP가 요청 처리와 화면 출력을 모두 담당한다.
- Model 2 방식은 Servlet이 Controller, JSP가 View 역할을 맡는다.
- Front Controller 패턴은 모든 요청의 공통 진입점을 만든다.
- Spring MVC의 Front Controller는 DispatcherServlet이다.
- DispatcherServlet은 여러 전략 객체와 협력해 요청을 처리한다.

앞으로 Spring MVC를 학습할 때는 `@Controller` 사용법만 보는 것이 아니라, DispatcherServlet을 중심으로 요청이 어떻게 흐르는지 함께 봐야겠다.
