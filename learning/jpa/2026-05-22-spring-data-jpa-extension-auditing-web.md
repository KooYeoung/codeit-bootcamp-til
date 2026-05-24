# Spring Data JPA 확장 기능: 사용자 정의 리포지토리, Auditing, Web 확장 정리

## 학습 날짜

2026-05-22

## 학습 주제

- 사용자 정의 리포지토리
- 사용자 정의 구현 최신 방식
- Querydsl 연동 기준
- Auditing
- `@CreatedDate`, `@LastModifiedDate`
- `@CreatedBy`, `@LastModifiedBy`
- `AuditorAware`
- 도메인 클래스 컨버터
- Web 페이징과 정렬
- Pageable 기본값 설정

---

## 1. 사용자 정의 리포지토리가 필요한 이유

Spring Data JPA는 인터페이스만 정의하면 기본 CRUD 구현체를 자동으로 만들어준다.

하지만 모든 데이터 접근 요구사항을 공통 인터페이스만으로 해결할 수는 없다.

다음과 같은 경우 직접 구현이 필요할 수 있다.

- 복잡한 동적 쿼리
- Querydsl 사용
- EntityManager 직접 사용
- Spring JdbcTemplate 사용
- MyBatis 사용
- 특정 DB 기능 사용
- 성능 최적화를 위한 별도 조회 로직

이때 사용자 정의 리포지토리를 사용할 수 있다.

---

## 2. 사용자 정의 리포지토리 구조

먼저 사용자 정의 인터페이스를 만든다.

```java
public interface MemberRepositoryCustom {

    List<Member> findMemberCustom();
}
```

그 다음 구현 클래스를 만든다.

```java
@RequiredArgsConstructor
public class MemberRepositoryCustomImpl implements MemberRepositoryCustom {

    private final EntityManager em;

    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
}
```

마지막으로 Spring Data JPA 리포지토리에서 사용자 정의 인터페이스를 함께 상속한다.

```java
public interface MemberRepository
        extends JpaRepository<Member, Long>, MemberRepositoryCustom {
}
```

이제 `memberRepository.findMemberCustom()`을 호출할 수 있다.

---

## 3. 구현 클래스 이름 규칙

기존 방식은 리포지토리 인터페이스 이름에 `Impl`을 붙이는 방식이었다.

```text
MemberRepository + Impl
= MemberRepositoryImpl
```

최신 방식으로는 사용자 정의 인터페이스 이름에 `Impl`을 붙이는 방식도 지원한다.

```text
MemberRepositoryCustom + Impl
= MemberRepositoryCustomImpl
```

최신 방식이 더 직관적이다.

이유는 다음과 같다.

- 사용자 정의 인터페이스와 구현체 이름이 대응된다.
- 여러 사용자 정의 인터페이스를 분리하기 쉽다.
- 리포지토리 본체 이름에 구현 클래스가 강하게 묶이지 않는다.

따라서 실무에서는 `MemberRepositoryCustomImpl`처럼 작성하는 방식이 더 좋다.

---

## 4. 사용자 정의 리포지토리 사용 기준

모든 추가 기능을 반드시 사용자 정의 리포지토리로 만들 필요는 없다.

별도의 조회 전용 리포지토리 클래스를 만들어도 된다.

```java
@Repository
@RequiredArgsConstructor
public class MemberQueryRepository {

    private final EntityManager em;
}
```

이 방식은 Spring Data JPA 사용자 정의 기능과 직접 관련은 없지만, 복잡한 조회를 분리하기에는 충분히 좋은 구조이다.

실무 기준으로는 다음처럼 선택할 수 있다.

- 특정 리포지토리에 자연스럽게 붙는 기능: 사용자 정의 리포지토리
- 화면/API 전용 복잡한 조회: 별도 QueryRepository
- 동적 쿼리 중심: Querydsl 기반 사용자 정의 리포지토리 또는 QueryRepository

---

## 5. Auditing이 필요한 이유

엔티티를 운영하다 보면 다음 정보를 추적해야 하는 경우가 많다.

- 등록일
- 수정일
- 등록자
- 수정자

이 필드는 대부분의 엔티티에 공통으로 필요하다.

매번 각 엔티티에 직접 작성하면 중복이 많아진다.

JPA 이벤트나 Spring Data JPA Auditing을 사용하면 이런 공통 값을 자동으로 채울 수 있다.

---

## 6. 순수 JPA 이벤트 방식

순수 JPA에서는 `@PrePersist`, `@PreUpdate`를 사용할 수 있다.

```java
@MappedSuperclass
@Getter
public class JpaBaseEntity {

    @Column(updatable = false)
    private LocalDateTime createdDate;

    private LocalDateTime updatedDate;

    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createdDate = now;
        updatedDate = now;
    }

    @PreUpdate
    public void preUpdate() {
        updatedDate = LocalDateTime.now();
    }
}
```

엔티티가 저장되기 전에 `@PrePersist`가 호출되고, 수정되기 전에 `@PreUpdate`가 호출된다.

이 방식은 JPA 표준 기능이다.

---

## 7. Spring Data JPA Auditing 설정

Spring Data JPA Auditing을 사용하려면 설정 클래스에 `@EnableJpaAuditing`을 추가해야 한다.

```java
@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {
}
```

공통 엔티티에는 `@EntityListeners(AuditingEntityListener.class)`를 붙인다.

```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseTimeEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}
```

이제 엔티티 저장 시 등록 시간이 자동으로 들어가고, 수정 시 수정 시간이 자동으로 갱신된다.

---

## 8. 등록자와 수정자

등록자와 수정자를 자동으로 넣으려면 다음 애노테이션을 사용한다.

```java
@CreatedBy
@Column(updatable = false)
private String createdBy;

@LastModifiedBy
private String lastModifiedBy;
```

그리고 현재 사용자를 제공하는 `AuditorAware` 빈이 필요하다.

```java
@Bean
public AuditorAware<String> auditorProvider() {
    return () -> Optional.of(UUID.randomUUID().toString());
}
```

실무에서는 랜덤 값이 아니라 로그인 사용자 ID를 반환해야 한다.

예를 들어 Spring Security를 사용한다면 SecurityContext에서 사용자 정보를 가져올 수 있다.

---

## 9. BaseTimeEntity와 BaseEntity 분리

실무에서는 등록일과 수정일은 거의 모든 엔티티에 필요하다.

하지만 등록자와 수정자는 필요하지 않을 수도 있다.

그래서 다음처럼 분리하면 좋다.

```java
@MappedSuperclass
@Getter
public class BaseTimeEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}
```

```java
@MappedSuperclass
@Getter
public class BaseEntity extends BaseTimeEntity {

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}
```

시간 정보만 필요한 엔티티는 `BaseTimeEntity`를 상속하고, 사용자 추적까지 필요한 엔티티는 `BaseEntity`를 상속하면 된다.

---

## 10. 도메인 클래스 컨버터

도메인 클래스 컨버터는 HTTP 요청 파라미터로 넘어온 엔티티 ID를 실제 엔티티 객체로 변환해준다.

사용 전 코드는 다음과 같다.

```java
@GetMapping("/members/{id}")
public String findMember(@PathVariable("id") Long id) {
    Member member = memberRepository.findById(id).get();
    return member.getUsername();
}
```

사용 후에는 다음처럼 받을 수 있다.

```java
@GetMapping("/members/{id}")
public String findMember(@PathVariable("id") Member member) {
    return member.getUsername();
}
```

요청은 ID로 들어오지만, Spring Data JPA가 리포지토리를 통해 엔티티를 찾아서 파라미터로 넣어준다.

---

## 11. 도메인 클래스 컨버터 주의사항

도메인 클래스 컨버터로 받은 엔티티는 단순 조회용으로만 사용하는 것이 좋다.

이유는 트랜잭션이 없는 범위에서 조회될 수 있기 때문이다.

컨트롤러에서 엔티티를 직접 수정해도 변경 감지가 동작하지 않을 수 있다.

또한 컨트롤러가 엔티티에 직접 의존하면 API 계층과 도메인 계층의 결합이 강해질 수 있다.

실무에서는 다음 방식을 더 권장한다.

- 컨트롤러에서는 ID를 받는다.
- 서비스 계층에서 트랜잭션 안에서 엔티티를 조회한다.
- 변경은 서비스 계층에서 처리한다.
- 응답은 DTO로 반환한다.

---

## 12. Web 페이징과 정렬

Spring Data는 컨트롤러 파라미터로 `Pageable`을 받을 수 있게 지원한다.

```java
@GetMapping("/members")
public Page<Member> list(Pageable pageable) {
    return memberRepository.findAll(pageable);
}
```

요청 파라미터는 다음처럼 사용할 수 있다.

```text
/members?page=0&size=3&sort=id,desc&sort=username,desc
```

의미는 다음과 같다.

- `page`: 현재 페이지 번호, 0부터 시작
- `size`: 한 페이지 크기
- `sort`: 정렬 조건

정렬 조건은 여러 개 지정할 수 있다.

---

## 13. Pageable 기본값 설정

전체 기본값은 설정 파일에서 지정할 수 있다.

```yaml
spring:
  data:
    web:
      pageable:
        default-page-size: 20
        max-page-size: 2000
```

개별 컨트롤러 메서드에서는 `@PageableDefault`를 사용할 수 있다.

```java
@GetMapping("/members")
public Page<Member> list(
        @PageableDefault(size = 12, sort = "username", direction = Sort.Direction.DESC)
        Pageable pageable
) {
    return memberRepository.findAll(pageable);
}
```

페이지 번호는 기본적으로 0부터 시작한다.

사용자 화면에서는 1페이지부터 보여주고 싶을 수 있으므로, API 스펙에서 0 기반인지 1 기반인지 명확히 정해야 한다.

---

## 14. Page를 그대로 API 응답으로 반환할 때 주의

컨트롤러에서 `Page<Member>`를 그대로 반환하면 엔티티가 API 응답으로 노출될 수 있다.

```java
@GetMapping("/members")
public Page<Member> list(Pageable pageable) {
    return memberRepository.findAll(pageable);
}
```

실무에서는 엔티티를 직접 반환하기보다 DTO로 변환하는 것이 좋다.

```java
@GetMapping("/members")
public Page<MemberDto> list(Pageable pageable) {
    return memberRepository.findAll(pageable)
            .map(member -> new MemberDto(member.getId(), member.getUsername()));
}
```

이렇게 하면 페이징 메타데이터는 유지하면서 API 응답 스펙은 DTO로 분리할 수 있다.

---

## 15. 보충 정리

확장 기능을 사용할 때는 편리함과 계층 분리를 함께 고려해야 한다.

- 사용자 정의 리포지토리는 Querydsl이나 복잡한 조회를 붙일 때 유용하다.
- 모든 조회를 무조건 사용자 정의 리포지토리로 넣기보다 QueryRepository를 분리해도 된다.
- Auditing은 공통 생성/수정 정보를 자동화하는 기능이다.
- 등록자/수정자는 로그인 사용자 정보와 연결해야 실무적으로 의미가 있다.
- 도메인 클래스 컨버터는 편리하지만 컨트롤러에서 엔티티를 직접 다루게 만들 수 있으므로 주의한다.
- Pageable을 컨트롤러에서 바로 받을 수 있지만, 응답은 DTO로 변환하는 것이 좋다.
- 페이지 번호가 0부터 시작한다는 점은 프론트엔드와 API 스펙에서 명확히 맞춰야 한다.

---

## 16. 오늘 정리

Spring Data JPA의 확장 기능은 기본 CRUD를 넘어서 실무에 필요한 기능을 제공한다.

중요한 정리는 다음과 같다.

- 사용자 정의 리포지토리는 Spring Data JPA 기능을 확장할 때 사용한다.
- 최신 방식으로는 사용자 정의 인터페이스명 + `Impl` 구현체가 직관적이다.
- Auditing은 등록일, 수정일, 등록자, 수정자를 자동 관리한다.
- `@EnableJpaAuditing` 설정이 필요하다.
- `AuditorAware`는 현재 사용자를 제공한다.
- 도메인 클래스 컨버터는 단순 조회용으로만 사용하는 것이 좋다.
- Web 페이징은 `Pageable`로 편리하게 처리할 수 있다.
- API 응답에서는 엔티티 대신 DTO를 반환하는 것이 좋다.

앞으로 확장 기능을 사용할 때는 “편리하다”는 이유만으로 바로 적용하지 말고, 트랜잭션 범위, 계층 분리, API 스펙 안정성을 함께 고려해야겠다.
