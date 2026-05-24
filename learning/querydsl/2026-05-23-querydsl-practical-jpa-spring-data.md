# Querydsl 실무 활용: 순수 JPA와 Spring Data JPA 연동 정리

## 학습 날짜

2026-05-23

## 학습 주제

- 순수 JPA 리포지토리와 Querydsl
- `JPAQueryFactory` 빈 등록
- 동적 쿼리와 성능 최적화 조회
- `MemberSearchCondition`
- `MemberTeamDto`
- BooleanBuilder 방식
- where 다중 파라미터 방식
- 조회 API 컨트롤러
- Spring Data JPA 리포지토리와 Querydsl
- 사용자 정의 리포지토리
- Querydsl 페이징
- count 쿼리 최적화

---

## 1. 순수 JPA 리포지토리와 Querydsl

순수 JPA 리포지토리는 `EntityManager`를 직접 사용한다.

```java
@Repository
public class MemberJpaRepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    public MemberJpaRepository(EntityManager em) {
        this.em = em;
        this.queryFactory = new JPAQueryFactory(em);
    }

    public void save(Member member) {
        em.persist(member);
    }

    public Optional<Member> findById(Long id) {
        Member findMember = em.find(Member.class, id);
        return Optional.ofNullable(findMember);
    }

    public List<Member> findAll_Querydsl() {
        return queryFactory
                .selectFrom(member)
                .fetch();
    }

    public List<Member> findByUsername_Querydsl(String username) {
        return queryFactory
                .selectFrom(member)
                .where(member.username.eq(username))
                .fetch();
    }
}
```

JPQL 문자열 대신 Q-Type을 사용하므로 필드명 변경이나 오타에 더 안전하다.

---

## 2. JPAQueryFactory 스프링 빈 등록

`JPAQueryFactory`를 스프링 빈으로 등록해서 주입받아 사용할 수도 있다.

```java
@Bean
JPAQueryFactory jpaQueryFactory(EntityManager em) {
    return new JPAQueryFactory(em);
}
```

이때 동시성 문제를 크게 걱정하지 않아도 된다.

스프링이 주입하는 `EntityManager`는 실제 엔티티 매니저가 아니라 프록시이다.

실제 사용 시점에 트랜잭션 단위로 실제 엔티티 매니저와 영속성 컨텍스트를 연결해준다.

---

## 3. 성능 최적화용 DTO

실무 조회에서는 엔티티 전체를 조회하기보다 화면이나 API에 필요한 데이터만 DTO로 조회하는 경우가 많다.

```java
@Data
public class MemberTeamDto {

    private Long memberId;
    private String username;
    private int age;
    private Long teamId;
    private String teamName;

    @QueryProjection
    public MemberTeamDto(Long memberId, String username, int age, Long teamId, String teamName) {
        this.memberId = memberId;
        this.username = username;
        this.age = age;
        this.teamId = teamId;
        this.teamName = teamName;
    }
}
```

`@QueryProjection`을 사용하면 Querydsl에서 DTO 생성자도 타입 체크를 받을 수 있다.

다만 DTO가 Querydsl에 의존하는 단점이 있다.

---

## 4. 검색 조건 객체

동적 검색 조건은 별도 객체로 받는 것이 좋다.

```java
@Data
public class MemberSearchCondition {

    private String username;
    private String teamName;
    private Integer ageGoe;
    private Integer ageLoe;
}
```

이렇게 하면 검색 조건이 늘어나도 메서드 파라미터가 길어지는 문제를 줄일 수 있다.

컨트롤러에서는 요청 파라미터를 이 객체로 받을 수 있다.

---

## 5. BooleanBuilder 방식

동적 쿼리는 `BooleanBuilder`로 만들 수 있다.

```java
BooleanBuilder builder = new BooleanBuilder();

if (hasText(condition.getUsername())) {
    builder.and(member.username.eq(condition.getUsername()));
}

if (hasText(condition.getTeamName())) {
    builder.and(team.name.eq(condition.getTeamName()));
}

if (condition.getAgeGoe() != null) {
    builder.and(member.age.goe(condition.getAgeGoe()));
}

if (condition.getAgeLoe() != null) {
    builder.and(member.age.loe(condition.getAgeLoe()));
}
```

이 방식은 이해하기 쉽지만 조건이 많아지면 if 문이 늘어나고 재사용성이 떨어질 수 있다.

---

## 6. where 다중 파라미터 방식

where 다중 파라미터 방식은 조건 메서드를 분리한다.

```java
public List<MemberTeamDto> search(MemberSearchCondition condition) {
    return queryFactory
            .select(new QMemberTeamDto(
                    member.id.as("memberId"),
                    member.username,
                    member.age,
                    team.id.as("teamId"),
                    team.name.as("teamName")
            ))
            .from(member)
            .leftJoin(member.team, team)
            .where(
                    usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe())
            )
            .fetch();
}
```

조건 메서드는 다음처럼 작성할 수 있다.

```java
private BooleanExpression usernameEq(String username) {
    return hasText(username) ? member.username.eq(username) : null;
}

private BooleanExpression teamNameEq(String teamName) {
    return hasText(teamName) ? team.name.eq(teamName) : null;
}

private BooleanExpression ageGoe(Integer ageGoe) {
    return ageGoe != null ? member.age.goe(ageGoe) : null;
}

private BooleanExpression ageLoe(Integer ageLoe) {
    return ageLoe != null ? member.age.loe(ageLoe) : null;
}
```

이 방식의 장점은 조건 메서드를 다른 쿼리에서도 재사용할 수 있다는 점이다.

실무에서는 이 방식이 더 깔끔하다.

---

## 7. 조회 API 컨트롤러

검색 조건 객체는 컨트롤러에서 바로 받을 수 있다.

```java
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberJpaRepository memberJpaRepository;

    @GetMapping("/v1/members")
    public List<MemberTeamDto> searchMemberV1(MemberSearchCondition condition) {
        return memberJpaRepository.search(condition);
    }
}
```

요청 예시는 다음과 같다.

```text
/v1/members?username=member1&teamName=teamA&ageGoe=10&ageLoe=35
```

보충하면, 실무 API에서는 엔티티를 직접 반환하지 않고 DTO를 반환하는 것이 좋다.

이 예제도 `MemberTeamDto`를 반환하므로 API 응답 구조를 엔티티와 분리할 수 있다.

---

## 8. Spring Data JPA 리포지토리와 사용자 정의 리포지토리

Spring Data JPA를 사용하면 기본 CRUD와 단순 메서드 쿼리를 쉽게 사용할 수 있다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsername(String username);
}
```

하지만 Querydsl 전용 검색 기능은 `JpaRepository`만으로 표현하기 어렵다.

복잡한 동적 검색이 필요하면 사용자 정의 리포지토리를 추가해야 한다.

사용자 정의 리포지토리 구성은 세 단계로 진행된다.

1. 사용자 정의 인터페이스 작성
2. 사용자 정의 인터페이스 구현
3. Spring Data JPA 리포지토리에 사용자 정의 인터페이스 상속

```java
public interface MemberRepositoryCustom {

    List<MemberTeamDto> search(MemberSearchCondition condition);
}
```

```java
public interface MemberRepository
        extends JpaRepository<Member, Long>, MemberRepositoryCustom {
}
```

이제 기본 CRUD와 Querydsl 검색 기능을 하나의 리포지토리에서 사용할 수 있다.

---

## 9. Querydsl 페이징 연동

Spring Data의 `Pageable`을 Querydsl에 적용할 수 있다.

```java
List<MemberTeamDto> content = queryFactory
        .select(new QMemberTeamDto(
                member.id.as("memberId"),
                member.username,
                member.age,
                team.id.as("teamId"),
                team.name.as("teamName")
        ))
        .from(member)
        .leftJoin(member.team, team)
        .where(
                usernameEq(condition.getUsername()),
                teamNameEq(condition.getTeamName()),
                ageGoe(condition.getAgeGoe()),
                ageLoe(condition.getAgeLoe())
        )
        .offset(pageable.getOffset())
        .limit(pageable.getPageSize())
        .fetch();
```

페이지 번호는 0부터 시작한다.

프론트엔드와 API 스펙에서 0 기반 페이지인지 1 기반 페이지인지 명확히 정해야 한다.

---

## 10. count 쿼리 최적화

최신 Querydsl에서는 content 조회와 count 조회를 분리한다.

```java
JPAQuery<Long> countQuery = queryFactory
        .select(member.count())
        .from(member)
        .leftJoin(member.team, team)
        .where(
                usernameEq(condition.getUsername()),
                teamNameEq(condition.getTeamName()),
                ageGoe(condition.getAgeGoe()),
                ageLoe(condition.getAgeLoe())
        );

return PageableExecutionUtils.getPage(content, pageable, countQuery::fetchOne);
```

`PageableExecutionUtils.getPage()`는 count 쿼리를 생략할 수 있는 경우 생략해준다.

예를 들어 첫 페이지인데 조회된 content 수가 page size보다 작으면 전체 count를 따로 조회할 필요가 없다.

보충하면, 데이터 조회 쿼리에는 조인이 필요하지만 count에는 조인이 필요 없는 경우가 있다.

이런 경우 count 쿼리를 더 단순하게 분리하면 성능 개선이 가능하다.

단, `teamName` 조건처럼 조인된 테이블 조건이 count에도 필요하다면 조인을 제거하면 안 된다.

---

## 11. Querydsl과 Sort

Spring Data의 `Sort`를 Querydsl의 `OrderSpecifier`로 변환할 수 있다.

```java
for (Sort.Order o : pageable.getSort()) {
    PathBuilder pathBuilder = new PathBuilder(
            member.getType(),
            member.getMetadata()
    );

    query.orderBy(new OrderSpecifier(
            o.isAscending() ? Order.ASC : Order.DESC,
            pathBuilder.get(o.getProperty())
    ));
}
```

다만 문자열 기반 property는 런타임 오류 가능성이 있다.

정렬 조건이 복잡하거나 조인 필드 정렬이 필요한 경우에는 명시적인 정렬 조건을 코드로 작성하는 것이 더 안전할 수 있다.

---

## 12. Querydsl 리포지토리 설계 기준

실무에서는 다음처럼 역할을 나누면 좋다.

### Spring Data JPA Repository

- 기본 CRUD
- 단순 메서드 쿼리
- 단순 `@Query`

### Querydsl Custom Repository

- 복잡한 동적 검색
- 페이징 + DTO 조회
- 조건 조합이 많은 조회
- 성능 최적화가 필요한 목록 조회

### 별도 QueryRepository

- 특정 화면이나 API 전용 조회
- 여러 엔티티를 조합하는 복잡한 조회
- 도메인 리포지토리와 분리하고 싶은 읽기 전용 쿼리

모든 Querydsl 코드를 무조건 도메인 리포지토리에 넣기보다, 조회 성격에 따라 분리하는 것이 유지보수에 좋다.

---

## 오늘 정리

Querydsl은 순수 JPA 리포지토리에서도 사용할 수 있고, Spring Data JPA와 함께 사용할 수도 있다.

핵심은 동적 쿼리와 복잡한 조회를 명확하고 타입 안전하게 작성하는 것이다.

중요한 정리는 다음과 같다.

- 순수 JPA 리포지토리에 `JPAQueryFactory`를 추가해 Querydsl을 사용할 수 있다.
- `JPAQueryFactory`는 스프링 빈으로 등록해 주입받을 수 있다.
- DTO 조회는 API 성능 최적화에 유용하다.
- 동적 검색 조건은 `MemberSearchCondition` 같은 객체로 받는 것이 좋다.
- `BooleanBuilder`보다 where 다중 파라미터 방식이 재사용성과 가독성 면에서 좋다.
- Spring Data JPA와 Querydsl을 연결하려면 사용자 정의 리포지토리를 사용한다.
- 페이징은 content 쿼리와 count 쿼리를 분리해서 생각한다.
- Querydsl 5.0에서는 `fetchResults()`, `fetchCount()` 대신 별도 count 쿼리를 작성한다.
- `PageableExecutionUtils`로 count 쿼리를 생략할 수 있는 경우 최적화할 수 있다.
