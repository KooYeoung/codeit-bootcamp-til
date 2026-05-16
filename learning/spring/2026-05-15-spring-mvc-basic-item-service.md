# Spring MVC 1편: 기본 기능과 웹 페이지 만들기 정리

## 학습 날짜

2026-05-15

## 학습 주제

- Spring MVC 기본 기능
- 로깅
- 요청 매핑
- HTTP 요청 헤더 조회
- 요청 파라미터 처리
- `@RequestParam`
- `@ModelAttribute`
- `@RequestBody`
- `@ResponseBody`
- HTTP 메시지 컨버터
- Thymeleaf 기반 웹 페이지 만들기
- 상품 관리 예제
- PRG 패턴
- `RedirectAttributes`

---

## 1. Spring MVC 기본 기능 개요

Spring MVC는 HTTP 요청을 처리하고 응답을 생성하기 위한 다양한 기능을 제공한다.

서블릿을 직접 사용할 때는 `HttpServletRequest`, `HttpServletResponse`를 직접 다루어야 했다.

Spring MVC에서는 애노테이션 기반으로 요청을 매핑하고, 요청 데이터를 객체로 변환하고, 응답을 View 또는 HTTP Body로 반환할 수 있다.

이번 학습에서는 다음 흐름을 중심으로 정리했다.

- 요청 URL 매핑
- HTTP 메서드 매핑
- 요청 헤더 조회
- 쿼리 파라미터와 HTML Form 처리
- 요청 메시지 바디 처리
- JSON 요청과 응답 처리
- View 템플릿 응답
- HTTP API 응답
- 메시지 컨버터 구조

---

## 2. 로깅

실무에서는 `System.out.println()` 대신 로그를 사용한다.

Spring Boot는 기본적으로 SLF4J와 Logback을 사용한다.

SLF4J는 로그 인터페이스이고, Logback은 실제 구현체라고 이해할 수 있다.

로그는 다음처럼 사용할 수 있다.

```java
private final Logger log = LoggerFactory.getLogger(getClass());
```

Lombok을 사용하면 `@Slf4j`로 간단히 로그 객체를 만들 수 있다.

```java
@Slf4j
@RestController
public class LogTestController {
}
```

로그 레벨은 다음 순서로 구분된다.

```text
TRACE > DEBUG > INFO > WARN > ERROR
```

개발 환경에서는 DEBUG 로그를 사용하고, 운영 환경에서는 INFO 이상만 출력하는 식으로 조절할 수 있다.

올바른 로그 사용 방식은 다음과 같다.

```java
log.debug("data={}", data);
```

다음 방식은 좋지 않다.

```java
log.debug("data=" + data);
```

문자열 더하기 방식은 로그가 출력되지 않는 레벨이어도 문자열 연산이 먼저 실행될 수 있기 때문이다.

---

## 3. @Controller와 @RestController

`@Controller`는 Spring MVC에서 컨트롤러 역할을 하는 클래스에 붙인다.

`@Controller`에서 메서드가 문자열을 반환하면 기본적으로 View 이름으로 인식한다.

```java
@Controller
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

반면 `@RestController`는 반환 값을 View 이름으로 보지 않고 HTTP 메시지 바디에 직접 작성한다.

```java
@RestController
public class HelloApiController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

`@RestController`는 `@Controller`와 `@ResponseBody`가 합쳐진 것처럼 이해할 수 있다.

---

## 4. 요청 매핑

요청 매핑은 어떤 URL과 HTTP 메서드를 어떤 컨트롤러 메서드가 처리할지 연결하는 것이다.

기본적으로 `@RequestMapping`을 사용할 수 있다.

```java
@RequestMapping("/hello-basic")
public String helloBasic() {
    return "ok";
}
```

HTTP 메서드를 함께 지정할 수도 있다.

```java
@RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)
public String mappingGetV1() {
    return "ok";
}
```

더 간단하게는 `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping`을 사용할 수 있다.

```java
@GetMapping("/mapping-get-v2")
public String mappingGetV2() {
    return "ok";
}
```

---

## 5. @PathVariable

`@PathVariable`은 URL 경로에 포함된 값을 꺼낼 때 사용한다.

```java
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
    return "ok";
}
```

예를 들어 다음 요청이 들어오면:

```text
/mapping/userA
```

`userId` 값으로 `userA`를 받을 수 있다.

변수명이 같다면 애노테이션 값을 생략할 수도 있다.

```java
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable String userId) {
    return "ok";
}
```

여러 경로 변수를 사용할 수도 있다.

```java
@GetMapping("/mapping/users/{userId}/orders/{orderId}")
public String mappingPath(@PathVariable String userId,
                          @PathVariable Long orderId) {
    return "ok";
}
```

---

## 6. 요청 헤더 조회

Spring MVC는 요청 헤더, 쿠키, Locale, HTTP 메서드 같은 정보를 편리하게 받을 수 있다.

```java
public String headers(HttpServletRequest request,
                      HttpServletResponse response,
                      HttpMethod httpMethod,
                      Locale locale,
                      @RequestHeader MultiValueMap<String, String> headerMap,
                      @RequestHeader("host") String host,
                      @CookieValue(value = "myCookie", required = false) String cookie) {
    return "ok";
}
```

`@RequestHeader`를 사용하면 특정 헤더나 전체 헤더를 조회할 수 있다.

`@CookieValue`를 사용하면 특정 쿠키 값을 조회할 수 있다.

`MultiValueMap`은 하나의 키에 여러 값을 가질 수 있는 Map 구조이다.

HTTP 헤더나 쿼리 파라미터처럼 같은 키에 여러 값이 들어올 수 있는 경우 사용할 수 있다.

---

## 7. HTTP 요청 데이터 전달 방식

클라이언트가 서버로 데이터를 전달하는 대표적인 방식은 세 가지이다.

## GET 쿼리 파라미터

URL에 데이터를 포함해서 전달한다.

```text
/request-param?username=hello&age=20
```

검색, 필터, 페이징에서 자주 사용한다.

## POST HTML Form

HTML Form 데이터를 메시지 바디에 담아 전달한다.

```http
content-type: application/x-www-form-urlencoded

username=hello&age=20
```

회원 가입, 상품 주문 같은 HTML Form 처리에 사용한다.

## HTTP message body

JSON, XML, TEXT 같은 데이터를 HTTP 메시지 바디에 직접 담아 전달한다.

HTTP API에서 주로 사용하며, 최근에는 JSON을 많이 사용한다.

```json
{"username":"hello", "age":20}
```

---

## 8. @RequestParam

`@RequestParam`은 요청 파라미터를 조회할 때 사용한다.

GET 쿼리 파라미터와 POST HTML Form은 둘 다 같은 형식으로 전달되기 때문에 `@RequestParam`으로 조회할 수 있다.

```java
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
        @RequestParam("username") String memberName,
        @RequestParam("age") int memberAge) {
    return "ok";
}
```

요청 파라미터 이름과 변수명이 같으면 이름을 생략할 수 있다.

```java
@RequestParam String username
```

단순 타입이면 `@RequestParam` 자체도 생략할 수 있다.

```java
String username
```

하지만 학습 단계에서는 명확성을 위해 애노테이션을 붙이는 것이 이해하기 좋다.

필수 여부와 기본값도 지정할 수 있다.

```java
@RequestParam(required = false) String username
@RequestParam(defaultValue = "guest") String username
```

---

## 9. @ModelAttribute

`@ModelAttribute`는 요청 파라미터를 객체에 바인딩할 때 사용한다.

예를 들어 다음 객체가 있다고 하자.

```java
@Data
public class HelloData {
    private String username;
    private int age;
}
```

요청 파라미터가 다음과 같이 들어오면:

```text
/model-attribute?username=hello&age=20
```

Spring MVC는 객체를 생성하고 setter를 통해 값을 넣어준다.

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
    return "ok";
}
```

`@ModelAttribute`는 다음 과정을 자동으로 처리한다.

1. `HelloData` 객체 생성
2. 요청 파라미터 이름으로 프로퍼티 찾기
3. setter를 통해 값 바인딩
4. 타입 변환 수행

`@ModelAttribute`도 생략할 수 있지만, 처음에는 명시하는 것이 좋다.

---

## 10. @RequestBody

`@RequestBody`는 HTTP 메시지 바디를 직접 읽을 때 사용한다.

쿼리 파라미터나 HTML Form이 아니라 JSON처럼 메시지 바디에 직접 담긴 데이터를 처리할 때 사용한다.

단순 텍스트 요청은 다음처럼 받을 수 있다.

```java
@PostMapping("/request-body-string-v2")
public String requestBodyString(@RequestBody String messageBody) {
    return "ok";
}
```

JSON 요청을 객체로 받을 수도 있다.

```java
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJson(@RequestBody HelloData data) {
    return "ok";
}
```

이때 HTTP 메시지 컨버터가 JSON을 읽어 Java 객체로 변환한다.

JSON 요청을 처리하려면 요청의 `Content-Type`이 `application/json`이어야 한다.

중요한 점은 `@RequestBody`는 생략하면 안 된다는 것이다.

생략하면 Spring MVC가 `@ModelAttribute`로 처리하려고 할 수 있다.

---

## 11. @ResponseBody

`@ResponseBody`는 반환 값을 View 이름으로 해석하지 않고, HTTP 응답 메시지 바디에 직접 넣는다.

```java
@ResponseBody
@GetMapping("/response-body-string-v1")
public String responseBody() {
    return "ok";
}
```

객체를 반환하면 HTTP 메시지 컨버터가 JSON으로 변환해준다.

```java
@ResponseBody
@GetMapping("/response-body-json-v2")
public HelloData responseBodyJson() {
    HelloData helloData = new HelloData();
    helloData.setUsername("userA");
    helloData.setAge(20);
    return helloData;
}
```

`@RestController`를 사용하면 클래스 전체에 `@ResponseBody`가 적용된 것처럼 동작한다.

---

## 12. HTTP 메시지 컨버터

HTTP 메시지 컨버터는 HTTP 요청과 응답의 메시지 바디를 변환하는 역할을 한다.

요청에서는 HTTP 메시지 바디를 읽어 Java 객체로 변환한다.

응답에서는 Java 객체를 HTTP 메시지 바디로 변환한다.

대표적인 메시지 컨버터는 다음과 같다.

- `StringHttpMessageConverter`
- `MappingJackson2HttpMessageConverter`

문자열을 처리할 때는 `StringHttpMessageConverter`가 사용된다.

JSON을 객체로 변환하거나 객체를 JSON으로 변환할 때는 `MappingJackson2HttpMessageConverter`가 사용된다.

요청에서는 `Content-Type`을 기준으로 어떤 컨버터를 사용할지 판단한다.

응답에서는 `Accept` 헤더와 반환 타입을 참고해 컨버터를 선택한다.

---

## 13. 요청 매핑 핸들러 어댑터 구조

Spring MVC에서 애노테이션 기반 컨트롤러를 처리하는 핵심은 `RequestMappingHandlerAdapter`이다.

이 어댑터는 컨트롤러 메서드를 호출하기 전에 필요한 인자를 만들어준다.

예를 들어 다음과 같은 파라미터를 처리할 수 있다.

- `@RequestParam`
- `@ModelAttribute`
- `@RequestBody`
- `HttpServletRequest`
- `HttpServletResponse`
- `@RequestHeader`
- `@CookieValue`

이를 처리하는 것이 ArgumentResolver이다.

컨트롤러 메서드 실행 결과를 응답으로 바꾸는 역할은 ReturnValueHandler가 담당한다.

HTTP 메시지 컨버터는 `@RequestBody`, `@ResponseBody`, `HttpEntity`, `ResponseEntity`와 함께 사용된다.

---

## 14. 웹 페이지 만들기: 상품 관리 예제

상품 관리 예제에서는 다음 기능을 구현한다.

- 상품 목록
- 상품 상세
- 상품 등록
- 상품 수정

상품 도메인은 다음 필드를 가진다.

- 상품 ID
- 상품명
- 가격
- 수량

```java
@Data
public class Item {
    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;
}
```

상품 저장소는 메모리 기반으로 구현한다.

```java
@Repository
public class ItemRepository {
    private static final Map<Long, Item> store = new HashMap<>();
    private static long sequence = 0L;
}
```

실무에서는 동시성 문제를 고려해야 하지만, 학습 예제에서는 단순한 구조로 진행한다.

---

## 15. Thymeleaf와 Model

Spring MVC에서 Controller는 Model에 데이터를 담고 View 이름을 반환한다.

```java
@GetMapping("/basic/items")
public String items(Model model) {
    List<Item> items = itemRepository.findAll();
    model.addAttribute("items", items);
    return "basic/items";
}
```

Thymeleaf는 Model에 담긴 데이터를 사용해 HTML을 동적으로 렌더링한다.

예를 들어 상품 목록을 반복해서 출력할 수 있다.

```html
<tr th:each="item : ${items}">
    <td th:text="${item.id}">1</td>
    <td th:text="${item.itemName}">상품명</td>
    <td th:text="${item.price}">10000</td>
    <td th:text="${item.quantity}">10</td>
</tr>
```

Controller는 데이터를 준비하고, View는 화면 표현에 집중한다.

---

## 16. 상품 상세

상품 상세는 경로 변수로 상품 ID를 받아 해당 상품을 조회한다.

```java
@GetMapping("/basic/items/{itemId}")
public String item(@PathVariable long itemId, Model model) {
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "basic/item";
}
```

`@PathVariable`을 사용하면 `/basic/items/1` 같은 URL에서 `1` 값을 꺼낼 수 있다.

상세 화면에서는 Model에 담긴 `item` 정보를 출력한다.

---

## 17. 상품 등록 폼과 등록 처리

상품 등록 폼은 GET 요청으로 화면을 보여준다.

```java
@GetMapping("/basic/items/add")
public String addForm() {
    return "basic/addForm";
}
```

등록 처리는 POST 요청으로 처리한다.

```java
@PostMapping("/basic/items/add")
public String addItem(@ModelAttribute Item item) {
    itemRepository.save(item);
    return "basic/item";
}
```

`@ModelAttribute`를 사용하면 HTML Form에서 전달된 `itemName`, `price`, `quantity` 값이 `Item` 객체에 자동으로 바인딩된다.

하지만 이렇게 등록 후 바로 View를 반환하면 새로고침 시 POST 요청이 다시 발생해 중복 등록될 수 있다.

이 문제를 해결하기 위해 PRG 패턴을 사용한다.

---

## 18. PRG 패턴

PRG는 Post/Redirect/Get의 약자이다.

상품 등록 같은 POST 요청을 처리한 뒤 바로 View를 반환하지 않고, redirect로 GET 요청을 다시 보내게 하는 방식이다.

```java
@PostMapping("/basic/items/add")
public String addItem(@ModelAttribute Item item) {
    Item savedItem = itemRepository.save(item);
    return "redirect:/basic/items/" + savedItem.getId();
}
```

이렇게 하면 브라우저의 마지막 요청은 GET 요청이 된다.

따라서 사용자가 새로고침을 해도 상품 등록 POST가 반복되지 않는다.

PRG 패턴은 등록, 수정, 삭제 같은 변경 요청 이후 중복 처리를 막기 위해 중요하다.

---

## 19. RedirectAttributes

redirect URL을 문자열로 직접 조합하면 인코딩 문제나 가독성 문제가 생길 수 있다.

`RedirectAttributes`를 사용하면 redirect에 필요한 값을 더 안전하게 전달할 수 있다.

```java
@PostMapping("/basic/items/add")
public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes) {
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/basic/items/{itemId}";
}
```

이 경우 `{itemId}`는 path variable처럼 사용되고, 나머지 값은 쿼리 파라미터로 붙을 수 있다.

예를 들어 다음과 같은 URL이 만들어질 수 있다.

```text
/basic/items/3?status=true
```

상세 화면에서는 `status=true` 여부에 따라 저장 완료 메시지를 보여줄 수 있다.

---

## 20. 상품 수정

상품 수정도 GET과 POST로 나누어 처리한다.

GET 요청은 수정 폼을 보여준다.

```java
@GetMapping("/basic/items/{itemId}/edit")
public String editForm(@PathVariable Long itemId, Model model) {
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "basic/editForm";
}
```

POST 요청은 수정 내용을 저장한다.

```java
@PostMapping("/basic/items/{itemId}/edit")
public String edit(@PathVariable Long itemId, @ModelAttribute Item item) {
    itemRepository.update(itemId, item);
    return "redirect:/basic/items/{itemId}";
}
```

수정 후에도 redirect를 사용해 상세 화면으로 이동한다.

---

## 21. 오늘 정리

Spring MVC 기본 기능은 HTTP 요청과 응답을 더 편리하게 처리하기 위한 기능들의 모음이다.

중요한 구분은 다음과 같다.

- URL 경로 값: `@PathVariable`
- 쿼리 파라미터, HTML Form 단일 값: `@RequestParam`
- 쿼리 파라미터, HTML Form 객체 바인딩: `@ModelAttribute`
- HTTP 메시지 바디, JSON 요청: `@RequestBody`
- HTTP 메시지 바디 직접 응답: `@ResponseBody`
- 객체와 JSON 변환: HTTP 메시지 컨버터
- View 렌더링: Controller가 View 이름 반환
- 변경 요청 후 중복 방지: PRG 패턴

상품 관리 예제를 통해 Spring MVC의 기본 흐름을 실제 웹 페이지에 적용하는 방법을 확인했다.

앞으로 Spring MVC를 사용할 때는 단순히 애노테이션을 외우기보다, 요청 데이터가 어디에 담겨 있는지와 응답을 어떤 방식으로 반환할지 기준으로 선택해야겠다.
