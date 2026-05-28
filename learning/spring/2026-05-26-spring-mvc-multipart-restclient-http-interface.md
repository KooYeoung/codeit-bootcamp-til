# Spring MVC Multipart, RestClient, HTTP Interface 정리

## 학습 날짜

2026-05-26

## 학습 주제

- Multipart 요청
- `multipart/form-data`
- Servlet 파일 업로드
- Spring `MultipartFile`
- `@RequestParam`
- `@ModelAttribute`
- `@RequestPart`
- Multipart Process
- RestClient
- HTTP Interface
- 외부 API 호출 보충

---

## 1. Multipart가 필요한 이유

일반 HTML Form 전송은 보통 `application/x-www-form-urlencoded` 형식을 사용한다.

이 방식은 텍스트 기반의 key-value 데이터를 전송하는 데 적합하다.

하지만 파일은 바이너리 데이터이다.

또한 파일 업로드에서는 일반 텍스트 필드와 파일 데이터를 함께 보내야 하는 경우가 많다.

예를 들어 다음 데이터를 한 요청으로 보낼 수 있다.

- username
- title
- description
- file

이럴 때 `multipart/form-data` 형식을 사용한다.

---

## 2. multipart/form-data 구조

Multipart 요청은 본문이 여러 part로 나뉜다.

각 part는 자신의 Header와 Body를 가진다.

예를 들어 다음과 같은 구조이다.

```text
part 1: username=alice
part 2: file=spring.png
part 3: metadata={...json...}
```

각 part는 boundary로 구분된다.

서버는 이 boundary를 기준으로 요청 본문을 나누어 텍스트 필드와 파일 필드를 구분한다.

---

## 3. Servlet 방식 파일 업로드

Servlet에서는 `Part` 또는 `HttpServletRequest`를 통해 파일 업로드를 처리할 수 있다.

```java
Collection<Part> parts = request.getParts();
```

각 Part에서 파일 이름, Content-Type, 크기, InputStream 등을 확인할 수 있다.

하지만 Servlet API를 직접 사용하면 파일 처리 코드가 길어지고 반복될 수 있다.

Spring MVC는 이를 더 편하게 처리할 수 있도록 `MultipartFile`을 제공한다.

---

## 4. MultipartFile

Spring MVC에서 업로드 파일은 `MultipartFile`로 받을 수 있다.

```java
@PostMapping("/upload")
public String upload(@RequestParam MultipartFile file) throws IOException {
    String originalFilename = file.getOriginalFilename();
    file.transferTo(new File("/upload/" + originalFilename));
    return "ok";
}
```

`MultipartFile`의 주요 기능은 다음과 같다.

- 원본 파일명 조회
- 파일 크기 조회
- Content-Type 조회
- 파일 내용 읽기
- 지정한 위치로 저장

---

## 5. @RequestParam MultipartFile

단순히 파일 하나 또는 여러 개를 받을 때는 `@RequestParam`을 사용할 수 있다.

```java
@PostMapping("/files")
public String upload(@RequestParam("file") MultipartFile file) {
    return "ok";
}
```

여러 파일은 `List<MultipartFile>`로 받을 수 있다.

```java
@PostMapping("/files")
public String upload(@RequestParam List<MultipartFile> files) {
    return "ok";
}
```

---

## 6. @ModelAttribute와 파일 업로드

일반 Form 필드와 파일을 함께 객체로 받고 싶으면 `@ModelAttribute`를 사용할 수 있다.

```java
public class UploadForm {
    private String title;
    private MultipartFile file;
}
```

```java
@PostMapping("/upload")
public String upload(@ModelAttribute UploadForm form) {
    return "ok";
}
```

HTML Form 기반 파일 업로드에서는 이 방식이 자연스럽다.

---

## 7. @RequestPart

`@RequestPart`는 Multipart 요청의 특정 part를 읽을 때 사용한다.

특히 JSON part와 파일 part를 함께 받는 API에서 유용하다.

```java
@PostMapping("/api/upload")
public ResponseEntity<String> upload(
        @RequestPart("metadata") UploadMetadata metadata,
        @RequestPart("file") MultipartFile file
) {
    return ResponseEntity.ok("ok");
}
```

`@RequestPart`의 JSON part는 HttpMessageConverter를 통해 객체로 변환될 수 있다.

따라서 API 방식의 multipart 처리에서는 `@RequestPart`가 적합한 경우가 많다.

---

## 8. 파일 저장 시 주의사항

업로드 파일을 저장할 때는 원본 파일명을 그대로 서버 파일명으로 사용하지 않는 것이 좋다.

문제는 다음과 같다.

- 파일명 충돌
- 경로 조작 위험
- 특수문자 문제
- 운영체제별 파일명 제한

보통은 서버 저장 파일명을 UUID 등으로 새로 만든다.

```text
원본 파일명: profile.png
저장 파일명: 550e8400-e29b-41d4-a716-446655440000.png
```

DB에는 원본 파일명과 저장 파일명을 함께 보관할 수 있다.

---

## 9. RestClient

RestClient는 Spring 6.1부터 사용할 수 있는 동기 HTTP 클라이언트이다.

기존 RestTemplate보다 현대적인 체이닝 API를 제공한다.

```java
RestClient restClient = RestClient.create("https://api.example.com");

User user = restClient.get()
        .uri("/users/{id}", 1)
        .retrieve()
        .body(User.class);
```

RestClient는 외부 API를 호출하고 응답을 객체로 변환하는 데 사용할 수 있다.

---

## 10. RestClient 요청 설정

RestClient에서는 HTTP Method, URL, Header, Body를 체이닝 방식으로 설정할 수 있다.

```java
User created = restClient.post()
        .uri("/users")
        .contentType(MediaType.APPLICATION_JSON)
        .body(new CreateUserRequest("kim"))
        .retrieve()
        .body(User.class);
```

GET 요청에서는 보통 Body를 사용하지 않고, POST/PUT/PATCH 요청에서 Body를 사용한다.

Accept 헤더를 통해 원하는 응답 형식을 지정할 수도 있다.

---

## 11. RestClient 오류 처리

RestClient의 `retrieve()`는 응답 상태 코드가 오류일 때 예외를 발생시킬 수 있다.

세밀한 오류 처리가 필요하면 상태 코드별 처리 로직을 작성할 수 있다.

```java
restClient.get()
        .uri("/users/{id}", id)
        .retrieve()
        .onStatus(HttpStatusCode::is4xxClientError, (request, response) -> {
            throw new IllegalArgumentException("client error");
        })
        .body(User.class);
```

더 세밀하게 응답 전체를 다루고 싶으면 `exchange()`를 사용할 수 있다.

---

## 12. HTTP Interface

HTTP Interface는 Java 인터페이스에 HTTP 호출 정보를 선언하고, 프록시 객체로 외부 API를 호출하는 방식이다.

```java
public interface UserClient {
    @GetExchange("/users/{id}")
    User findById(@PathVariable Long id);
}
```

이 방식은 반복되는 RestClient 호출 코드를 줄이고, 외부 API 계약을 인터페이스로 표현할 수 있다는 장점이 있다.

---

## 13. RestClient와 HTTP Interface 차이

| 구분 | RestClient | HTTP Interface |
|---|---|---|
| 방식 | 직접 호출 코드 작성 | 인터페이스 선언 기반 |
| 장점 | 세밀한 제어가 쉽다 | 반복 API 호출 코드 감소 |
| 적합한 경우 | 단순 호출, 세부 오류 처리 | 외부 API가 많고 계약이 명확한 경우 |
| 형태 | 체이닝 API | 프록시 기반 선언형 API |

둘 중 하나만 정답은 아니다.

외부 API 호출이 단순하면 RestClient를 직접 사용하고, 같은 외부 API를 여러 곳에서 반복 호출한다면 HTTP Interface를 고려할 수 있다.

---

## 14. 보충 정리

파일 업로드와 외부 API 호출은 실무에서 자주 등장하지만, 보안과 안정성을 함께 고려해야 한다.

파일 업로드에서는 다음을 확인해야 한다.

- 파일 크기 제한
- 허용 확장자
- Content-Type 검증
- 저장 경로 분리
- 원본 파일명 신뢰 금지
- 다운로드 시 권한 확인

외부 API 호출에서는 다음을 확인해야 한다.

- timeout 설정
- 오류 응답 처리
- 재시도 정책
- 로그 마스킹
- 장애 격리
- 응답 DTO 분리

---

## 15. 오늘 정리

Multipart와 RestClient는 웹 애플리케이션에서 입출력 경계에 해당하는 중요한 기능이다.

중요한 정리는 다음과 같다.

- 파일 업로드는 `multipart/form-data`를 사용한다.
- Multipart 요청은 여러 part로 나뉜다.
- Spring MVC에서는 `MultipartFile`로 파일을 편리하게 받을 수 있다.
- 단순 파일은 `@RequestParam MultipartFile`로 받을 수 있다.
- Form 객체와 파일은 `@ModelAttribute`로 함께 받을 수 있다.
- JSON part와 파일 part를 함께 받을 때는 `@RequestPart`가 유용하다.
- 업로드 파일은 원본 파일명과 저장 파일명을 분리하는 것이 안전하다.
- RestClient는 동기 HTTP 클라이언트이다.
- HTTP Interface는 외부 API 호출을 인터페이스로 선언하는 방식이다.
- 외부 API 호출은 오류 처리, timeout, 응답 DTO 설계를 함께 고려해야 한다.
