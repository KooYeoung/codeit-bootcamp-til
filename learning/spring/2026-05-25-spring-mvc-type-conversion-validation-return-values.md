# Spring MVC 타입 변환, 검증, 반환값 처리 정리

## 학습 날짜

2026-05-25

## 학습 주제

- DataBinder
- `@InitBinder`
- Converter
- ConverterFactory
- ConversionService
- Formatter
- Validation
- BindingResult
- Validator
- Bean Validation
- ReturnValueHandler
- `@ResponseBody`
- ResponseEntity
- RedirectAttributes, Flash Attributes

---

## 1. DataBinder란

DataBinder는 HTTP 요청 파라미터를 Java 객체에 바인딩하는 과정에서 사용된다.

예를 들어 Form 요청으로 들어온 문자열 값을 객체 필드에 넣어야 한다.

```text
name=kim&age=20
```

이때 `age`는 HTTP 요청에서는 문자열이지만, Java 객체에서는 `int`나 `Integer`일 수 있다.

DataBinder는 다음 작업을 연결한다.

- 요청 파라미터 바인딩
- 타입 변환
- 검증
- 바인딩 오류 저장

즉, `@ModelAttribute`로 객체를 받을 때 내부적으로 중요한 역할을 한다.

---

## 2. @InitBinder

`@InitBinder`는 특정 컨트롤러에서 바인딩 설정을 커스터마이징할 때 사용한다.

예를 들어 날짜 형식을 지정할 수 있다.

```java
@InitBinder
public void initBinder(WebDataBinder binder) {
    // 바인딩 설정 추가
}
```

`@InitBinder`는 특정 Controller에만 적용할 수도 있고, `@ControllerAdvice`와 함께 사용해 여러 Controller에 공통 적용할 수도 있다.

---

## 3. Converter

Converter는 한 타입을 다른 타입으로 변환하는 단일 변환기이다.

```java
public class StringToIntegerConverter implements Converter<String, Integer> {
    @Override
    public Integer convert(String source) {
        return Integer.valueOf(source);
    }
}
```

Converter는 요청 파라미터를 객체 필드 타입으로 바꾸거나, 특정 문자열 형식을 도메인 타입으로 바꾸는 데 사용할 수 있다.

---

## 4. ConversionService

ConversionService는 여러 Converter를 등록하고 관리하는 타입 변환 서비스이다.

개별 Converter가 변환 로직 하나라면, ConversionService는 애플리케이션 전반의 변환 체계이다.

Spring MVC는 요청 파라미터 바인딩, `@RequestParam`, `@PathVariable`, `@ModelAttribute` 등에서 ConversionService를 활용한다.

```java
conversionService.convert("10", Integer.class);
```

---

## 5. Formatter

Formatter는 문자 기반 입출력 변환에 특화되어 있다.

예를 들어 다음과 같은 상황에 적합하다.

- 숫자를 `1,000`처럼 출력
- `2026-05-25` 문자열을 날짜 타입으로 변환
- Locale에 따라 통화 형식을 다르게 처리

Converter가 범용 타입 변환이라면, Formatter는 웹 화면과 사용자 입력에 가까운 포맷 변환이라고 이해할 수 있다.

---

## 6. Validation이 필요한 이유

사용자 입력은 항상 올바르다고 믿으면 안 된다.

예를 들어 주문 Form에서 다음 문제가 발생할 수 있다.

- 상품명을 입력하지 않음
- 가격이 음수임
- 수량이 너무 큼
- 날짜 형식이 잘못됨
- 여러 필드의 조합이 비즈니스 규칙을 만족하지 않음

클라이언트 검증은 사용자 편의성을 높이는 데 좋지만, 조작할 수 있으므로 서버 검증은 반드시 필요하다.

---

## 7. BindingResult

`BindingResult`는 바인딩 오류와 검증 오류를 담는 객체이다.

```java
@PostMapping("/order")
public String submit(
        @ModelAttribute Order order,
        BindingResult bindingResult
) {
    if (bindingResult.hasErrors()) {
        return "order/form";
    }
    return "redirect:/orders";
}
```

중요한 점은 `BindingResult`가 검증 대상 객체 바로 뒤에 와야 한다는 것이다.

```java
@ModelAttribute Order order, BindingResult bindingResult
```

순서가 맞지 않으면 어떤 객체의 오류인지 연결할 수 없다.

---

## 8. Field Error와 Object Error

특정 필드에 대한 오류는 `rejectValue()`로 처리할 수 있다.

```java
bindingResult.rejectValue("item", "required", "상품명은 필수입니다.");
```

객체 전체와 관련된 오류는 `reject()`를 사용할 수 있다.

```java
bindingResult.reject("totalPriceMin", "총 주문 금액이 부족합니다.");
```

필드 하나의 문제가 아니라 여러 필드를 함께 계산해야 하는 검증은 객체 오류로 처리하는 것이 자연스럽다.

---

## 9. Validator 분리

검증 로직이 컨트롤러에 많아지면 유지보수가 어렵다.

Spring의 `Validator` 인터페이스를 구현해 검증 책임을 분리할 수 있다.

```java
public class OrderValidator implements Validator {
    @Override
    public boolean supports(Class<?> clazz) {
        return Order.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        Order order = (Order) target;
        // 검증 로직
    }
}
```

검증기를 분리하면 컨트롤러는 요청 흐름을 담당하고, Validator는 검증 규칙을 담당하게 된다.

---

## 10. Bean Validation

Bean Validation은 애노테이션 기반 검증 표준이다.

```java
public class OrderForm {
    @NotBlank
    private String item;

    @Min(100)
    @Max(10000)
    private int price;
}
```

Spring MVC에서는 `@Valid` 또는 `@Validated`를 사용해 검증을 실행한다.

```java
@PostMapping("/order")
public String submit(@Validated @ModelAttribute OrderForm form,
                     BindingResult bindingResult) {
    if (bindingResult.hasErrors()) {
        return "order/form";
    }
    return "redirect:/orders";
}
```

`@Validated`는 Spring이 제공하며, validation group 기능을 사용할 수 있다.

---

## 11. @RequestBody + Validation

JSON 요청에서도 Bean Validation을 사용할 수 있다.

```java
@PostMapping("/api/orders")
public ResponseEntity<?> create(@Valid @RequestBody OrderRequest request) {
    return ResponseEntity.ok().build();
}
```

다만 Form 검증과 API 검증의 오류 응답 방식은 다르다.

- Form 검증 실패: 다시 HTML Form 화면으로 이동
- API 검증 실패: JSON 오류 응답 반환

API에서는 오류 응답 스펙을 별도로 설계하는 것이 중요하다.

---

## 12. Return Value 처리

컨트롤러 메서드는 다양한 값을 반환할 수 있다.

- String View 이름
- ModelAndView
- `@ResponseBody` 객체
- ResponseEntity
- RedirectView
- void

Spring MVC는 반환값을 처리하기 위해 HandlerMethodReturnValueHandler를 사용한다.

파라미터를 ArgumentResolver가 처리했다면, 반환값은 ReturnValueHandler가 처리한다고 볼 수 있다.

---

## 13. @ResponseBody와 ResponseEntity

`@ResponseBody`는 반환값을 View 이름으로 해석하지 않고 HTTP 응답 본문에 직접 쓴다.

```java
@ResponseBody
@GetMapping("/api/users")
public UserDto user() {
    return new UserDto("kim");
}
```

객체는 HttpMessageConverter를 통해 JSON으로 변환된다.

`ResponseEntity`는 본문뿐 아니라 상태 코드와 헤더까지 제어할 수 있다.

```java
return ResponseEntity.status(HttpStatus.CREATED)
        .header("Location", "/users/1")
        .body(userDto);
```

API 응답에서는 `ResponseEntity`가 유용하다.

---

## 14. RedirectAttributes와 Flash Attributes

Redirect를 할 때는 URL이 새 요청으로 바뀐다.

일반 Model 데이터는 redirect 이후 유지되지 않는다.

Redirect URL에 값을 붙이고 싶다면 `RedirectAttributes`를 사용할 수 있다.

```java
redirectAttributes.addAttribute("id", savedId);
return "redirect:/orders/{id}";
```

한 번만 사용할 메시지는 Flash Attribute로 전달할 수 있다.

```java
redirectAttributes.addFlashAttribute("message", "저장되었습니다.");
```

Flash Attribute는 redirect 이후 다음 요청에서 한 번 사용되고 사라진다.

---

## 15. 보충 정리

타입 변환, 바인딩, 검증은 따로 떨어진 기능처럼 보이지만 실제로는 연결되어 있다.

```text
HTTP 요청 파라미터
→ DataBinder
→ 타입 변환
→ 객체 바인딩
→ 검증
→ BindingResult
```

이 흐름을 이해하면 Form 처리에서 발생하는 오류를 더 쉽게 분석할 수 있다.

---

## 16. 오늘 정리

Spring MVC의 Form 처리와 API 처리는 내부적으로 다양한 전략 객체가 협력한다.

중요한 정리는 다음과 같다.

- DataBinder는 요청 값을 객체에 바인딩한다.
- Converter는 타입 변환 로직 하나를 담당한다.
- ConversionService는 변환기들을 관리한다.
- Formatter는 문자 기반 포맷 변환에 특화되어 있다.
- BindingResult는 바인딩/검증 오류를 담는다.
- Validator로 검증 로직을 분리할 수 있다.
- Bean Validation은 애노테이션 기반 검증을 제공한다.
- `@ResponseBody`는 응답 본문에 직접 값을 쓴다.
- ResponseEntity는 상태 코드, 헤더, 본문을 함께 제어한다.
- Flash Attribute는 redirect 후 한 번만 사용할 데이터를 전달한다.
