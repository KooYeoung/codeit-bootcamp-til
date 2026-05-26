# Spring MVC HandlerMapping과 @RequestMapping 정리

## 학습 날짜

2026-05-25

## 학습 주제

- HandlerMapping
- `@RequestMapping`
- 요청 매핑 조건
- HTTP Method 매핑
- Params, Headers 매핑
- Consumes, Produces
- HandlerMethod

---

## 1. HandlerMapping의 역할

HandlerMapping은 클라이언트 요청을 처리할 Handler를 찾는 역할을 한다.

Spring MVC에서 Handler는 보통 Controller 메서드이다.

```text
DispatcherServlet → HandlerMapping → Handler 조회
```

DispatcherServlet은 직접 Controller를 찾지 않는다.

등록된 HandlerMapping 목록을 순서대로 조회하면서 현재 요청을 처리할 수 있는 Handler를 찾는다.

HandlerMapping이 반환하는 것은 단순 Handler만이 아니라, 인터셉터 체인을 포함한 HandlerExecutionChain일 수 있다.

---

## 2. RequestMappingHandlerMapping

애노테이션 기반 Controller를 사용할 때 핵심 HandlerMapping은 `RequestMappingHandlerMapping`이다.

이 객체는 애플리케이션 시작 시점에 `@Controller`, `@RequestMapping`, `@GetMapping` 등이 붙은 메서드를 스캔한다.

그리고 요청 조건과 HandlerMethod를 매핑 정보로 등록한다.

요청이 들어오면 URL, HTTP Method, Header, Parameter, Content-Type, Accept 조건 등을 비교해 가장 적절한 HandlerMethod를 찾는다.

---

## 3. @RequestMapping 기본

`@RequestMapping`은 HTTP 요청을 특정 컨트롤러 메서드와 연결하는 애노테이션이다.

```java
@RequestMapping("/hello")
public String hello() {
    return "hello";
}
```

클래스 레벨과 메서드 레벨에 함께 사용할 수 있다.

```java
@RequestMapping("/members")
@Controller
public class MemberController {

    @RequestMapping("/new")
    public String newMember() {
        return "member/new";
    }
}
```

이 경우 최종 경로는 `/members/new`가 된다.

---

## 4. HTTP Method 매핑

HTTP Method 조건을 지정할 수 있다.

```java
@RequestMapping(value = "/members", method = RequestMethod.GET)
public String members() {
    return "members";
}
```

실무에서는 축약 애노테이션을 많이 사용한다.

```java
@GetMapping("/members")
@PostMapping("/members")
@PutMapping("/members/{id}")
@DeleteMapping("/members/{id}")
```

HTTP Method를 명확히 구분하면 REST API 설계에서 의미가 더 분명해진다.

---

## 5. Params 매핑

특정 요청 파라미터가 있을 때만 매핑되도록 할 수 있다.

```java
@GetMapping(value = "/order", params = "type=pizza")
public String pizzaOrder() {
    return "pizza";
}
```

예를 들어 `/order?type=pizza` 요청에만 이 메서드가 호출된다.

Params 조건은 같은 URL이라도 파라미터 값에 따라 다른 Handler를 선택하고 싶을 때 사용할 수 있다.

---

## 6. Headers 매핑

특정 Header 조건으로 매핑할 수 있다.

```java
@GetMapping(value = "/members", headers = "X-API-VERSION=1")
public String apiV1() {
    return "v1";
}
```

Header 조건은 API 버전, 클라이언트 종류, 특정 요청 구분이 필요할 때 사용할 수 있다.

다만 Header 조건이 많아지면 API 이해가 어려워질 수 있으므로 남용하지 않는 것이 좋다.

---

## 7. consumes와 produces

`consumes`는 요청의 Content-Type 조건을 지정한다.

```java
@PostMapping(value = "/users", consumes = "application/json")
public String createJsonUser() {
    return "ok";
}
```

클라이언트가 JSON 요청 본문을 보낼 때 사용한다.

`produces`는 클라이언트가 원하는 응답 타입인 Accept 조건과 관련된다.

```java
@GetMapping(value = "/users", produces = "application/json")
public User user() {
    return new User();
}
```

정리하면 다음과 같다.

- Content-Type: 클라이언트가 서버로 보내는 요청 본문 형식
- Accept: 클라이언트가 서버에게 원하는 응답 형식
- consumes: 서버가 받을 수 있는 요청 본문 형식
- produces: 서버가 반환할 수 있는 응답 형식

---

## 8. HandlerMethod

Spring MVC에서 애노테이션 기반 Controller 메서드는 내부적으로 HandlerMethod로 표현된다.

HandlerMethod는 다음 정보를 가진다.

- Controller Bean
- 호출할 Method
- Method Parameter 정보
- 반환 타입 정보

HandlerMapping은 요청에 맞는 HandlerMethod를 찾고, HandlerAdapter가 이 HandlerMethod를 실행한다.

이후 파라미터를 해석하고 반환값을 처리하는 구조가 이어진다.

---

## 9. 보충 정리

처음에는 URL만 맞으면 Controller가 실행된다고 생각하기 쉽다.

하지만 Spring MVC는 훨씬 많은 조건을 함께 비교한다.

- URL Path
- HTTP Method
- Query Parameter
- Header
- Content-Type
- Accept

이 조건을 잘 활용하면 하나의 URL이라도 요청 의미를 더 명확하게 구분할 수 있다.

반대로 조건이 너무 많으면 API가 복잡해지므로, URL 설계와 HTTP Method 설계를 먼저 명확히 하는 것이 중요하다.

---

## 10. 오늘 정리

HandlerMapping은 요청을 처리할 Handler를 찾는 핵심 구성요소이다.

중요한 정리는 다음과 같다.

- DispatcherServlet은 HandlerMapping을 통해 Handler를 찾는다.
- 애노테이션 기반 Controller는 RequestMappingHandlerMapping이 처리한다.
- `@RequestMapping`은 URL뿐 아니라 method, params, headers, consumes, produces 조건을 가진다.
- `@GetMapping`, `@PostMapping`은 HTTP Method 매핑을 편하게 해주는 축약 애노테이션이다.
- Content-Type은 요청 본문 형식, Accept는 원하는 응답 형식이다.
- HandlerMethod는 Controller Bean과 실제 호출 메서드 정보를 담는다.
