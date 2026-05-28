# Spring MVC 실행 구조: Reflection과 InvocableHandlerMethod 정리

## 학습 날짜

2026-05-26

## 학습 주제

- Java Reflection
- Method
- MethodParameter
- HandlerMethod
- InvocableHandlerMethod
- ServletInvocableHandlerMethod
- ArgumentResolver
- ReturnValueHandler
- 동적 메서드 호출 구조

---

## 1. Spring MVC 실행 구조를 이해해야 하는 이유

Spring MVC에서는 컨트롤러 메서드를 다음처럼 자유롭게 작성할 수 있다.

```java
@GetMapping("/users/{id}")
public ResponseEntity<UserDto> findUser(
        @PathVariable Long id,
        @RequestParam(required = false) String type
) {
    return ResponseEntity.ok(userService.find(id, type));
}
```

메서드 파라미터도 다양하고 반환 타입도 다양하다.

그런데 Spring MVC는 이런 메서드를 어떻게 찾아서, 어떤 값을 넣고, 어떻게 반환값을 처리할까?

이 질문에 답하기 위해 Reflection과 `InvocableHandlerMethod` 구조를 학습해야 한다.

---

## 2. Java Reflection

Reflection은 런타임에 클래스, 필드, 메서드 정보를 조회하고 사용할 수 있게 해주는 기능이다.

일반 메서드 호출은 컴파일 시점에 호출 대상이 정해진다.

```java
service.save("data");
```

Reflection을 사용하면 런타임에 메서드 정보를 찾아 호출할 수 있다.

```java
Method method = service.getClass().getMethod("save", String.class);
method.invoke(service, "data");
```

Spring MVC는 요청 URL에 맞는 컨트롤러 메서드를 찾고, Reflection 기반으로 해당 메서드를 호출한다.

---

## 3. Method와 MethodParameter

`Method`는 Java Reflection에서 메서드 자체를 표현하는 객체이다.

메서드 이름, 반환 타입, 파라미터 타입, 애노테이션 정보를 확인할 수 있다.

`MethodParameter`는 Spring이 메서드 파라미터 하나를 표현하기 위해 사용하는 객체이다.

예를 들어 다음 메서드가 있다고 하자.

```java
public String hello(@RequestParam String name, @PathVariable Long id) {
    return "ok";
}
```

Spring MVC는 각 파라미터에 대해 다음 정보를 확인한다.

- 파라미터 타입
- 파라미터 이름
- 파라미터 위치
- 붙어 있는 애노테이션
- 필수 여부

이 정보를 바탕으로 어떤 ArgumentResolver가 처리할지 결정한다.

---

## 4. HandlerMethod

HandlerMethod는 Controller Bean과 실행할 Method 정보를 함께 담는 객체이다.

단순히 메서드 정보만 있으면 실제 호출 대상 객체를 알 수 없다.

따라서 Spring MVC는 다음 정보를 함께 관리한다.

- Controller Bean
- 호출할 Method
- MethodParameter 정보

HandlerMapping은 요청 조건에 맞는 HandlerMethod를 찾아 반환한다.

HandlerAdapter는 이 HandlerMethod를 실행한다.

---

## 5. InvocableHandlerMethod

`InvocableHandlerMethod`는 HandlerMethod를 실제로 호출할 수 있도록 확장한 객체이다.

핵심 역할은 다음과 같다.

1. 메서드 파라미터 목록을 분석한다.
2. 각 파라미터를 처리할 ArgumentResolver를 찾는다.
3. 요청 정보에서 파라미터 값을 만든다.
4. Reflection으로 실제 메서드를 호출한다.

즉, `InvocableHandlerMethod`는 “컨트롤러 메서드를 실행 가능한 형태로 만든 객체”라고 볼 수 있다.

---

## 6. ServletInvocableHandlerMethod

`ServletInvocableHandlerMethod`는 Servlet 환경에서 컨트롤러 메서드 실행과 반환값 처리를 담당한다.

`InvocableHandlerMethod`가 메서드 호출에 집중한다면, `ServletInvocableHandlerMethod`는 반환값 처리까지 연결한다.

예를 들어 반환값이 다음 중 무엇인지에 따라 처리 방식이 달라진다.

- View 이름
- ModelAndView
- `@ResponseBody` 객체
- ResponseEntity
- void

반환값 처리는 HandlerMethodReturnValueHandler가 담당한다.

---

## 7. ArgumentResolver와 ReturnValueHandler 관계

컨트롤러 메서드 실행 전에는 ArgumentResolver가 필요하다.

```text
HTTP 요청 → ArgumentResolver → 메서드 파라미터 값
```

컨트롤러 메서드 실행 후에는 ReturnValueHandler가 필요하다.

```text
메서드 반환값 → ReturnValueHandler → View 또는 Response Body
```

정리하면 다음과 같다.

| 시점 | 담당 객체 | 역할 |
|---|---|---|
| 메서드 호출 전 | HandlerMethodArgumentResolver | 파라미터 값 생성 |
| 메서드 호출 | InvocableHandlerMethod | 실제 메서드 실행 |
| 메서드 호출 후 | HandlerMethodReturnValueHandler | 반환값 처리 |

---

## 8. 커스텀 동적 메서드 호출 구조

강의자료에서는 특정 애노테이션이 붙은 메서드를 찾아 동적으로 호출하는 예제를 다룬다.

핵심 구조는 다음과 같다.

- MethodExtractor: 특정 애노테이션이 붙은 메서드를 추출
- DynamicMethodInvoker: 요청 정보에 맞는 메서드 호출
- CustomHandlerMethodArgumentResolver: 메서드 파라미터 값 생성
- DispatcherController: 클라이언트 요청 수신

이 구조는 Spring MVC 내부 구조를 단순화해 직접 구현해보는 예제라고 볼 수 있다.

Spring MVC도 요청에 맞는 메서드를 찾고, 파라미터를 만들고, 메서드를 호출하고, 반환값을 응답으로 처리한다.

---

## 9. 보충 정리

Reflection은 강력하지만 무분별하게 사용하면 코드가 복잡해지고 컴파일 시점 안정성이 떨어질 수 있다.

프레임워크 내부에서는 유연한 구조를 만들기 위해 Reflection이 자주 사용된다.

하지만 일반 애플리케이션 코드에서는 꼭 필요한 경우에만 제한적으로 사용하는 것이 좋다.

Spring MVC를 사용할 때 개발자가 직접 Reflection을 사용할 일은 많지 않다.

다만 내부 동작을 이해하면 `@RequestParam`, `@ModelAttribute`, `@RequestBody`, `ResponseEntity` 같은 기능이 어떻게 동작하는지 더 명확해진다.

---

## 10. 오늘 정리

Spring MVC는 Reflection과 여러 전략 객체를 활용해 컨트롤러 메서드를 동적으로 실행한다.

중요한 정리는 다음과 같다.

- Reflection은 런타임에 메서드 정보를 조회하고 호출할 수 있게 해준다.
- Method는 Java 메서드 정보를 표현한다.
- MethodParameter는 Spring이 메서드 파라미터 정보를 표현하는 객체이다.
- HandlerMethod는 Controller Bean과 Method를 함께 담는다.
- InvocableHandlerMethod는 파라미터를 준비하고 실제 메서드를 호출한다.
- ServletInvocableHandlerMethod는 Servlet 환경에서 반환값 처리까지 연결한다.
- ArgumentResolver는 파라미터 값을 만든다.
- ReturnValueHandler는 반환값을 처리한다.

앞으로 Spring MVC 내부 오류를 분석할 때는 “요청 매핑 → 파라미터 해석 → 메서드 호출 → 반환값 처리” 흐름으로 따라가야겠다.
