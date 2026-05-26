# Servlet, HttpServletRequest, HttpServletResponse, Servlet Container 정리

## 학습 날짜

2026-05-24

## 학습 주제

- Servlet 개념
- Servlet 생명주기
- Servlet Container
- HttpServletRequest
- HttpServletResponse
- 요청 데이터 처리
- 응답 데이터 처리

---

## 1. Servlet이란

Servlet은 서버에서 실행되는 Java 기반 웹 컴포넌트이다.

클라이언트가 HTTP 요청을 보내면 Servlet Container가 해당 요청에 맞는 Servlet을 찾아 실행한다.

Servlet은 HTTP 요청을 처리하고, 응답을 만들어 클라이언트에게 반환한다.

Servlet을 직접 구현할 때는 보통 `HttpServlet`을 상속한다.

```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) {
        // GET 요청 처리
    }
}
```

---

## 2. Servlet 생명주기

Servlet은 Servlet Container가 관리한다.

생명주기는 크게 세 단계로 볼 수 있다.

```text
생성 및 초기화 → 요청 처리 → 종료
```

## init()

Servlet 객체가 생성된 후 최초 한 번 호출된다.

초기화 파라미터를 읽거나, 필요한 리소스를 준비할 때 사용할 수 있다.

## service()

요청마다 호출된다.

`service()`는 HTTP 메서드에 따라 `doGet()`, `doPost()`, `doPut()`, `doDelete()` 같은 메서드로 분기한다.

## destroy()

Servlet이 종료될 때 한 번 호출된다.

열어둔 리소스를 정리할 때 사용할 수 있다.

---

## 3. Servlet 로딩 방식

Servlet은 지연 로딩 또는 즉시 로딩될 수 있다.

## 지연 로딩

첫 번째 요청이 들어올 때 Servlet 객체를 생성한다.

별도 설정이 없으면 일반적으로 지연 로딩으로 동작한다.

## 즉시 로딩

애플리케이션 시작 시점에 Servlet 객체를 미리 생성한다.

`loadOnStartup` 또는 `web.xml`의 `<load-on-startup>` 설정으로 제어할 수 있다.

즉시 준비되어야 하는 Servlet은 애플리케이션 시작 시점에 로딩하도록 설정할 수 있다.

---

## 4. Servlet Container

Servlet Container는 Servlet의 생명주기를 관리하고 HTTP 요청과 응답을 처리하는 실행 환경이다.

대표적으로 Tomcat, Jetty, JBoss, Jeus 등이 있다.

Servlet Container의 역할은 다음과 같다.

- HTTP 요청 수신
- 요청 파싱
- URL과 Servlet 매핑
- `HttpServletRequest`, `HttpServletResponse` 생성
- Servlet 실행
- 응답 전송
- Servlet 생명주기 관리

Spring Boot에서 사용하는 내장 Tomcat도 Servlet Container이다.

---

## 5. HttpServletRequest

`HttpServletRequest`는 클라이언트 요청 정보를 담고 있는 객체이다.

Servlet Container는 HTTP 요청을 분석해서 요청 정보를 객체에 담고 Servlet에 전달한다.

주요 정보는 다음과 같다.

- HTTP 메서드
- 요청 URI와 URL
- Query String
- Header
- Cookie
- Session
- 요청 파라미터
- 요청 본문
- 원격/로컬 주소 정보

예시는 다음과 같다.

```java
request.getMethod();
request.getRequestURI();
request.getHeader("User-Agent");
request.getParameter("username");
request.getSession(false);
```

---

## 6. 요청 파라미터 처리

URL 쿼리 파라미터와 HTML Form 데이터는 `getParameter()` 계열 API로 읽을 수 있다.

```java
String name = request.getParameter("name");
String[] hobbies = request.getParameterValues("hobby");
Map<String, String[]> parameterMap = request.getParameterMap();
```

GET 요청의 쿼리 파라미터는 URL에 노출된다.

따라서 비밀번호나 개인정보 같은 민감한 값은 URL 쿼리 파라미터로 전달하면 안 된다.

POST Form 요청은 보통 `application/x-www-form-urlencoded` 형식으로 요청 본문에 데이터가 담긴다.

하지만 Servlet API에서는 Form 데이터도 `getParameter()`로 동일하게 읽을 수 있다.

---

## 7. REST API 요청 본문 처리

JSON이나 Plain Text처럼 요청 본문에 직접 담긴 데이터는 `getParameter()`로 처리하지 않는다.

본문을 직접 읽어야 한다.

문자 데이터는 `getReader()`를 사용할 수 있다.

```java
BufferedReader reader = request.getReader();
```

바이너리 데이터는 `getInputStream()`을 사용할 수 있다.

```java
ServletInputStream inputStream = request.getInputStream();
```

JSON은 Jackson의 `ObjectMapper` 등을 사용해 Java 객체나 Map으로 변환할 수 있다.

```java
ObjectMapper objectMapper = new ObjectMapper();
Map<String, Object> body = objectMapper.readValue(request.getReader(), Map.class);
```

Spring MVC에서는 이런 작업을 `@RequestBody`와 `HttpMessageConverter`가 대신 처리해준다.

---

## 8. HttpServletResponse

`HttpServletResponse`는 서버가 클라이언트에게 보낼 응답 정보를 설정하는 객체이다.

주요 기능은 다음과 같다.

- 상태 코드 설정
- 응답 헤더 설정
- Content-Type 설정
- 응답 본문 작성
- Redirect 처리
- Cookie 추가
- Error 응답 전송

예시는 다음과 같다.

```java
response.setStatus(HttpServletResponse.SC_OK);
response.setHeader("X-Custom-Header", "value");
response.setContentType("application/json");
response.getWriter().write("hello");
response.sendRedirect("/home");
```

---

## 9. setStatus와 sendError 차이

`setStatus()`는 응답 상태 코드만 설정한다.

```java
response.setStatus(HttpServletResponse.SC_ACCEPTED);
```

반면 `sendError()`는 오류 응답을 전송하기 위한 메서드이다.

```java
response.sendError(HttpServletResponse.SC_BAD_REQUEST, "Invalid request");
```

`sendError()`가 실행되면 오류 응답 처리가 시작되고, 이후 응답을 수정하기 어렵다.

정상 흐름에서 단순 상태 코드만 지정하려면 `setStatus()`를 사용하고, 오류 상황을 명확히 전달하려면 `sendError()`를 사용한다.

---

## 10. 보충 정리

Servlet을 학습하는 이유는 Spring MVC의 내부 구조를 이해하기 위해서이다.

Spring MVC를 사용하면 `HttpServletRequest`, `HttpServletResponse`를 직접 다루지 않아도 되는 경우가 많다.

하지만 내부적으로는 여전히 Servlet Container 위에서 동작한다.

Spring MVC의 `@RequestParam`, `@ModelAttribute`, `@RequestBody`, `ResponseEntity` 같은 기능도 결국 Servlet 요청과 응답을 더 편리하게 다루기 위한 추상화이다.

---

## 11. 오늘 정리

Servlet은 Spring MVC의 기반 기술이다.

중요한 정리는 다음과 같다.

- Servlet은 Servlet Container 안에서 실행된다.
- Servlet 생명주기는 `init()`, `service()`, `destroy()`로 이해할 수 있다.
- 요청마다 `service()`가 호출된다.
- `HttpServletRequest`는 요청 정보를 읽는 객체이다.
- `HttpServletResponse`는 응답 정보를 설정하는 객체이다.
- 쿼리 파라미터와 Form 데이터는 `getParameter()`로 읽을 수 있다.
- JSON 요청 본문은 `getReader()` 또는 `getInputStream()`으로 읽어야 한다.
- Spring MVC는 이런 Servlet API 사용을 더 편리하게 추상화한다.
