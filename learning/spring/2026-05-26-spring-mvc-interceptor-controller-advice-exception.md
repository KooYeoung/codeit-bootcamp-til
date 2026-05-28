# Spring MVC 공통 기능과 예외 처리 정리

## 학습 날짜

2026-05-26

## 학습 주제

- HandlerInterceptor
- preHandle
- postHandle
- afterCompletion
- HandlerMethod 활용
- `@ControllerAdvice`
- `@RestControllerAdvice`
- Servlet 예외 처리
- Spring Boot 기본 오류 처리
- HandlerExceptionResolver
- `@ExceptionHandler`
- `@ResponseStatus`
- ResponseStatusException

---

## 1. Interceptor가 필요한 이유

웹 애플리케이션에서는 여러 Controller에 공통으로 적용해야 하는 기능이 많다.

예를 들어 다음 기능이 있다.

- 인증 확인
- 권한 검사
- 요청 로그
- 성능 측정
- 공통 통계 수집

이런 기능을 모든 Controller 메서드에 직접 작성하면 중복이 많아진다.

Spring MVC는 이런 공통 기능을 처리하기 위해 Interceptor를 제공한다.

---

## 2. HandlerInterceptor 구조

Interceptor는 `HandlerInterceptor`를 구현해서 만든다.

핵심 메서드는 다음 세 가지이다.

```java
boolean preHandle(HttpServletRequest request,
                  HttpServletResponse response,
                  Object handler)

void postHandle(HttpServletRequest request,
                HttpServletResponse response,
                Object handler,
                ModelAndView modelAndView)

void afterCompletion(HttpServletRequest request,
                     HttpServletResponse response,
                     Object handler,
                     Exception ex)
```

---

## 3. preHandle

`preHandle()`은 Controller 호출 전에 실행된다.

반환값이 `true`이면 다음 단계로 진행한다.

반환값이 `false`이면 요청 처리를 중단한다.

따라서 로그인 체크나 권한 검사에 적합하다.

```java
if (!isLogin(request)) {
    response.sendRedirect("/login");
    return false;
}
return true;
```

---

## 4. postHandle

`postHandle()`은 Controller 호출 후, View 렌더링 전에 실행된다.

Controller가 반환한 ModelAndView를 확인하거나 수정할 수 있다.

다만 `@ResponseBody` API 응답에서는 View 렌더링 흐름과 다르게 동작할 수 있으므로 사용 목적을 명확히 해야 한다.

---

## 5. afterCompletion

`afterCompletion()`은 요청 처리가 완료된 후 실행된다.

View 렌더링 이후에 호출되며, 예외가 발생해도 호출된다.

따라서 리소스 정리, 로그 마무리, ThreadLocal 제거 같은 반드시 수행해야 하는 작업에 적합하다.

---

## 6. Interceptor와 Filter 차이

Filter와 Interceptor는 모두 공통 기능 처리에 사용할 수 있다.

하지만 위치가 다르다.

| 구분 | Filter | Interceptor |
|---|---|---|
| 기술 | Servlet | Spring MVC |
| 실행 위치 | DispatcherServlet 전 | DispatcherServlet 후, Controller 전후 |
| 제공 정보 | request, response | request, response, handler, ModelAndView |
| 주 사용처 | 인코딩, 보안 필터, 전체 웹 요청 | 인증, 인가, 컨트롤러 공통 로직 |

Spring MVC 내부 정보가 필요하면 Interceptor가 더 적합하다.

---

## 7. @ControllerAdvice

`@ControllerAdvice`는 여러 Controller에 공통 기능을 적용할 수 있게 해준다.

함께 사용할 수 있는 대표 기능은 다음과 같다.

- `@InitBinder`: 전역 바인딩 설정
- `@ModelAttribute`: 전역 모델 데이터 추가
- `@ExceptionHandler`: 전역 예외 처리

```java
@ControllerAdvice
public class GlobalControllerAdvice {
}
```

`@RestControllerAdvice`는 `@ControllerAdvice`에 `@ResponseBody`가 결합된 형태로, REST API 예외 응답 처리에 자주 사용한다.

---

## 8. Servlet 예외 처리

Servlet에서 예외가 처리되지 않으면 예외는 WAS까지 전달된다.

WAS는 오류 상태 코드에 맞는 오류 페이지를 찾아 응답할 수 있다.

또는 `response.sendError()`를 호출해 명시적으로 오류 처리 흐름을 시작할 수 있다.

```java
response.sendError(HttpServletResponse.SC_BAD_REQUEST, "bad request");
```

`sendError()`는 정상 응답이 아니라 오류 응답을 전송하기 위한 기능이다.

---

## 9. Spring Boot 기본 오류 처리

Spring Boot는 기본적으로 `/error` 경로를 통해 오류를 처리한다.

BasicErrorController가 요청의 Accept 헤더나 상황에 따라 HTML 오류 페이지 또는 JSON 오류 응답을 만들 수 있다.

예를 들어 브라우저 요청에는 HTML 오류 페이지가 적합하고, REST API 요청에는 JSON 오류 응답이 적합하다.

HTML 오류 페이지와 API 오류 응답은 요구사항이 다르므로 구분해서 설계해야 한다.

---

## 10. HandlerExceptionResolver

HandlerExceptionResolver는 Controller에서 발생한 예외를 처리할 수 있는 전략 인터페이스이다.

Spring MVC는 예외가 발생하면 등록된 ExceptionResolver들을 순서대로 실행해 예외를 처리할 수 있는지 확인한다.

기본 구현체에는 다음이 있다.

- ExceptionHandlerExceptionResolver
- ResponseStatusExceptionResolver
- DefaultHandlerExceptionResolver

---

## 11. @ExceptionHandler

`@ExceptionHandler`는 특정 예외를 처리할 메서드를 지정한다.

```java
@ExceptionHandler(IllegalArgumentException.class)
public ResponseEntity<ErrorResponse> handleIllegalArgument(IllegalArgumentException e) {
    return ResponseEntity.badRequest()
            .body(new ErrorResponse("BAD_REQUEST", e.getMessage()));
}
```

Controller 내부에 작성하면 해당 Controller에만 적용된다.

`@ControllerAdvice` 또는 `@RestControllerAdvice` 안에 작성하면 여러 Controller에 공통 적용할 수 있다.

---

## 12. @ResponseStatus와 ResponseStatusException

`@ResponseStatus`는 예외 클래스나 예외 처리 메서드에 HTTP 상태 코드를 지정할 수 있다.

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class NotFoundException extends RuntimeException {
}
```

`ResponseStatusException`은 런타임에 상태 코드와 메시지를 지정해 예외를 던질 수 있다.

```java
throw new ResponseStatusException(HttpStatus.NOT_FOUND, "user not found");
```

간단한 상태 코드 매핑에는 편리하지만, API 오류 응답 구조를 통일하려면 `@ExceptionHandler` 기반 전역 처리 방식이 더 적합하다.

---

## 13. DefaultHandlerExceptionResolver

DefaultHandlerExceptionResolver는 Spring MVC 내부에서 발생하는 여러 기본 예외를 HTTP 상태 코드로 변환한다.

예를 들어 지원하지 않는 HTTP 메서드로 요청하면 405 Method Not Allowed가 반환될 수 있다.

이런 기본 예외 처리가 있기 때문에 Spring MVC는 잘못된 요청에 대해 적절한 상태 코드를 자동으로 반환할 수 있다.

---

## 14. API 예외 처리 보충

API 예외 응답은 일관된 구조를 가지는 것이 좋다.

예를 들어 다음처럼 설계할 수 있다.

```json
{
  "code": "USER_NOT_FOUND",
  "message": "사용자를 찾을 수 없습니다.",
  "status": 404
}
```

중요한 기준은 다음과 같다.

- HTTP 상태 코드는 의미에 맞게 사용한다.
- 클라이언트가 분기할 수 있는 오류 코드를 제공한다.
- 사용자에게 보여줄 메시지와 내부 로그 메시지를 구분한다.
- 예외를 무조건 500으로 처리하지 않는다.
- 예외 응답 스펙을 전역에서 일관되게 관리한다.

---

## 15. 오늘 정리

Spring MVC의 공통 기능과 예외 처리는 유지보수에 매우 중요하다.

중요한 정리는 다음과 같다.

- Interceptor는 Controller 호출 전후에 공통 기능을 적용한다.
- `preHandle()`은 요청 진행 여부를 제어할 수 있다.
- `afterCompletion()`은 예외가 발생해도 호출되어 정리 작업에 적합하다.
- `@ControllerAdvice`는 여러 Controller에 공통 기능을 적용한다.
- `@RestControllerAdvice`는 API 전역 예외 처리에 적합하다.
- Servlet 예외는 WAS까지 전달될 수 있다.
- Spring Boot는 BasicErrorController로 기본 오류 처리를 제공한다.
- HandlerExceptionResolver는 예외 처리 전략을 제공한다.
- `@ExceptionHandler`는 실무 API 예외 처리에서 가장 자주 사용된다.
