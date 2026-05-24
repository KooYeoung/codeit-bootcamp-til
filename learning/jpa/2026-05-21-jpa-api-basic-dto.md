# Spring Boot JPA 활용2 - API 개발 기본 정리

## 학습 날짜

2026-05-21

## 학습 주제

- 회원 등록 API
- 회원 수정 API
- 회원 조회 API
- 엔티티 직접 노출 문제
- Request DTO와 Response DTO
- 변경 감지
- 컬렉션 응답 래핑
- Spring Boot 3.x 환경 주의사항

---

## 1. API 개발 기본에서 가장 중요한 기준

API를 만들 때 가장 중요한 기준은 엔티티를 외부 API 스펙으로 직접 사용하지 않는 것이다.

엔티티는 애플리케이션 내부의 핵심 도메인 모델이다.

반면 API 요청과 응답은 외부 클라이언트와 약속한 스펙이다.

이 둘을 같은 객체로 사용하면 다음 문제가 생긴다.

- 엔티티 변경이 API 스펙 변경으로 이어진다.
- API 검증 요구사항이 엔티티에 들어간다.
- 화면이나 API 응답을 위한 `@JsonIgnore` 같은 프레젠테이션 로직이 엔티티에 들어간다.
- 하나의 엔티티에 여러 API의 요청/응답 요구사항을 모두 담기 어렵다.
- 의도하지 않은 필드가 외부에 노출될 수 있다.

따라서 실무에서는 API 요청과 응답에 별도의 DTO를 사용하는 것이 안전하다.

---

## 2. 회원 등록 API V1 - 엔티티를 직접 받는 방식

V1 방식은 요청 본문을 엔티티에 직접 매핑한다.

```java
@PostMapping("/api/v1/members")
public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}
```

이 방식은 코드가 간단해 보인다.

하지만 요청 JSON이 엔티티 구조와 직접 연결된다.

예를 들어 회원 등록 API에서 이름만 필요하더라도, `Member` 엔티티의 모든 구조가 API 요청 스펙에 영향을 줄 수 있다.

또한 `@NotEmpty` 같은 API 검증 요구사항이 엔티티에 들어가게 된다.

이렇게 되면 엔티티가 순수한 도메인 모델이 아니라 API 요청 모델 역할까지 하게 된다.

---

## 3. 회원 등록 API V1의 문제점

엔티티를 요청 DTO로 직접 사용하면 다음 문제가 있다.

## 엔티티에 프레젠테이션 계층 로직이 섞인다

API 요청 검증이나 JSON 직렬화 관련 요구사항이 엔티티에 들어간다.

도메인 모델이 화면이나 API 스펙에 종속된다.

## API마다 요구사항이 다르다

회원 등록 API에서는 이름만 필요할 수 있다.

회원 수정 API에서는 이름과 연락처가 필요할 수 있다.

관리자용 API에서는 더 많은 값이 필요할 수 있다.

하나의 `Member` 엔티티가 모든 API 요구사항을 담기는 어렵다.

## 엔티티 변경이 API 변경으로 이어진다

엔티티에 필드가 추가되거나 이름이 바뀌면 API 요청/응답 스펙도 함께 바뀔 수 있다.

외부 클라이언트 입장에서는 갑자기 API 스펙이 변경되는 문제가 된다.

---

## 4. 회원 등록 API V2 - DTO를 받는 방식

V2 방식은 엔티티 대신 요청 DTO를 받는다.

```java
@Data
static class CreateMemberRequest {
    private String name;
}
```

컨트롤러는 DTO를 받고, 필요한 값으로 엔티티를 생성한다.

```java
@PostMapping("/api/v2/members")
public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
    Member member = new Member();
    member.setName(request.getName());

    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}
```

이 방식은 API 요청 스펙과 엔티티를 분리한다.

요청 스펙이 바뀌어도 엔티티가 반드시 바뀌지는 않고, 엔티티가 바뀌어도 API 스펙이 바로 바뀌지 않는다.

---

## 5. 요청 DTO 사용의 장점

요청 DTO를 사용하면 다음 장점이 있다.

- API 스펙을 명확하게 표현할 수 있다.
- 엔티티와 프레젠테이션 계층을 분리할 수 있다.
- API별 검증 조건을 DTO에 따로 둘 수 있다.
- 엔티티 변경이 외부 API 스펙에 바로 영향을 주지 않는다.
- 클라이언트가 어떤 값을 보내야 하는지 명확해진다.

예를 들어 등록 요청과 수정 요청을 따로 만들 수 있다.

```java
@Data
class CreateMemberRequest {
    private String name;
}

@Data
class UpdateMemberRequest {
    private String name;
}
```

현재는 필드가 같더라도 API 목적이 다르면 DTO를 분리하는 것이 나중에 유지보수에 유리하다.

---

## 6. 회원 수정 API

회원 수정 API도 엔티티 대신 DTO를 받는다.

```java
@PatchMapping("/api/v2/members/{id}")
public UpdateMemberResponse updateMemberV2(
        @PathVariable("id") Long id,
        @RequestBody @Valid UpdateMemberRequest request) {

    memberService.update(id, request.getName());
    Member findMember = memberService.findOne(id);

    return new UpdateMemberResponse(findMember.getId(), findMember.getName());
}
```

강의 자료에서는 `PUT`을 사용하지만, 이름만 변경하는 부분 수정이라면 REST 관점에서는 `PATCH` 또는 `POST`가 더 적절할 수 있다.

`PUT`은 보통 전체 교체 의미가 강하고, `PATCH`는 일부 필드 변경 의미가 강하다.

---

## 7. 변경 감지로 수정하기

회원 수정 서비스는 영속 상태의 엔티티를 조회한 뒤 값을 변경한다.

```java
@Transactional
public void update(Long id, String name) {
    Member member = memberRepository.findOne(id);
    member.setName(name);
}
```

여기서 별도의 `save()`를 호출하지 않는다.

트랜잭션 안에서 조회한 `Member`는 영속 상태이다.

영속 상태의 엔티티 값을 변경하면 트랜잭션 커밋 시점에 JPA가 변경 감지를 통해 UPDATE SQL을 실행한다.

이것을 dirty checking이라고 한다.

실무에서는 수정 시 `merge()`보다 변경 감지를 사용하는 방식이 더 명확하다.

---

## 8. 수정 API 응답

수정 후 응답 DTO를 따로 반환한다.

```java
@Data
@AllArgsConstructor
static class UpdateMemberResponse {
    private Long id;
    private String name;
}
```

응답 DTO를 사용하면 클라이언트에게 필요한 값만 내려줄 수 있다.

또한 엔티티 내부 구조가 외부로 노출되지 않는다.

---

## 9. 회원 조회 API V1 - 엔티티 직접 노출

회원 조회 V1은 엔티티를 직접 반환한다.

```java
@GetMapping("/api/v1/members")
public List<Member> membersV1() {
    return memberService.findMembers();
}
```

이 방식은 좋지 않다.

문제는 다음과 같다.

- 엔티티의 모든 값이 노출될 수 있다.
- 응답 스펙을 맞추기 위해 엔티티에 `@JsonIgnore` 같은 설정이 들어갈 수 있다.
- 엔티티가 바뀌면 API 응답도 바뀐다.
- 컬렉션을 바로 반환하면 나중에 응답 구조 확장이 어렵다.
- 양방향 연관관계가 있으면 JSON 직렬화 무한 루프가 발생할 수 있다.

실무에서는 엔티티를 API 응답으로 직접 노출하면 안 된다.

---

## 10. 회원 조회 API V2 - DTO 반환

회원 조회 V2는 엔티티를 DTO로 변환해서 반환한다.

```java
@GetMapping("/api/v2/members")
public Result membersV2() {
    List<Member> findMembers = memberService.findMembers();

    List<MemberDto> collect = findMembers.stream()
            .map(m -> new MemberDto(m.getName()))
            .collect(Collectors.toList());

    return new Result(collect);
}
```

회원 응답 DTO는 다음처럼 만들 수 있다.

```java
@Data
@AllArgsConstructor
static class MemberDto {
    private String name;
}
```

이렇게 하면 응답으로 필요한 값만 선택해서 내려줄 수 있다.

---

## 11. 컬렉션 응답을 Result로 감싸는 이유

컬렉션을 바로 반환할 수도 있다.

```java
return collect;
```

하지만 이렇게 하면 최상위 응답이 배열이 된다.

나중에 count나 meta 정보를 추가하고 싶을 때 API 스펙을 바꾸기 어렵다.

그래서 다음처럼 래퍼 객체로 감싸는 것이 좋다.

```java
@Data
@AllArgsConstructor
static class Result<T> {
    private T data;
}
```

응답 예시는 다음처럼 만들 수 있다.

```json
{
  "data": [
    { "name": "userA" },
    { "name": "userB" }
  ]
}
```

나중에 count를 추가하기도 쉽다.

```json
{
  "count": 2,
  "data": [
    { "name": "userA" },
    { "name": "userB" }
  ]
}
```

---

## 12. Spring Boot 3.x 환경 주의사항

강의 자료는 Spring Boot 3.x 사용 시 다음 내용을 확인하라고 안내한다.

- Java 17 이상 사용
- `javax` 패키지 대신 `jakarta` 패키지 사용
- H2 데이터베이스 2.1.214 이상 사용

예를 들어 JPA 애노테이션 패키지는 다음처럼 변경된다.

```java
// 과거
import javax.persistence.Entity;

// Spring Boot 3.x
import jakarta.persistence.Entity;
```

검증 애노테이션도 변경된다.

```java
// 과거
import javax.validation.Valid;

// Spring Boot 3.x
import jakarta.validation.Valid;
```

이 부분은 예전 강의 코드나 블로그 코드를 따라 하다가 자주 막히는 부분이므로 주의해야 한다.

---

## 13. 보충: DTO는 어디까지 나누어야 할까?

DTO는 API 스펙에 맞춰 분리하는 것이 기본이다.

다만 너무 많은 DTO가 생기면 관리가 어려울 수 있다.

처음에는 다음 기준으로 나누면 좋다.

- 등록 요청 DTO
- 수정 요청 DTO
- 단건 조회 응답 DTO
- 목록 조회 응답 DTO
- 상세 조회 응답 DTO

요청 DTO와 응답 DTO는 역할이 다르므로 분리하는 것이 좋다.

요청 DTO는 클라이언트가 보내는 값을 검증하는 데 집중한다.

응답 DTO는 클라이언트에게 보여줄 값을 구성하는 데 집중한다.

---

## 14. 보충: Entity, DTO, Service의 역할

각 객체의 역할은 다음처럼 나누어 생각할 수 있다.

## Entity

- 핵심 도메인 상태와 행위를 가진다.
- JPA가 관리한다.
- DB 테이블과 매핑된다.
- API 스펙에 종속되지 않아야 한다.

## Request DTO

- API 요청 스펙을 표현한다.
- 요청 값 검증을 담당한다.
- 엔티티와 직접 동일할 필요가 없다.

## Response DTO

- API 응답 스펙을 표현한다.
- 클라이언트에게 필요한 값만 포함한다.
- 엔티티 내부 구조를 숨긴다.

## Service

- 비즈니스 로직을 처리한다.
- 트랜잭션 경계를 관리한다.
- 엔티티 변경은 변경 감지를 활용한다.

---

## 15. 오늘 정리

API 개발 기본의 핵심은 엔티티와 API 스펙을 분리하는 것이다.

엔티티는 내부 도메인 모델이고, DTO는 외부와 주고받는 계약이다.

중요한 정리는 다음과 같다.

- 요청 값으로 엔티티를 직접 받지 않는다.
- 응답 값으로 엔티티를 직접 반환하지 않는다.
- API 스펙에 맞는 DTO를 만든다.
- 수정은 트랜잭션 안에서 엔티티를 조회한 뒤 변경 감지를 사용한다.
- 부분 수정은 `PATCH` 또는 `POST`를 고려한다.
- 컬렉션 응답은 Result로 감싸면 확장성이 좋아진다.
- Spring Boot 3.x에서는 Java 17, jakarta 패키지, H2 2.x 이상을 확인해야 한다.

앞으로 API를 만들 때는 먼저 엔티티를 그대로 노출하고 싶어도, 유지보수와 API 안정성을 위해 DTO를 기준으로 설계해야겠다.
