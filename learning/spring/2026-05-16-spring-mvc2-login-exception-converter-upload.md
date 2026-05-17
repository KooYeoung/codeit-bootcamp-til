# Spring MVC2 로그인, 예외 처리, 타입 컨버터, 파일 업로드 정리

## 학습 날짜

2026-05-16

## 학습 주제

- 쿠키
- 세션
- HttpSession
- 필터
- 인터셉터
- ArgumentResolver
- 예외 처리와 오류 페이지
- API 예외 처리
- 타입 컨버터
- 포맷터
- 파일 업로드

---

## 1. 로그인 요구사항

로그인 기능은 단순히 사용자가 아이디와 비밀번호를 입력하는 것에서 끝나지 않는다.

로그인 후에는 다음과 같은 기능이 필요하다.

- 로그인 전 홈 화면 표시
- 회원 가입
- 로그인
- 로그인 후 사용자 이름 표시
- 로그인 사용자만 상품 관리 접근 가능
- 로그아웃

특히 중요한 요구사항은 인증되지 않은 사용자가 상품 관리 화면에 직접 접근하지 못하도록 막는 것이다.

화면에서 상품 관리 버튼을 숨기는 것만으로는 부족하다.

사용자가 직접 `/items` 같은 URL을 입력하면 접근할 수 있기 때문이다.

따라서 서버에서 반드시 로그인 여부를 확인해야 한다.

---

## 2. 쿠키

쿠키는 서버가 클라이언트에게 전달하고, 클라이언트가 이후 요청마다 다시 서버에 보내는 값이다.

서버는 응답에 `Set-Cookie` 헤더를 넣어 쿠키를 전달한다.

브라우저는 이후 같은 서버에 요청할 때 `Cookie` 헤더를 함께 보낸다.

로그인 처리에서 쿠키를 사용하면 다음과 같은 흐름이 가능하다.

1. 사용자가 로그인한다.
2. 서버가 로그인 성공 시 쿠키를 응답한다.
3. 브라우저는 쿠키를 저장한다.
4. 이후 요청마다 쿠키를 서버에 보낸다.
5. 서버는 쿠키 값을 보고 로그인 사용자인지 판단한다.

하지만 쿠키에 회원 ID 같은 중요한 값을 그대로 저장하면 보안 문제가 생긴다.

쿠키는 클라이언트가 가지고 있기 때문에 사용자가 값을 조작할 수 있다.

---

## 3. 쿠키의 보안 문제

쿠키에 중요한 정보를 직접 저장하면 다음 문제가 생길 수 있다.

- 사용자가 쿠키 값을 임의로 변경할 수 있다.
- 쿠키가 탈취되면 다른 사용자가 로그인한 것처럼 사용할 수 있다.
- 쿠키에 민감한 정보가 그대로 노출될 수 있다.

예를 들어 쿠키에 `memberId=1`이 저장되어 있고, 사용자가 이 값을 `memberId=2`로 바꿀 수 있다면 다른 회원으로 로그인한 것처럼 동작할 위험이 있다.

따라서 쿠키에는 중요한 값을 직접 저장하면 안 된다.

대신 서버에는 중요한 값을 저장하고, 클라이언트에는 추측하기 어려운 식별자만 전달하는 방식이 필요하다.

이 방식이 세션이다.

---

## 4. 세션

세션은 서버가 로그인 상태를 보관하는 방식이다.

쿠키에는 실제 회원 정보가 아니라 세션 ID만 저장한다.

흐름은 다음과 같다.

1. 사용자가 로그인한다.
2. 서버가 세션 저장소에 로그인 회원 정보를 저장한다.
3. 서버는 추측하기 어려운 세션 ID를 생성한다.
4. 세션 ID를 쿠키로 클라이언트에 전달한다.
5. 클라이언트는 이후 요청마다 세션 ID 쿠키를 보낸다.
6. 서버는 세션 ID로 세션 저장소를 조회해 로그인 사용자를 찾는다.

이 방식에서는 클라이언트가 실제 회원 정보를 알 수 없다.

또한 서버가 세션을 무효화하면 로그아웃 처리를 할 수 있다.

---

## 5. HttpSession

스프링 MVC에서는 서블릿이 제공하는 `HttpSession`을 사용할 수 있다.

직접 세션 저장소를 만들 수도 있지만, 실무에서는 보통 `HttpSession`을 활용한다.

로그인 성공 시 세션에 회원 정보를 저장할 수 있다.

```java
HttpSession session = request.getSession();
session.setAttribute("loginMember", loginMember);
```

로그인 여부를 확인할 때는 세션에서 값을 꺼낸다.

```java
HttpSession session = request.getSession(false);

if (session == null) {
    return "redirect:/login";
}

Member loginMember = (Member) session.getAttribute("loginMember");
```

`request.getSession(false)`는 기존 세션이 없으면 새로 만들지 않는다.

로그인 확인 용도로는 불필요한 세션 생성을 막기 위해 `false`를 사용하는 것이 좋다.

로그아웃 시에는 세션을 무효화한다.

```java
session.invalidate();
```

---

## 6. 필터

필터는 서블릿 기술에서 제공하는 공통 처리 기능이다.

요청 흐름은 다음과 같다.

```text
HTTP 요청 → WAS → 필터 → 서블릿 → 컨트롤러
```

필터는 요청이 서블릿으로 들어가기 전에 먼저 동작한다.

따라서 요청 로그, 인증 체크, 인코딩 처리 같은 공통 로직에 사용할 수 있다.

필터는 요청을 다음 단계로 넘길지 말지를 결정할 수 있다.

```java
chain.doFilter(request, response);
```

이 코드를 호출하면 다음 필터 또는 서블릿으로 요청이 넘어간다.

호출하지 않으면 요청 흐름이 중단된다.

로그인하지 않은 사용자를 로그인 화면으로 보내고 싶을 때 사용할 수 있다.

---

## 7. 인터셉터

인터셉터는 스프링 MVC가 제공하는 공통 처리 기능이다.

요청 흐름은 다음과 같다.

```text
HTTP 요청 → WAS → 필터 → DispatcherServlet → 인터셉터 → 컨트롤러
```

인터셉터는 스프링 MVC 내부에서 동작하기 때문에 컨트롤러 호출 전후 처리를 더 세밀하게 다룰 수 있다.

대표 메서드는 다음과 같다.

```java
boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
```

- `preHandle`: 컨트롤러 호출 전 실행
- `postHandle`: 컨트롤러 호출 후, 뷰 렌더링 전 실행
- `afterCompletion`: 뷰 렌더링 후 실행

인증 체크는 보통 `preHandle`에서 처리한다.

---

## 8. 필터와 인터셉터 차이

필터와 인터셉터는 모두 공통 관심사를 처리할 수 있다.

하지만 동작 위치와 목적이 다르다.

### 필터

- 서블릿 기술
- DispatcherServlet 호출 전 동작
- 모든 서블릿 요청에 적용 가능
- request, response를 교체할 수 있음
- 웹 전반의 공통 처리에 적합

### 인터셉터

- 스프링 MVC 기술
- DispatcherServlet 이후, 컨트롤러 호출 전후 동작
- handler 정보를 사용할 수 있음
- 컨트롤러와 관련된 공통 처리에 적합
- URL 패턴 설정이 편리함

스프링 MVC 애플리케이션에서 로그인 인증 체크는 인터셉터를 사용하면 더 편리한 경우가 많다.

---

## 9. ArgumentResolver

ArgumentResolver는 컨트롤러 메서드의 파라미터를 원하는 방식으로 만들어주는 기능이다.

예를 들어 로그인 회원을 매번 세션에서 직접 꺼내면 컨트롤러 코드가 반복된다.

```java
HttpSession session = request.getSession(false);
Member loginMember = (Member) session.getAttribute("loginMember");
```

이 반복을 줄이기 위해 커스텀 애노테이션과 ArgumentResolver를 만들 수 있다.

```java
@GetMapping("/")
public String home(@Login Member loginMember) {
    // 로그인 회원 사용
}
```

이렇게 하면 컨트롤러는 세션 조회 로직을 몰라도 된다.

로그인 회원을 꺼내는 공통 로직은 ArgumentResolver가 담당한다.

---

## 10. 예외 처리와 오류 페이지

웹 애플리케이션에서 예외가 발생했는데 애플리케이션 내부에서 처리하지 못하면 WAS까지 예외가 전달된다.

흐름은 다음과 같다.

```text
WAS ← 필터 ← 서블릿 ← 인터셉터 ← 컨트롤러
```

서블릿은 예외 처리를 위해 다음 방식을 제공한다.

- 예외 발생
- `response.sendError(statusCode, message)`

예외가 발생하면 보통 HTTP 500 상태 코드가 반환된다.

존재하지 않는 URL을 호출하면 HTTP 404 상태 코드가 반환된다.

사용자에게 기본 오류 화면을 그대로 보여주는 것은 좋지 않기 때문에, 서비스에서는 오류 페이지를 별도로 제공해야 한다.

---

## 11. 스프링 부트 오류 페이지

스프링 부트는 기본 오류 처리 기능을 제공한다.

정적 오류 페이지를 만들려면 다음 위치를 사용할 수 있다.

```text
resources/static/error/404.html
resources/static/error/500.html
resources/templates/error/4xx.html
resources/templates/error/5xx.html
```

스프링 부트는 상태 코드에 맞는 오류 페이지를 찾아 보여준다.

기본적인 오류 페이지는 스프링 부트 기능을 활용하면 편리하다.

다만 서비스 요구사항에 맞는 메시지, 디자인, 로그 추적은 별도로 고려해야 한다.

---

## 12. API 예외 처리

HTML 화면은 오류 페이지를 보여주면 된다.

하지만 API는 오류 페이지 HTML을 반환하면 안 된다.

API 클라이언트는 JSON 응답을 기대한다.

따라서 API 예외는 상황에 맞는 JSON 오류 응답을 내려주어야 한다.

예를 들어 다음과 같은 형태가 필요하다.

```json
{
  "status": 400,
  "code": "BAD_REQUEST",
  "message": "잘못된 요청입니다."
}
```

API 예외 처리에서 중요한 것은 오류 응답 스펙을 일관성 있게 설계하는 것이다.

---

## 13. HandlerExceptionResolver

스프링 MVC는 예외를 처리하기 위해 HandlerExceptionResolver를 제공한다.

컨트롤러에서 예외가 발생하면 ExceptionResolver가 예외를 처리할 기회를 가진다.

스프링이 기본으로 제공하는 ExceptionResolver도 있다.

- `ExceptionHandlerExceptionResolver`
- `ResponseStatusExceptionResolver`
- `DefaultHandlerExceptionResolver`

각각의 역할은 다르다.

- `ExceptionHandlerExceptionResolver`: `@ExceptionHandler` 처리
- `ResponseStatusExceptionResolver`: `@ResponseStatus`, `ResponseStatusException` 처리
- `DefaultHandlerExceptionResolver`: 스프링 내부 기본 예외를 HTTP 상태 코드로 변환

실무에서는 `@ExceptionHandler`와 `@ControllerAdvice`를 많이 사용한다.

---

## 14. @ExceptionHandler와 @ControllerAdvice

`@ExceptionHandler`는 특정 예외를 처리하는 메서드를 지정한다.

```java
@ExceptionHandler(IllegalArgumentException.class)
public ResponseEntity<ErrorResult> illegalExHandle(IllegalArgumentException e) {
    ErrorResult errorResult = new ErrorResult("BAD", e.getMessage());
    return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
}
```

컨트롤러마다 반복되는 예외 처리 로직은 `@ControllerAdvice`로 분리할 수 있다.

```java
@RestControllerAdvice
public class ExControllerAdvice {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResult> illegalExHandle(IllegalArgumentException e) {
        ErrorResult errorResult = new ErrorResult("BAD", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
    }
}
```

컨트롤러는 핵심 요청 처리에 집중하고, 예외 처리 로직은 별도 클래스로 분리할 수 있다.

API 예외 처리는 대부분 이 구조를 사용하면 관리하기 좋다.

---

## 15. 스프링 타입 컨버터

HTTP 요청 파라미터는 기본적으로 문자이다.

예를 들어 `/hello?data=10`에서 `10`은 숫자처럼 보이지만 실제로는 문자열이다.

스프링 MVC는 요청 파라미터를 컨트롤러 파라미터 타입에 맞게 자동 변환해준다.

```java
@GetMapping("/hello")
public String hello(@RequestParam Integer data) {
    return "ok";
}
```

여기서 `data=10` 문자열이 `Integer` 타입으로 변환된다.

타입 변환은 다음과 같은 곳에서 사용된다.

- `@RequestParam`
- `@ModelAttribute`
- `@PathVariable`
- `@Value`
- 뷰 렌더링

---

## 16. Converter, ConversionService, Formatter

직접 타입 변환 규칙을 만들고 싶으면 `Converter`를 구현할 수 있다.

```java
public class StringToIntegerConverter implements Converter<String, Integer> {
    @Override
    public Integer convert(String source) {
        return Integer.valueOf(source);
    }
}
```

`ConversionService`는 여러 Converter를 등록하고 관리하는 역할을 한다.

`Formatter`는 문자와 객체 사이의 변환에 특화되어 있으며, Locale 정보도 활용할 수 있다.

예를 들어 숫자를 화면에 출력할 때 천 단위 콤마를 붙이거나, 날짜를 특정 형식으로 출력할 때 사용할 수 있다.

정리하면 다음과 같다.

- `Converter`: 타입을 다른 타입으로 변환
- `ConversionService`: Converter들을 등록하고 사용하는 서비스
- `Formatter`: 문자 기반 출력과 입력 포맷 처리

---

## 17. 파일 업로드

일반적인 HTML Form 전송 방식은 `application/x-www-form-urlencoded`이다.

이 방식은 폼 데이터를 문자로 전송한다.

```text
username=kim&age=20
```

하지만 파일은 문자 데이터가 아니라 바이너리 데이터이다.

또한 파일 업로드 폼에서는 보통 문자 데이터와 파일 데이터를 함께 전송해야 한다.

예를 들어 다음 데이터를 함께 보낼 수 있다.

- 상품명
- 설명
- 첨부 파일
- 이미지 파일

이 문제를 해결하기 위해 `multipart/form-data` 방식이 필요하다.

---

## 18. multipart/form-data와 MultipartFile

파일 업로드를 하려면 HTML Form에 다음 설정이 필요하다.

```html
<form method="post" enctype="multipart/form-data">
    <input type="text" name="itemName">
    <input type="file" name="file">
</form>
```

`multipart/form-data`는 요청 본문을 여러 part로 나누어 전송한다.

각 part는 서로 다른 데이터를 담을 수 있다.

- 문자 데이터 part
- 파일 데이터 part

스프링은 multipart 요청을 처리하고, 컨트롤러에서 `MultipartFile`로 받을 수 있게 해준다.

```java
@PostMapping("/upload")
public String upload(@RequestParam MultipartFile file) throws IOException {
    String originalFilename = file.getOriginalFilename();
    file.transferTo(new File("저장 경로"));
    return "ok";
}
```

`MultipartFile`에서 사용할 수 있는 주요 기능은 다음과 같다.

- 원본 파일명 조회
- 파일 크기 조회
- 파일 내용 읽기
- 지정한 위치로 파일 저장

---

## 19. 업로드 파일명과 저장 파일명 분리

파일을 저장할 때는 사용자가 업로드한 원본 파일명을 그대로 저장하지 않는 것이 좋다.

서버 내부 저장 파일명은 UUID 등을 사용해서 충돌을 방지하는 것이 안전하다.

예를 들어 사용자가 `spring.png` 파일을 업로드했다고 하자.

이 파일명을 그대로 서버에 저장하면 같은 이름의 파일이 업로드될 때 덮어쓰기 문제가 생길 수 있다.

그래서 보통 다음처럼 관리한다.

- 원본 파일명: 사용자가 업로드한 이름
- 저장 파일명: 서버 내부에서 사용할 고유한 이름

```text
원본 파일명: spring.png
저장 파일명: 550e8400-e29b-41d4-a716-446655440000.png
```

DB에는 원본 파일명과 저장 파일명을 함께 저장할 수 있다.

사용자가 다운로드할 때는 원본 파일명으로 내려주고, 서버에서는 저장 파일명으로 실제 파일을 찾는다.

---

## 20. 오늘 정리

오늘 학습한 내용은 웹 애플리케이션에서 실무적으로 자주 필요한 기능들이다.

로그인 처리는 쿠키만으로 구현하면 보안 문제가 있으므로 세션 기반으로 처리하는 것이 안전하다.

인증 체크 같은 공통 관심사는 컨트롤러마다 반복하지 않고 필터나 인터셉터로 분리해야 한다.

예외 처리는 HTML 화면과 API 응답을 구분해서 설계해야 한다.

타입 컨버터와 포맷터는 HTTP 요청 값과 화면 출력 값을 원하는 타입과 형식으로 다루게 도와준다.

파일 업로드는 `multipart/form-data`와 `MultipartFile`을 이해하는 것이 핵심이다.

중요한 정리는 다음과 같다.

- 쿠키는 클라이언트가 보관하고 요청마다 전달한다.
- 세션은 서버가 로그인 정보를 보관하고 클라이언트에는 세션 ID만 전달한다.
- 필터는 서블릿 앞단에서 동작한다.
- 인터셉터는 스프링 MVC 내부에서 컨트롤러 호출 전후에 동작한다.
- API 예외는 JSON 응답 스펙을 일관성 있게 설계해야 한다.
- `@ExceptionHandler`와 `@ControllerAdvice`를 사용하면 API 예외 처리를 깔끔하게 분리할 수 있다.
- 스프링 타입 컨버터는 문자 요청 값을 필요한 타입으로 바꿔준다.
- 파일 업로드는 `multipart/form-data`와 `MultipartFile`을 사용한다.

앞으로 로그인, 예외 처리, 파일 업로드를 구현할 때는 단순 기능 구현뿐 아니라 보안, 공통 관심사 분리, 응답 스펙 설계까지 함께 고려해야겠다.
