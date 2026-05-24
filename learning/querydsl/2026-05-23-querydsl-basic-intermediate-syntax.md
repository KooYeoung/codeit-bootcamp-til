# Querydsl 기본 문법과 중급 문법 정리

## 학습 날짜

2026-05-23

## 학습 주제

- JPQL vs Querydsl
- Q-Type 활용
- 검색 조건
- 결과 조회
- 정렬
- 페이징
- 집합
- 조인과 on절
- 페치 조인
- 서브쿼리
- Case 문
- 프로젝션
- DTO 조회
- `@QueryProjection`
- 동적 쿼리
- 벌크 연산
- SQL function

---

## 1. JPQL과 Querydsl 비교

JPQL은 문자열로 쿼리를 작성한다.

```java
String qlString =
        "select m from Member m " +
        "where m.username = :username";

Member findMember = em.createQuery(qlString, Member.class)
        .setParameter("username", "member1")
        .getSingleResult();
```

Querydsl은 자바 코드로 쿼리를 작성한다.

```java
Member findMember = queryFactory
        .select(member)
        .from(member)
        .where(member.username.eq("member1"))
        .fetchOne();
```

둘의 차이는 다음과 같다.

- JPQL은 문자열 기반이다.
- Querydsl은 코드 기반이다.
- JPQL의 오타는 실행 시점에 발견될 수 있다.
- Querydsl은 필드명 오류를 컴파일 시점에 발견할 수 있다.
- JPQL은 파라미터 바인딩을 직접 작성한다.
- Querydsl은 파라미터 바인딩을 자동으로 처리한다.

Querydsl은 JPQL을 대체한다기보다, JPQL을 자바 코드로 안전하게 작성하게 도와주는 빌더라고 이해할 수 있다.

---

## 2. JPAQueryFactory와 Q-Type

Querydsl 쿼리는 보통 `JPAQueryFactory`로 시작한다.

```java
JPAQueryFactory queryFactory = new JPAQueryFactory(em);
```

Q-Type은 Querydsl이 생성하는 쿼리 전용 클래스이다.

사용 방법은 크게 두 가지이다.

```java
QMember qMember = new QMember("m");
QMember qMember = QMember.member;
```

일반적으로 같은 테이블을 여러 번 조인해야 하는 상황이 아니라면 기본 인스턴스를 static import해서 사용하는 방식이 간단하다.

```java
import static study.querydsl.entity.QMember.member;

Member findMember = queryFactory
        .selectFrom(member)
        .where(member.username.eq("member1"))
        .fetchOne();
```

---

## 3. 검색 조건

Querydsl은 JPQL에서 제공하는 다양한 검색 조건을 메서드로 제공한다.

```java
member.username.eq("member1")
member.username.ne("member1")
member.username.isNotNull()
member.age.in(10, 20)
member.age.between(10, 30)
member.age.goe(30)
member.age.gt(30)
member.age.loe(30)
member.age.lt(30)
member.username.like("member%")
member.username.contains("member")
member.username.startsWith("member")
```

조건은 `and()`, `or()`로 연결할 수 있다.

또는 `where()`에 여러 조건을 파라미터로 넘길 수 있다.

```java
queryFactory
        .selectFrom(member)
        .where(
                member.username.eq("member1"),
                member.age.eq(10)
        )
        .fetch();
```

`where()`에 여러 조건을 넘기면 AND 조건으로 연결된다.

---

## 4. 결과 조회

Querydsl의 대표 결과 조회 메서드는 다음과 같다.

### fetch

리스트를 조회한다.

조회 결과가 없으면 빈 리스트를 반환한다.

```java
List<Member> result = queryFactory
        .selectFrom(member)
        .fetch();
```

### fetchOne

단건을 조회한다.

결과가 없으면 `null`을 반환하고, 결과가 둘 이상이면 예외가 발생한다.

```java
Member result = queryFactory
        .selectFrom(member)
        .fetchOne();
```

### fetchFirst

첫 번째 한 건을 조회한다.

내부적으로 `limit(1).fetchOne()`과 비슷하게 이해할 수 있다.

```java
Member result = queryFactory
        .selectFrom(member)
        .fetchFirst();
```

### Querydsl 5.0 보충

기존에 사용하던 `fetchResults()`, `fetchCount()`는 Querydsl 5.0 흐름에서는 사용을 피하는 것이 좋다.

복잡한 쿼리에서 count 쿼리를 자동으로 생성하는 방식이 정확하지 않을 수 있기 때문이다.

count가 필요하면 별도 count 쿼리를 명시적으로 작성한다.

---

## 5. 정렬과 페이징

정렬은 `orderBy()`를 사용한다.

```java
List<Member> result = queryFactory
        .selectFrom(member)
        .orderBy(member.age.desc(), member.username.asc())
        .fetch();
```

페이징은 `offset`, `limit`를 사용한다.

```java
List<Member> result = queryFactory
        .selectFrom(member)
        .orderBy(member.username.desc())
        .offset(1)
        .limit(2)
        .fetch();
```

`offset`은 0부터 시작한다.

Spring Data JPA의 `Pageable`과 연동할 때는 `pageable.getOffset()`, `pageable.getPageSize()`를 사용할 수 있다.

---

## 6. 조인과 페치 조인

기본 조인은 다음처럼 작성한다.

```java
List<Member> result = queryFactory
        .selectFrom(member)
        .join(member.team, team)
        .where(team.name.eq("teamA"))
        .fetch();
```

외부 조인은 `leftJoin()`을 사용한다.

```java
queryFactory
        .selectFrom(member)
        .leftJoin(member.team, team)
        .fetch();
```

`on`절을 사용하면 조인 대상을 필터링할 수 있다.

```java
queryFactory
        .select(member, team)
        .from(member)
        .leftJoin(member.team, team).on(team.name.eq("teamA"))
        .fetch();
```

페치 조인은 연관 엔티티를 함께 조회할 때 사용한다.

```java
Member result = queryFactory
        .selectFrom(member)
        .join(member.team, team).fetchJoin()
        .where(member.username.eq("member1"))
        .fetchOne();
```

일반 조인은 SQL 조인만 수행하고 연관 엔티티를 영속성 컨텍스트에 함께 로딩하는 것을 보장하지 않는다.

페치 조인은 연관 엔티티까지 함께 조회해 영속 상태로 만든다.

---

## 7. 서브쿼리와 Case 문

Querydsl에서 서브쿼리는 `JPAExpressions`를 사용한다.

```java
List<Member> result = queryFactory
        .selectFrom(member)
        .where(member.age.eq(
                JPAExpressions
                        .select(memberSub.age.max())
                        .from(memberSub)
        ))
        .fetch();
```

JPA JPQL의 한계상 from 절 서브쿼리는 지원하지 않는다.

from 절 서브쿼리가 필요하면 쿼리를 나누거나, 조인으로 해결하거나, native SQL을 고려할 수 있다.

Case 문은 조건에 따라 다른 값을 반환할 때 사용한다.

```java
List<String> result = queryFactory
        .select(member.age
                .when(10).then("열살")
                .when(20).then("스무살")
                .otherwise("기타"))
        .from(member)
        .fetch();
```

복잡한 조건은 `CaseBuilder`를 사용할 수 있다.

---

## 8. 프로젝션

프로젝션은 select 대상을 지정하는 것이다.

대상이 하나면 해당 타입으로 바로 받을 수 있다.

```java
List<String> result = queryFactory
        .select(member.username)
        .from(member)
        .fetch();
```

대상이 둘 이상이면 `Tuple`이나 DTO로 조회한다.

```java
List<Tuple> result = queryFactory
        .select(member.username, member.age)
        .from(member)
        .fetch();
```

`Tuple`은 Querydsl 전용 타입이므로 리포지토리 내부에서만 사용하고, 서비스나 컨트롤러 밖으로 노출하지 않는 것이 좋다.

---

## 9. DTO 조회 방식

Querydsl은 DTO 조회를 여러 방식으로 지원한다.

### Projections.bean

setter를 사용해 값을 주입한다.

```java
List<MemberDto> result = queryFactory
        .select(Projections.bean(MemberDto.class,
                member.username,
                member.age))
        .from(member)
        .fetch();
```

### Projections.fields

필드에 직접 값을 주입한다.

```java
List<MemberDto> result = queryFactory
        .select(Projections.fields(MemberDto.class,
                member.username,
                member.age))
        .from(member)
        .fetch();
```

### Projections.constructor

생성자를 사용한다.

```java
List<MemberDto> result = queryFactory
        .select(Projections.constructor(MemberDto.class,
                member.username,
                member.age))
        .from(member)
        .fetch();
```

### @QueryProjection

DTO 생성자에 `@QueryProjection`을 붙이고 Q-Type을 생성해서 사용한다.

```java
List<MemberDto> result = queryFactory
        .select(new QMemberDto(member.username, member.age))
        .from(member)
        .fetch();
```

`@QueryProjection`은 컴파일 시점 타입 체크가 가능해 안전하다.

하지만 DTO가 Querydsl에 의존하게 되고 DTO Q-Type이 생성된다는 단점이 있다.

---

## 10. 동적 쿼리

`BooleanBuilder`는 조건을 하나씩 추가하면서 동적 쿼리를 만들 수 있다.

```java
BooleanBuilder builder = new BooleanBuilder();

if (usernameCond != null) {
    builder.and(member.username.eq(usernameCond));
}

if (ageCond != null) {
    builder.and(member.age.eq(ageCond));
}

return queryFactory
        .selectFrom(member)
        .where(builder)
        .fetch();
```

더 권장되는 방식은 조건 메서드를 분리하고 `where()`에 여러 파라미터로 넘기는 것이다.

```java
private BooleanExpression usernameEq(String usernameCond) {
    return hasText(usernameCond) ? member.username.eq(usernameCond) : null;
}

private BooleanExpression ageEq(Integer ageCond) {
    return ageCond != null ? member.age.eq(ageCond) : null;
}
```

사용할 때는 다음처럼 작성한다.

```java
return queryFactory
        .selectFrom(member)
        .where(
                usernameEq(usernameCond),
                ageEq(ageCond)
        )
        .fetch();
```

이 방식은 조건 메서드를 재사용할 수 있고 쿼리 본문이 읽기 쉬워진다.

---

## 11. 벌크 연산과 SQL function

벌크 수정은 여러 데이터를 한 번에 수정한다.

```java
long count = queryFactory
        .update(member)
        .set(member.username, "비회원")
        .where(member.age.lt(28))
        .execute();
```

벌크 삭제도 가능하다.

```java
long count = queryFactory
        .delete(member)
        .where(member.age.gt(18))
        .execute();
```

벌크 연산은 영속성 컨텍스트를 거치지 않는다.

이미 영속성 컨텍스트에 있는 엔티티와 DB 값이 달라질 수 있으므로 필요한 경우 `em.flush()`와 `em.clear()`를 고려한다.

SQL function은 `Expressions.stringTemplate()` 등으로 호출할 수 있다.

특정 DB 함수는 데이터베이스 의존성이 생길 수 있으므로 주의해야 한다.

---

## 오늘 정리

Querydsl 기본 문법은 JPQL을 자바 코드로 안전하게 작성하는 데 초점이 있다.

중급 문법에서는 DTO 조회, 동적 쿼리, 벌크 연산처럼 실무에서 자주 필요한 기능을 다룬다.

중요한 정리는 다음과 같다.

- Querydsl은 JPQL 빌더이다.
- Q-Type을 사용해 타입 안전하게 필드에 접근한다.
- `where()`의 null 무시 특성은 동적 쿼리에 매우 유용하다.
- `fetch()`는 리스트, `fetchOne()`은 단건, `fetchFirst()`는 첫 번째 한 건 조회에 사용한다.
- Querydsl 5.0에서는 count 쿼리를 별도로 작성하는 것이 좋다.
- DTO 조회는 여러 방식이 있고, 각 방식의 의존성과 타입 안정성을 비교해야 한다.
- `@QueryProjection`은 안전하지만 DTO가 Querydsl에 의존한다.
- 벌크 연산 후에는 영속성 컨텍스트와 DB 상태 불일치에 주의해야 한다.
- DB 함수 사용은 특정 DB 의존성을 고려해야 한다.
