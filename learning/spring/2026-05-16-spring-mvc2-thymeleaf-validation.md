# Spring MVC2 타임리프, 폼, 메시지, 검증 정리

## 학습 날짜

2026-05-16

## 학습 주제

- 타임리프 기본 기능
- 타임리프 스프링 통합과 폼
- 메시지와 국제화
- 검증 직접 처리
- BindingResult
- FieldError, ObjectError
- Validator 분리
- Bean Validation
- 폼 전송 객체 분리

---

## 1. 타임리프를 사용하는 이유

타임리프는 서버에서 HTML을 동적으로 렌더링하기 위한 템플릿 엔진이다.

정적인 HTML 파일은 고정된 내용만 보여줄 수 있다. 하지만 웹 애플리케이션에서는 서버에서 조회한 데이터에 따라 화면 내용이 달라져야 한다.

예를 들어 상품 목록, 상품 상세, 회원 정보 같은 화면은 서버에서 전달한 데이터를 HTML에 넣어서 보여주어야 한다.

타임리프는 이런 동적 HTML 렌더링을 자연스럽게 처리할 수 있게 도와준다.

또한 타임리프는 순수 HTML 구조를 최대한 유지하면서 작성할 수 있기 때문에, 서버 없이 HTML 파일을 열어도 어느 정도 화면 구조를 확인할 수 있다는 장점이 있다.

---

## 2. th:text와 th:utext

타임리프에서 텍스트를 출력할 때 가장 기본적으로 사용하는 속성은 `th:text`이다.

```html
<span th:text="${data}">기본 값</span>
```

`th:text`는 HTML 이스케이프를 적용한다.

예를 들어 데이터에 `<b>Hello</b>`가 들어있다면, 실제 태그로 해석하지 않고 문자 그대로 출력한다.

이 방식은 사용자가 입력한 값에 HTML이나 스크립트가 포함되어 있을 때 보안상 안전하다.

반면 `th:utext`는 이스케이프하지 않고 출력한다.

```html
<span th:utext="${data}">기본 값</span>
```

`th:utext`를 사용하면 HTML 태그가 실제 태그로 해석될 수 있다.

따라서 사용자가 입력한 값에는 함부로 사용하면 안 된다.

정리하면 다음과 같다.

- `th:text`: HTML 이스케이프 적용, 일반적으로 안전한 출력
- `th:utext`: 이스케이프 미적용, HTML을 그대로 렌더링

기본적으로는 `th:text`를 사용하는 것이 좋다.

---

## 3. SpringEL과 변수 접근

타임리프는 `${...}` 문법으로 모델에 담긴 값에 접근한다.

```html
<span th:text="${user.username}"></span>
```

이 문법은 SpringEL을 사용한다.

객체의 프로퍼티, 리스트, 맵 등에 접근할 수 있다.

```html
<span th:text="${user.username}"></span>
<span th:text="${users[0].username}"></span>
<span th:text="${userMap['userA'].username}"></span>
```

컨트롤러에서 모델에 값을 담으면 타임리프에서 해당 값을 사용할 수 있다.

```java
model.addAttribute("user", user);
```

타임리프는 화면에서 데이터를 표현하는 역할을 하고, 컨트롤러는 화면에 필요한 데이터를 모델에 담아 전달하는 역할을 한다.

---

## 4. URL 링크

타임리프에서 URL을 만들 때는 `@{...}` 문법을 사용한다.

```html
<a th:href="@{/items}">상품 목록</a>
```

경로 변수나 쿼리 파라미터도 처리할 수 있다.

```html
<a th:href="@{/items/{itemId}(itemId=${item.id})}">상품 상세</a>
<a th:href="@{/items(itemName=${item.itemName})}">상품 검색</a>
```

URL을 문자열로 직접 이어붙이면 실수하기 쉽다.

타임리프의 URL 문법을 사용하면 경로 변수와 파라미터를 더 안전하고 명확하게 표현할 수 있다.

---

## 5. 반복과 조건

타임리프에서 반복은 `th:each`를 사용한다.

```html
<tr th:each="item : ${items}">
    <td th:text="${item.id}"></td>
    <td th:text="${item.itemName}"></td>
</tr>
```

조건부 출력은 `th:if`, `th:unless`, `th:switch` 등을 사용할 수 있다.

```html
<span th:if="${item.price > 10000}">고가 상품</span>
<span th:unless="${item.price > 10000}">일반 상품</span>
```

이 기능을 사용하면 서버에서 전달한 데이터에 따라 화면 일부를 보여주거나 숨길 수 있다.

---

## 6. 템플릿 조각과 레이아웃

타임리프는 반복되는 HTML 구조를 조각으로 분리할 수 있다.

예를 들어 header, footer, menu 같은 영역은 여러 화면에서 반복된다.

이런 부분을 템플릿 조각으로 분리하면 중복을 줄일 수 있다.

```html
<div th:fragment="header">
    <h1>공통 헤더</h1>
</div>
```

다른 HTML에서 이 조각을 가져와 사용할 수 있다.

```html
<div th:replace="~{template/fragment :: header}"></div>
```

템플릿 조각과 레이아웃을 사용하면 화면 구조를 더 일관성 있게 관리할 수 있다.

---

## 7. 타임리프와 스프링 통합

타임리프는 스프링과 통합되면서 폼 처리, 검증 오류 출력, 메시지 처리 등을 편리하게 제공한다.

스프링 통합으로 제공되는 주요 기능은 다음과 같다.

- SpringEL 통합
- 스프링 빈 호출 지원
- `th:object`
- `th:field`
- `th:errors`
- `th:errorclass`
- 체크박스, 라디오 버튼, 셀렉트 박스 처리
- 메시지와 국제화 통합
- 검증 오류 처리 통합
- ConversionService 통합

스프링 부트에서는 `spring-boot-starter-thymeleaf` 의존성을 추가하면 대부분의 설정이 자동으로 적용된다.

---

## 8. th:object와 선택 변수 식

폼을 처리할 때는 `th:object`를 사용해 폼에서 사용할 객체를 지정할 수 있다.

```html
<form th:object="${item}" method="post">
    <input type="text" th:field="*{itemName}">
</form>
```

`th:object="${item}"`은 이 폼에서 사용할 객체를 `item`으로 지정한다는 뜻이다.

`*{itemName}`은 선택 변수 식이다.

이미 `th:object`로 `item`을 선택했기 때문에, `*{itemName}`은 `item.itemName`과 같은 의미로 이해할 수 있다.

폼 내부에서는 `th:object`와 `*{...}`를 함께 사용하면 코드가 더 간결해진다.

---

## 9. th:field

`th:field`는 폼 입력 필드에서 자주 필요한 속성을 자동으로 처리해준다.

```html
<input type="text" th:field="*{itemName}">
```

렌더링 결과는 다음과 비슷하다.

```html
<input type="text" id="itemName" name="itemName" value="...">
```

`th:field`는 다음 속성을 자동으로 맞춰준다.

- id
- name
- value
- checked
- selected

따라서 등록 폼과 수정 폼에서 같은 구조를 더 편리하게 사용할 수 있다.

특히 검증 오류가 발생했을 때 사용자가 입력한 값을 다시 보여주는 데 도움이 된다.

---

## 10. 체크박스, 라디오 버튼, 셀렉트 박스

타임리프는 체크박스, 라디오 버튼, 셀렉트 박스를 편리하게 처리한다.

체크박스는 체크하지 않으면 HTML Form 전송 시 값이 아예 넘어오지 않는다.

이 문제를 해결하기 위해 타임리프는 체크박스에 숨겨진 필드를 함께 만들어준다.

```html
<input type="checkbox" th:field="*{open}">
```

타임리프는 체크 여부를 판단할 수 있도록 필요한 hidden input을 자동으로 처리한다.

라디오 버튼과 셀렉트 박스는 여러 선택지 중 하나를 선택하는 경우에 사용한다.

`th:field`를 사용하면 현재 객체의 값과 선택지 값을 비교해서 자동으로 checked 또는 selected를 적용해준다.

---

## 11. 메시지 기능

메시지 기능은 화면에 표시되는 문구를 HTML에 직접 작성하지 않고 별도 파일에서 관리하는 기능이다.

예를 들어 화면 여러 곳에 “상품명”이라는 문구가 있다고 하자.

이 문구를 “상품 이름”으로 바꾸고 싶으면 모든 HTML을 찾아 수정해야 한다.

하지만 메시지 파일에 다음처럼 정의해두면 한 곳만 수정하면 된다.

```properties
item.itemName=상품명
item.price=가격
item.quantity=수량
```

타임리프에서는 `#{...}` 문법으로 메시지를 조회한다.

```html
<label th:text="#{item.itemName}"></label>
```

이 방식은 화면 문구를 코드와 분리하는 데 도움이 된다.

---

## 12. 국제화

국제화는 언어별 메시지 파일을 나누어 관리하는 방식이다.

```properties
# messages_ko.properties
item.itemName=상품명
```

```properties
# messages_en.properties
item.itemName=Item Name
```

사용자의 언어 환경에 따라 적절한 메시지 파일이 선택된다.

웹에서는 보통 HTTP `Accept-Language` 헤더를 참고할 수 있다.

국제화를 적용하면 같은 화면이라도 한국어 사용자에게는 한국어 문구를, 영어 사용자에게는 영어 문구를 보여줄 수 있다.

---

## 13. 검증이 필요한 이유

웹 폼에서 사용자가 잘못된 값을 입력할 수 있다.

예를 들어 상품 등록 화면에서 다음과 같은 문제가 발생할 수 있다.

- 상품명을 입력하지 않음
- 가격에 문자를 입력함
- 가격이 너무 작거나 너무 큼
- 수량이 너무 큼
- 가격과 수량의 합계가 최소 기준을 만족하지 않음

검증은 컨트롤러의 중요한 역할 중 하나이다.

클라이언트 검증은 사용자 편의성을 높이는 데 좋지만 조작할 수 있으므로 보안상 신뢰하면 안 된다.

따라서 최종적으로 서버 검증은 반드시 필요하다.

---

## 14. 직접 검증 처리의 한계

처음에는 컨트롤러에서 직접 검증 로직을 작성할 수 있다.

```java
Map<String, String> errors = new HashMap<>();

if (!StringUtils.hasText(item.getItemName())) {
    errors.put("itemName", "상품 이름은 필수입니다.");
}
```

검증 오류가 있으면 다시 폼으로 이동하고, 오류 메시지를 화면에 출력할 수 있다.

하지만 직접 검증 방식은 다음 문제가 있다.

- 검증 코드가 컨트롤러에 많아진다.
- 타입 오류 처리까지 직접 하기 어렵다.
- 오류 메시지 관리가 복잡해진다.
- 필드 오류와 객체 오류를 체계적으로 다루기 어렵다.

스프링은 이런 문제를 해결하기 위해 `BindingResult`를 제공한다.

---

## 15. BindingResult

`BindingResult`는 스프링에서 검증 오류를 담는 객체이다.

```java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, BindingResult bindingResult) {
    if (bindingResult.hasErrors()) {
        return "validation/v2/addForm";
    }

    itemRepository.save(item);
    return "redirect:/validation/v2/items/{itemId}";
}
```

중요한 점은 `BindingResult`는 검증 대상 바로 뒤에 위치해야 한다는 것이다.

```java
@ModelAttribute Item item, BindingResult bindingResult
```

순서가 바뀌면 스프링이 어떤 객체의 검증 결과인지 알 수 없다.

`BindingResult`가 있으면 타입 오류가 발생해도 컨트롤러가 호출되고, 오류 정보를 담아 다시 폼으로 이동할 수 있다.

---

## 16. FieldError와 ObjectError

필드 하나와 관련된 오류는 `FieldError`로 표현한다.

```java
bindingResult.addError(new FieldError("item", "itemName", "상품명은 필수입니다."));
```

여러 필드를 함께 확인해야 하는 오류는 `ObjectError`로 표현한다.

예를 들어 가격과 수량을 곱한 값이 10,000원 이상이어야 한다는 검증은 특정 필드 하나만의 문제가 아니다.

```java
bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다."));
```

정리하면 다음과 같다.

- `FieldError`: 특정 필드 오류
- `ObjectError`: 객체 전체와 관련된 오류

---

## 17. 오류 코드와 메시지

오류 메시지를 코드에 직접 작성하면 변경이 어렵다.

스프링은 오류 코드를 사용해서 메시지를 관리할 수 있게 해준다.

예를 들어 다음과 같은 메시지 코드를 만들 수 있다.

```properties
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1}까지 허용합니다.
```

오류 코드를 사용하면 화면에 보여줄 메시지를 properties 파일에서 관리할 수 있다.

또한 스프링은 `MessageCodesResolver`를 통해 구체적인 오류 코드부터 범용적인 오류 코드까지 단계적으로 찾는다.

이 덕분에 특정 필드에만 적용할 메시지와 공통으로 적용할 메시지를 나누어 관리할 수 있다.

---

## 18. Validator 분리

검증 로직이 많아지면 컨트롤러에서 분리하는 것이 좋다.

스프링은 `Validator` 인터페이스를 제공한다.

```java
@Component
public class ItemValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        Item item = (Item) target;

        if (!StringUtils.hasText(item.getItemName())) {
            errors.rejectValue("itemName", "required");
        }
    }
}
```

`supports()`는 이 검증기가 어떤 타입을 지원하는지 확인한다.

`validate()`는 실제 검증 로직을 수행한다.

검증기를 분리하면 컨트롤러는 요청 흐름을 처리하고, 검증기는 검증 책임을 담당하게 되어 역할이 더 명확해진다.

---

## 19. Bean Validation

Bean Validation은 검증 로직을 애노테이션으로 선언할 수 있게 해주는 표준 기술이다.

예를 들어 다음과 같이 사용할 수 있다.

```java
public class Item {

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(9999)
    private Integer quantity;
}
```

대표적인 검증 애노테이션은 다음과 같다.

- `@NotBlank`: null, 빈 문자열, 공백만 있는 문자열 허용 안 함
- `@NotNull`: null 허용 안 함
- `@Range`: 특정 범위 안의 값만 허용
- `@Max`: 최대값 제한

Bean Validation을 사용하면 반복되는 필드 검증 로직을 줄일 수 있다.

스프링에서는 `@Validated` 또는 `@Valid`를 사용해 검증을 실행할 수 있다.

---

## 20. Bean Validation의 한계와 폼 객체 분리

Bean Validation은 편리하지만 모든 상황을 완벽하게 해결하지는 않는다.

예를 들어 등록과 수정의 검증 조건이 다를 수 있다.

- 등록 시에는 id가 없어도 된다.
- 수정 시에는 id가 필요할 수 있다.
- 등록과 수정에서 요구하는 필수 값이 다를 수 있다.

이런 경우 groups를 사용할 수 있다.

하지만 groups를 사용하면 도메인 객체가 화면 요구사항에 영향을 많이 받을 수 있다.

그래서 실무에서는 폼 전송 객체를 따로 분리하는 방식이 더 명확할 수 있다.

예를 들어 다음처럼 나눌 수 있다.

- `ItemSaveForm`
- `ItemUpdateForm`

폼 객체를 분리하면 등록 폼과 수정 폼의 검증 규칙을 각각 명확하게 작성할 수 있다.

도메인 객체는 핵심 비즈니스 의미에 집중하고, 폼 객체는 화면 입력 요구사항에 집중하게 된다.

---

## 21. Bean Validation과 HTTP 메시지 컨버터

HTML Form 요청에서는 검증 오류가 발생하면 다시 폼 화면을 보여주면 된다.

하지만 API 요청에서는 오류를 JSON 응답으로 내려주어야 한다.

`@RequestBody`와 Bean Validation을 함께 사용할 때는 HTTP 메시지 컨버터가 JSON을 객체로 변환한 뒤 검증이 수행된다.

이때 JSON 자체가 객체로 변환되지 못하는 경우와, 객체 변환은 되었지만 검증에 실패한 경우를 구분해야 한다.

API에서는 검증 실패 시 클라이언트가 이해할 수 있는 오류 응답 구조를 따로 설계하는 것이 중요하다.

---

## 22. 오늘 정리

오늘 학습한 타임리프와 검증은 웹 애플리케이션의 폼 처리에서 매우 중요한 내용이다.

타임리프는 서버 데이터를 HTML에 자연스럽게 렌더링하고, 스프링과 통합하면 폼 처리와 검증 오류 출력까지 편리하게 지원한다.

검증은 처음에는 직접 구현할 수 있지만, 점점 `BindingResult`, `Validator`, Bean Validation으로 발전시키는 것이 좋다.

중요한 정리는 다음과 같다.

- 타임리프는 동적 HTML 렌더링을 담당한다.
- `th:object`, `th:field`는 폼 처리를 편리하게 해준다.
- 메시지 기능은 화면 문구를 한 곳에서 관리하게 해준다.
- 국제화는 언어별 메시지 파일을 통해 처리한다.
- 서버 검증은 반드시 필요하다.
- `BindingResult`는 검증 오류를 담고 다시 폼으로 돌아갈 수 있게 해준다.
- Bean Validation은 반복되는 검증 로직을 애노테이션으로 줄여준다.
- 등록과 수정 요구사항이 다르면 폼 객체를 분리하는 것이 더 명확할 수 있다.

앞으로 폼을 구현할 때는 화면 입력 객체, 검증 책임, 오류 메시지 관리 위치를 함께 고려해야겠다.
