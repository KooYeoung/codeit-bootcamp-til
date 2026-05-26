# Spring MVC Method Arguments와 HttpMessageConverter 정리

## 학습 날짜

2026-05-25

## 학습 주제

- HandlerAdapter
- HandlerMethodArgumentResolver
- `@RequestParam`
- `@PathVariable`
- `@ModelAttribute`
- `@RequestBody`
- `HttpEntity`, `RequestEntity`
- `@RequestHeader`, `@CookieValue`
- `@SessionAttributes`, `@SessionAttribute`
- HttpMessageConverter
- 커스텀 ArgumentResolver

---

## 1. HandlerAdapter의 역할

HandlerMapping이 요청에 맞는 Handler를 찾으면, HandlerAdapter가 그 Handler를 실제로 호출한다.

애노테이션 기반 컨트롤러에서는 `RequestMappingHandlerAdapter`가 핵심이다.

Controller 메서드는 매우 다양한 파라미터와 반환 타입을 가질 수 있다.

```java
@GetMapping("/members/{id}")
public String findMember(
        @PathVariable Long id,
        @RequestParam String name,
        Model model
) {
    return "member";
}
```

이 파라미터들을 누가 만들어 넣어주는지가 중요하다.

그 역할을 하는 것이 HandlerMethodArgumentResolver이다.

---

## 2. HandlerMethodArgumentResolver

HandlerMethodArgumentResolver는 컨트롤러 메서드 파라미터 값을 만들어주는 전략 인터페이스이다.

동작 흐름은 다음과 같다.

1. HandlerAdapter가 Controller 메서드를 호출하려고 한다.
2. 메서드 파라미터 목록을 확인한다.
3. 각 파라미터를 처리할 수 있는 ArgumentResolver를 찾는다.
4. Resolver가 요청 정보를 읽어 파라미터 값을 만든다.
5. 완성된 인자 배열로 Controller 메서드를 호출한다.

이 구조 덕분에 Spring MVC는 다양한 파라미터 타입을 지원할 수 있다.

---

## 3. @RequestParam

`@RequestParam`은 쿼리 파라미터나 Form 데이터를 받을 때 사용한다.

```java
@GetMapping("/search")
public String search(@RequestParam String keyword) {
    return "ok";
}
```

요청 예시는 다음과 같다.

```text
/search?keyword=spring
```

기본적으로 필수 값으로 처리되며, 없으면 오류가 발생할 수 있다.

필수가 아니면 `required = false`를 사용하거나 `defaultValue`를 지정할 수 있다.

```java
@RequestParam(required = false) String keyword
@RequestParam(defaultValue = "1") int page
```

---

## 4. @PathVariable

`@PathVariable`은 URL 경로 일부를 변수로 받을 때 사용한다.

```java
@GetMapping("/members/{memberId}")
public String find(@PathVariable Long memberId) {
    return "ok";
}
```

요청 예시는 다음과 같다.

```text
/members/10
```

REST API에서 자원 식별자를 표현할 때 자주 사용한다.

---

## 5. @ModelAttribute

`@ModelAttribute`는 요청 파라미터를 객체에 바인딩할 때 사용한다.

```java
@PostMapping("/members")
public String save(@ModelAttribute MemberForm form) {
    return "ok";
}
```

요청 파라미터 이름과 객체 필드 이름이 맞으면 Spring MVC가 객체를 생성하고 값을 넣어준다.

예를 들어 다음 요청은 `MemberForm.username`, `MemberForm.age`에 바인딩될 수 있다.

```text
/members?username=kim&age=20
```

객체 바인딩 과정에서는 타입 변환과 검증도 함께 연결될 수 있다.

---

## 6. @RequestBody

`@RequestBody`는 HTTP 요청 본문을 읽어 Java 객체로 변환할 때 사용한다.

```java
@PostMapping("/users")
public ResponseEntity<String> create(@RequestBody UserDto userDto) {
    return ResponseEntity.ok("ok");
}
```

JSON 요청 본문 예시는 다음과 같다.

```json
{
  "name": "kim",
  "age": 20
}
```

`@RequestBody`는 쿼리 파라미터나 Form 데이터를 읽는 것이 아니라, HTTP Body 자체를 읽는다.

JSON 변환은 HttpMessageConverter가 담당한다.

---

## 7. HttpEntity와 RequestEntity

`HttpEntity<T>`는 HTTP Header와 Body를 함께 다룰 수 있는 객체이다.

```java
@PostMapping("/users")
public ResponseEntity<String> create(HttpEntity<UserDto> httpEntity) {
    UserDto body = httpEntity.getBody();
    HttpHeaders headers = httpEntity.getHeaders();
    return ResponseEntity.ok("ok");
}
```

`RequestEntity<T>`는 HttpEntity에 HTTP Method와 URL 정보까지 포함한 타입이다.

요청 전체 정보를 더 명확히 다루고 싶을 때 사용할 수 있다.

---

## 8. Header, Cookie, Session 관련 파라미터

Spring MVC는 Header, Cookie, Session 값도 편리하게 받을 수 있다.

## @RequestHeader

```java
@GetMapping("/headers")
public String header(@RequestHeader("User-Agent") String userAgent) {
    return userAgent;
}
```

## @CookieValue

```java
@GetMapping("/cookie")
public String cookie(@CookieValue("SESSION") String sessionId) {
    return sessionId;
}
```

## @SessionAttribute

```java
@GetMapping("/login-user")
public String loginUser(@SessionAttribute("loginMember") Member member) {
    return member.getName();
}
```

`@SessionAttribute`는 이미 세션에 저장된 값을 조회할 때 사용한다.

`@SessionAttributes`는 특정 Model 속성을 세션에 보관하는 기능으로, 사용 목적이 다르다.

---

## 9. HttpMessageConverter

HttpMessageConverter는 HTTP Body와 Java 객체 사이를 변환하는 구성요소이다.

요청에서는 다음 역할을 한다.

```text
HTTP Request Body(JSON) → Java Object
```

응답에서는 다음 역할을 한다.

```text
Java Object → HTTP Response Body(JSON)
```

대표적인 구현체는 다음과 같다.

- `StringHttpMessageConverter`: 문자열 처리
- `MappingJackson2HttpMessageConverter`: JSON 처리
- `ByteArrayHttpMessageConverter`: 바이트 배열 처리

`@RequestBody`와 `@ResponseBody`는 HttpMessageConverter와 밀접하게 연결된다.

---

## 10. MessageConverter가 동작하지 않는 경우

HttpMessageConverter는 HTTP Body를 읽거나 쓸 때 동작한다.

따라서 다음과 같은 경우에는 직접 관련이 적다.

- `@RequestParam`으로 쿼리 파라미터를 읽는 경우
- `@ModelAttribute`로 Form 데이터를 바인딩하는 경우
- View 이름을 반환해 HTML을 렌더링하는 경우

JSON API에서는 MessageConverter가 핵심이지만, HTML Form 기반 MVC에서는 DataBinder와 ViewResolver가 더 중요할 수 있다.

---

## 11. 커스텀 HandlerMethodArgumentResolver

Spring MVC는 개발자가 직접 ArgumentResolver를 만들 수 있게 지원한다.

예를 들어 로그인 사용자를 컨트롤러에서 매번 세션에서 꺼내지 않고, 커스텀 애노테이션으로 받을 수 있다.

```java
@GetMapping("/me")
public String me(@LoginMember Member member) {
    return member.getName();
}
```

커스텀 Resolver는 다음 두 가지를 구현한다.

- 이 파라미터를 지원하는지 판단
- 실제 파라미터 값을 생성

이 기능은 컨트롤러 중복 코드를 줄이고, 공통 파라미터 처리 규칙을 중앙화할 때 유용하다.

---

## 12. 보충 정리

요청 데이터를 받을 때 기준은 데이터가 어디에 있느냐이다.

| 위치 | 사용 방법 |
|---|---|
| URL 경로 | `@PathVariable` |
| Query String | `@RequestParam` |
| HTML Form | `@RequestParam`, `@ModelAttribute` |
| JSON Body | `@RequestBody` |
| Header | `@RequestHeader` |
| Cookie | `@CookieValue` |
| Session | `@SessionAttribute` |

이 기준을 잡으면 어떤 애노테이션을 사용해야 할지 훨씬 명확해진다.

---

## 13. 오늘 정리

Spring MVC의 컨트롤러 메서드가 다양한 파라미터를 받을 수 있는 이유는 HandlerMethodArgumentResolver 덕분이다.

중요한 정리는 다음과 같다.

- HandlerAdapter는 Handler를 실제로 호출한다.
- HandlerMethodArgumentResolver는 메서드 파라미터 값을 만들어준다.
- `@RequestParam`은 쿼리/Form 데이터에 사용한다.
- `@PathVariable`은 URL 경로 변수에 사용한다.
- `@ModelAttribute`는 요청 파라미터를 객체에 바인딩한다.
- `@RequestBody`는 HTTP Body를 객체로 변환한다.
- HttpMessageConverter는 Body와 Java 객체 사이를 변환한다.
- 커스텀 ArgumentResolver로 공통 파라미터 처리를 확장할 수 있다.
