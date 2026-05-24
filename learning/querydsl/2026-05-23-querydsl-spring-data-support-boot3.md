# Spring Data JPA가 제공하는 Querydsl 기능과 Querydsl 5.0 보충 정리

## 학습 날짜

2026-05-23

## 학습 주제

- `QuerydslPredicateExecutor`
- Querydsl Web 지원
- `QuerydslRepositorySupport`
- Querydsl 지원 클래스 직접 만들기
- Spring Boot 3.x Querydsl 5.0
- `PageableExecutionUtils` 패키지 변경
- `fetchResults()`, `fetchCount()` Deprecated
- count 쿼리 분리
- 실무 적용 기준

---

## 1. Spring Data JPA가 제공하는 Querydsl 기능

Spring Data JPA는 Querydsl과 관련된 몇 가지 편의 기능을 제공한다.

대표적인 기능은 다음과 같다.

- `QuerydslPredicateExecutor`
- Querydsl Web 지원
- `QuerydslRepositorySupport`

이 기능들은 간단한 상황에서는 편리하다.

하지만 복잡한 실무 환경에서는 제약이 많으므로 한계를 이해하고 사용하는 것이 중요하다.

---

## 2. QuerydslPredicateExecutor

`QuerydslPredicateExecutor`는 Predicate를 사용해 Querydsl 조건을 리포지토리에서 바로 사용할 수 있게 해준다.

```java
public interface QuerydslPredicateExecutor<T> {

    Optional<T> findById(Predicate predicate);

    Iterable<T> findAll(Predicate predicate);

    long count(Predicate predicate);

    boolean exists(Predicate predicate);
}
```

리포지토리에 적용하면 다음과 같다.

```java
interface MemberRepository
        extends JpaRepository<Member, Long>,
                QuerydslPredicateExecutor<Member> {
}
```

사용 예시는 다음과 같다.

```java
Iterable<Member> result = memberRepository.findAll(
        member.age.between(10, 40)
                .and(member.username.eq("member1"))
);
```

간단한 조건 조회에는 사용할 수 있다.

---

## 3. QuerydslPredicateExecutor 한계

`QuerydslPredicateExecutor`는 실무에서 한계가 명확하다.

주요 한계는 다음과 같다.

- left join이 어렵다.
- 묵시적 조인은 가능하지만 명시적 조인 제어가 어렵다.
- 클라이언트 코드가 Querydsl Predicate에 의존한다.
- 서비스 계층이 Querydsl이라는 구현 기술을 알게 된다.
- DTO 조회나 복잡한 페이징 최적화에 적합하지 않다.
- 복잡한 검색 조건을 명확하게 표현하기 어렵다.

보충하면, 서비스 계층은 가능하면 비즈니스 의도를 표현해야 한다.

서비스에서 `member.age.between(...)` 같은 Querydsl 조건을 직접 작성하면 데이터 접근 기술이 서비스로 새어나올 수 있다.

---

## 4. Querydsl Web 지원

Querydsl Web 지원은 웹 요청 파라미터를 Predicate로 바꿔주는 기능이다.

간단한 조건에서는 편리할 수 있다.

하지만 한계도 크다.

- 단순한 조건만 처리하기 쉽다.
- 조건 커스터마이징이 복잡하다.
- 컨트롤러가 Querydsl에 의존한다.
- API 스펙과 Querydsl 내부 구조가 섞일 수 있다.
- 복잡한 실무 검색 조건에는 적합하지 않다.

실무에서는 컨트롤러가 Querydsl Predicate를 직접 받기보다, 검색 조건 DTO를 받는 방식이 더 명확하다.

```java
public Page<MemberTeamDto> search(MemberSearchCondition condition, Pageable pageable) {
    return memberRepository.searchPageComplex(condition, pageable);
}
```

---

## 5. QuerydslRepositorySupport

`QuerydslRepositorySupport`는 Spring Data JPA가 제공하는 Querydsl 지원 클래스이다.

장점은 다음과 같다.

- `getQuerydsl().applyPagination()`으로 페이징을 Querydsl에 적용할 수 있다.
- `EntityManager`를 제공한다.
- `from()`으로 쿼리를 시작할 수 있다.

하지만 한계가 있다.

- Querydsl 3.x 버전을 대상으로 만들어졌다.
- Querydsl 4.x 이후의 `JPAQueryFactory` 중심 흐름과 맞지 않는다.
- `select()`로 시작할 수 없다.
- `from()`으로 시작해야 한다.
- QueryFactory를 제공하지 않는다.
- Spring Data Sort 기능이 정상 동작하지 않는 문제가 있다.

따라서 최신 Querydsl 실무 코드에서는 직접 지원 클래스를 만들거나, 단순히 `JPAQueryFactory`를 주입받아 사용하는 방식을 더 많이 고려한다.

---

## 6. Querydsl 지원 클래스 직접 만들기

Spring Data가 제공하는 `QuerydslRepositorySupport`의 한계를 보완하기 위해 직접 지원 클래스를 만들 수 있다.

직접 만든 지원 클래스는 다음 기능을 제공할 수 있다.

- `JPAQueryFactory` 제공
- `EntityManager` 제공
- `select()` 지원
- `selectFrom()` 지원
- Spring Data Pageable 적용
- count 쿼리 분리
- Spring Data Sort 지원
- `PageableExecutionUtils` 활용

이런 지원 클래스를 만들면 여러 리포지토리에서 반복되는 페이징 처리와 Querydsl 설정 코드를 줄일 수 있다.

---

## 7. Spring Boot 3.x와 Querydsl 5.0

Spring Boot 3.x에서는 Querydsl 5.0 사용을 고려해야 한다.

중요한 확인 사항은 다음과 같다.

1. build.gradle 설정 변경
2. `PageableExecutionUtils` 패키지 변경
3. `fetchResults()`, `fetchCount()` 사용 지양

Spring Boot 3.x는 jakarta 기반이다.

따라서 Querydsl 의존성도 jakarta classifier를 맞춰야 한다.

```groovy
implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jakarta"
annotationProcessor "jakarta.annotation:jakarta.annotation-api"
annotationProcessor "jakarta.persistence:jakarta.persistence-api"
```

---

## 8. PageableExecutionUtils 패키지 변경

`PageableExecutionUtils`는 패키지 위치가 변경되었다.

기존 위치는 다음과 같다.

```java
org.springframework.data.repository.support.PageableExecutionUtils
```

신규 위치는 다음과 같다.

```java
org.springframework.data.support.PageableExecutionUtils
```

기능이 사라진 것은 아니고 패키지 위치가 변경된 것이다.

따라서 최신 Spring Data 환경에서는 신규 패키지를 import해야 한다.

---

## 9. fetchResults와 fetchCount Deprecated

Querydsl의 `fetchResults()`와 `fetchCount()`는 사용을 피하는 것이 좋다.

이 메서드들은 select 쿼리를 기반으로 count 쿼리를 내부에서 생성한다.

단순한 쿼리에서는 동작할 수 있지만, 복잡한 쿼리에서는 정확한 count 쿼리를 만들기 어렵다.

예를 들어 다음 요소가 섞이면 자동 count 변환이 문제가 될 수 있다.

- group by
- having
- distinct
- 복잡한 join
- fetch join
- DB 특화 함수
- 복잡한 projection

따라서 count가 필요하면 별도 count 쿼리를 작성한다.

---

## 10. count 쿼리 직접 작성

count 쿼리는 다음처럼 작성한다.

```java
Long totalCount = queryFactory
        .select(member.count())
        .from(member)
        .fetchOne();
```

`count(*)`가 필요하면 `Wildcard.count`를 사용할 수 있다.

```java
Long totalCount = queryFactory
        .select(Wildcard.count)
        .from(member)
        .fetchOne();
```

일반적으로 엔티티 ID 기준 count면 `member.count()`를 사용할 수 있다.

응답 결과는 숫자 하나이므로 `fetchOne()`을 사용한다.

---

## 11. 최신 페이징 예제

최신 방식에서는 content 조회와 count 조회를 분리한다.

```java
public Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition,
                                             Pageable pageable) {
    List<MemberTeamDto> content = queryFactory
            .select(new QMemberTeamDto(
                    member.id.as("memberId"),
                    member.username,
                    member.age,
                    team.id.as("teamId"),
                    team.name.as("teamName")))
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
}
```

이 방식은 Querydsl 5.0과 Spring Data 최신 흐름에 더 적합하다.

---

## 12. 실무 선택 기준

Querydsl 관련 기능은 다음 기준으로 선택하면 좋다.

### 단순 조회

Spring Data JPA 메서드 쿼리나 `@Query`를 사용한다.

```java
List<Member> findByUsername(String username);
```

### 복잡한 동적 검색

사용자 정의 리포지토리와 Querydsl을 사용한다.

```java
Page<MemberTeamDto> search(MemberSearchCondition condition, Pageable pageable);
```

### QuerydslPredicateExecutor

학습용이나 단순 조건에는 사용할 수 있지만, 복잡한 실무 검색에는 권장하지 않는다.

### Querydsl Web

컨트롤러가 Querydsl에 의존할 수 있으므로 실무에서는 신중하게 사용한다.

### QuerydslRepositorySupport

기존 지원 클래스의 한계를 알고 사용해야 한다.

최신 실무 흐름에서는 `JPAQueryFactory` 기반 직접 구현 또는 별도 지원 클래스를 고려한다.

---

## 13. Querydsl도 만능은 아니다

Querydsl은 매우 강력하지만 모든 문제를 해결하는 것은 아니다.

다음 상황에서는 다른 기술을 고려할 수 있다.

- DB 특화 기능을 적극적으로 사용해야 하는 경우
- 매우 복잡한 통계 쿼리
- 윈도우 함수나 재귀 쿼리 등 JPA/JPQL로 표현하기 어려운 경우
- 대량 데이터 배치 처리
- 성능상 SQL을 직접 세밀하게 제어해야 하는 경우

이런 경우에는 Native SQL, JdbcTemplate, MyBatis 등을 함께 고려할 수 있다.

중요한 것은 Querydsl을 쓰느냐가 아니라, 요구사항에 맞는 데이터 접근 방식을 선택하는 것이다.

---

## 오늘 정리

Spring Data JPA가 제공하는 Querydsl 기능은 간단한 상황에서는 편리하지만, 복잡한 실무 환경에서는 한계가 있다.

특히 `QuerydslPredicateExecutor`와 Querydsl Web 지원은 서비스나 컨트롤러가 Querydsl에 의존할 수 있으므로 주의해야 한다.

`QuerydslRepositorySupport`도 최신 `JPAQueryFactory` 중심 코드와는 맞지 않는 부분이 있다.

중요한 정리는 다음과 같다.

- `QuerydslPredicateExecutor`는 간단하지만 left join과 복잡한 조회에 한계가 있다.
- Querydsl Web은 컨트롤러가 Querydsl에 의존할 수 있다.
- `QuerydslRepositorySupport`는 최신 Querydsl 사용 흐름과 맞지 않는 제약이 있다.
- 실무에서는 사용자 정의 리포지토리와 `JPAQueryFactory`를 직접 사용하는 방식이 더 명확하다.
- Spring Boot 3.x에서는 Querydsl 5.0과 jakarta 설정을 맞춰야 한다.
- `PageableExecutionUtils`는 신규 패키지를 사용해야 한다.
- `fetchResults()`, `fetchCount()`는 피하고 count 쿼리를 별도로 작성하는 것이 좋다.
- 복잡한 쿼리는 Querydsl, 더 복잡하거나 DB 특화 기능이 필요하면 Native SQL, JdbcTemplate, MyBatis도 고려한다.
