# JPA 객체지향 쿼리 언어 JPQL 정리

## 학습 날짜

2026-05-20

## 학습 주제

- 객체지향 쿼리 언어
- JPQL
- Criteria
- QueryDSL
- 네이티브 SQL
- 기본 문법
- 파라미터 바인딩
- 프로젝션
- 페이징
- 조인
- 서브 쿼리
- 조건식
- 페치 조인
- 경로 표현식
- 다형성 쿼리
- 엔티티 직접 사용
- Named Query
- 벌크 연산

---

## 1. JPQL이 필요한 이유

JPA에서 가장 단순한 조회 방법은 `EntityManager.find()`이다.

```java
Member member = em.find(Member.class, memberId);
```

또는 객체 그래프 탐색으로 연관된 객체를 조회할 수 있다.

```java
Team team = member.getTeam();
```

하지만 조건이 있는 목록 조회가 필요하면 단순 조회만으로는 부족하다.

예를 들어 다음 요구사항이 있을 수 있다.

- 나이가 18살 이상인 회원 조회
- 이름에 특정 단어가 포함된 회원 조회
- 팀 이름으로 회원 조회
- 주문 금액 합계 조회
- 페이징 처리
- 정렬 처리

이런 경우 데이터베이스에서 필요한 데이터만 조회하는 쿼리가 필요하다.

JPA는 엔티티 객체를 대상으로 쿼리할 수 있는 JPQL을 제공한다.

---

## 2. JPQL이란

JPQL은 Java Persistence Query Language의 약자이다.

JPQL은 테이블이 아니라 엔티티 객체를 대상으로 하는 객체지향 쿼리 언어이다.

SQL과 문법은 비슷하지만 기준이 다르다.

- JPQL: 엔티티 객체와 필드를 대상으로 쿼리
- SQL: 데이터베이스 테이블과 컬럼을 대상으로 쿼리

예시는 다음과 같다.

```java
String jpql = "select m from Member m where m.age > 18";
List<Member> result = em.createQuery(jpql, Member.class)
        .getResultList();
```

여기서 `Member`는 테이블 이름이 아니라 엔티티 이름이다.

`age`도 컬럼명이 아니라 엔티티의 필드명이다.

JPQL은 실행 시점에 SQL로 변환되어 데이터베이스에 전달된다.

---

## 3. JPA의 다양한 쿼리 방법

JPA는 여러 쿼리 방법을 지원한다.

- JPQL
- Criteria
- QueryDSL
- 네이티브 SQL
- JDBC API 직접 사용
- MyBatis, SpringJdbcTemplate 함께 사용

### JPQL

문자열로 객체지향 쿼리를 작성한다.

가장 기본이 되는 JPA 쿼리 언어이다.

### Criteria

자바 코드로 JPQL을 작성하는 방식이다.

컴파일 시점에 일부 오류를 잡을 수 있지만 코드가 복잡하고 실용성이 떨어질 수 있다.

### QueryDSL

자바 코드로 타입 안전한 쿼리를 작성할 수 있다.

동적 쿼리 작성이 편리하고 실무에서 많이 사용된다.

### 네이티브 SQL

특정 데이터베이스 기능이 필요할 때 SQL을 직접 작성한다.

예를 들어 특정 DB 전용 함수나 힌트가 필요한 경우 사용할 수 있다.

---

## 4. JPQL 기본 문법

JPQL 기본 문법은 SQL과 유사하다.

```text
select_문 :: =
    select_절
    from_절
    [where_절]
    [groupby_절]
    [having_절]
    [orderby_절]
```

예시는 다음과 같다.

```java
select m from Member m where m.age > 18
```

주의할 점은 다음과 같다.

- 엔티티와 속성은 대소문자를 구분한다.
- JPQL 키워드는 대소문자를 구분하지 않는다.
- 테이블 이름이 아니라 엔티티 이름을 사용한다.
- 컬럼 이름이 아니라 필드 이름을 사용한다.
- 별칭은 필수이다.

---

## 5. TypedQuery와 Query

반환 타입이 명확하면 `TypedQuery`를 사용한다.

```java
TypedQuery<Member> query = em.createQuery(
        "select m from Member m",
        Member.class
);
```

반환 타입이 명확하지 않으면 `Query`를 사용한다.

```java
Query query = em.createQuery(
        "select m.username, m.age from Member m"
);
```

가능하면 타입 안정성을 위해 `TypedQuery`를 사용하는 것이 좋다.

---

## 6. 결과 조회

결과가 여러 건이면 `getResultList()`를 사용한다.

```java
List<Member> members = query.getResultList();
```

결과가 정확히 하나이면 `getSingleResult()`를 사용할 수 있다.

```java
Member member = query.getSingleResult();
```

다만 `getSingleResult()`는 결과가 없거나 둘 이상이면 예외가 발생한다.

따라서 단건 조회에서는 예외 처리를 고려해야 한다.

---

## 7. 파라미터 바인딩

JPQL에서는 이름 기준 파라미터 바인딩을 사용하는 것이 좋다.

```java
String jpql = "select m from Member m where m.username = :username";

List<Member> result = em.createQuery(jpql, Member.class)
        .setParameter("username", "member1")
        .getResultList();
```

위치 기준 파라미터도 가능하지만, 순서 변경에 취약하므로 권장하지 않는다.

이름 기준 파라미터는 의미가 명확하고 유지보수하기 좋다.

---

## 8. 프로젝션

프로젝션은 SELECT 절에서 조회할 대상을 지정하는 것이다.

조회 대상은 다음처럼 나눌 수 있다.

- 엔티티 프로젝션
- 임베디드 타입 프로젝션
- 스칼라 타입 프로젝션

엔티티를 조회하면 영속성 컨텍스트에서 관리된다.

```java
select m from Member m
```

여러 값을 조회할 때는 DTO로 바로 조회하는 방식도 사용할 수 있다.

```java
select new package.MemberDto(m.username, m.age) from Member m
```

DTO 생성자에 맞춰 패키지명을 포함해서 작성해야 한다.

---

## 9. 페이징

JPA는 페이징을 추상화해서 제공한다.

```java
List<Member> result = em.createQuery("select m from Member m order by m.id", Member.class)
        .setFirstResult(0)
        .setMaxResults(10)
        .getResultList();
```

- `setFirstResult`: 조회 시작 위치
- `setMaxResults`: 조회할 최대 개수

JPA는 데이터베이스 방언에 맞게 적절한 페이징 SQL을 생성한다.

---

## 10. 조인

JPQL도 조인을 지원한다.

### 내부 조인

```java
select m from Member m join m.team t
```

### 외부 조인

```java
select m from Member m left join m.team t
```

JPQL 조인은 테이블이 아니라 엔티티의 연관 필드를 기준으로 작성한다.

---

## 11. 페치 조인

페치 조인은 JPQL에서 성능 최적화를 위해 매우 중요하다.

연관된 엔티티나 컬렉션을 SQL 한 번으로 함께 조회하는 기능이다.

```java
select m from Member m join fetch m.team
```

일반 조인은 연관 엔티티를 함께 로딩하는 것을 보장하지 않는다.

반면 페치 조인은 연관 엔티티까지 함께 조회해서 영속성 컨텍스트에 올린다.

이를 통해 지연 로딩으로 인한 N+1 문제를 줄일 수 있다.

---

## 12. 컬렉션 페치 조인 주의사항

일대다 컬렉션을 페치 조인하면 데이터가 뻥튀기될 수 있다.

예를 들어 팀 하나에 회원이 여러 명이면 SQL 결과 row는 회원 수만큼 늘어난다.

JPQL에서 `distinct`를 사용하면 중복 엔티티 제거에 도움이 된다.

```java
select distinct t from Team t join fetch t.members
```

하지만 컬렉션 페치 조인은 페이징과 함께 사용할 때 주의해야 한다.

DB row 기준으로 페이징하면 엔티티 기준 페이징과 맞지 않을 수 있다.

실무에서는 컬렉션 페치 조인과 페이징을 함께 사용하기보다, batch size 설정이나 별도 조회 전략을 고려한다.

---

## 13. 경로 표현식

경로 표현식은 점을 찍어 객체 그래프를 탐색하는 문법이다.

```java
m.team.name
```

경로 표현식은 크게 다음으로 나눌 수 있다.

- 상태 필드
- 단일 값 연관 필드
- 컬렉션 값 연관 필드

단일 값 연관 필드는 묵시적 내부 조인이 발생할 수 있으므로 주의해야 한다.

컬렉션 값 연관 필드는 바로 더 탐색하기 어렵고, 명시적 조인을 사용하는 것이 좋다.

---

## 14. 다형성 쿼리

JPA는 상속관계 매핑에서 다형성 쿼리를 지원한다.

부모 타입을 조회하면 자식 타입도 함께 조회될 수 있다.

특정 자식 타입만 조회하고 싶으면 `TYPE`을 사용할 수 있다.

```java
select i from Item i
where type(i) in (Book, Movie)
```

부모 타입을 특정 자식 타입처럼 다루고 싶으면 `TREAT`를 사용할 수 있다.

```java
select i from Item i
where treat(i as Book).author = 'kim'
```

상속 구조를 조회할 때 유용하지만, 실행 SQL과 성능을 확인해야 한다.

---

## 15. 엔티티 직접 사용

JPQL에서 엔티티를 직접 사용하면 SQL에서는 해당 엔티티의 기본 키 값을 사용한다.

```java
select m from Member m where m = :member
```

이 쿼리는 SQL에서 `m.id = ?`처럼 동작한다.

외래 키 관계에서도 엔티티를 직접 사용할 수 있다.

```java
select m from Member m where m.team = :team
```

SQL에서는 `team_id = ?`로 변환된다.

---

## 16. Named Query

Named Query는 미리 정의해 이름을 부여한 JPQL이다.

```java
@NamedQuery(
    name = "Member.findByUsername",
    query = "select m from Member m where m.username = :username"
)
```

사용할 때는 이름으로 호출한다.

```java
List<Member> result = em.createNamedQuery("Member.findByUsername", Member.class)
        .setParameter("username", "member1")
        .getResultList();
```

Named Query는 정적 쿼리이므로 애플리케이션 로딩 시점에 문법 오류를 확인할 수 있다는 장점이 있다.

---

## 17. 벌크 연산

벌크 연산은 여러 데이터를 한 번의 쿼리로 수정하거나 삭제하는 기능이다.

```java
String qlString = "update Product p " +
        "set p.price = p.price * 1.1 " +
        "where p.stockAmount < :stockAmount";

int resultCount = em.createQuery(qlString)
        .setParameter("stockAmount", 10)
        .executeUpdate();
```

벌크 연산은 `executeUpdate()`를 사용하고, 영향받은 엔티티 수를 반환한다.

주의할 점은 벌크 연산이 영속성 컨텍스트를 무시하고 데이터베이스에 직접 실행된다는 것이다.

따라서 벌크 연산 후에는 영속성 컨텍스트를 초기화하는 것이 안전하다.

```java
em.clear();
```

---

## 18. JDBC, MyBatis, SpringJdbcTemplate 함께 사용 시 주의

JPA를 사용하면서 JDBC, MyBatis, SpringJdbcTemplate을 함께 사용할 수 있다.

하지만 이들은 JPA 영속성 컨텍스트를 거치지 않고 DB에 직접 SQL을 실행한다.

따라서 JPA가 아직 DB에 반영하지 않은 변경 내용이 있다면 문제가 생길 수 있다.

이런 경우 직접 SQL을 실행하기 전에 영속성 컨텍스트를 플러시해야 한다.

```java
em.flush();
```

---

## 19. QueryDSL 보충

Criteria는 자바 코드로 JPQL을 작성할 수 있지만 코드가 복잡해 실무에서 사용하기 어렵다.

QueryDSL은 자바 코드로 타입 안전한 쿼리를 작성할 수 있게 도와준다.

장점은 다음과 같다.

- 컴파일 시점에 문법 오류를 찾을 수 있다.
- 동적 쿼리 작성이 편리하다.
- 문자열 기반 JPQL보다 리팩토링에 유리하다.
- 실무에서 복잡한 검색 조건 처리에 많이 사용된다.

JPQL 기본기를 먼저 이해한 뒤, 실무 동적 쿼리는 QueryDSL로 확장하는 흐름이 좋다.

---

## 20. 오늘 정리

JPQL은 JPA에서 복잡한 조회를 처리하기 위한 핵심 쿼리 언어이다.

`em.find()`는 기본 키 단건 조회에는 편리하지만, 조건 검색이나 목록 조회에는 JPQL이 필요하다.

중요한 정리는 다음과 같다.

- JPQL은 테이블이 아니라 엔티티 객체를 대상으로 쿼리한다.
- JPQL은 SQL로 변환되어 실행된다.
- 엔티티명과 필드명을 기준으로 작성한다.
- 파라미터 바인딩은 이름 기준을 사용하는 것이 좋다.
- 페치 조인은 연관 엔티티를 함께 조회해 N+1 문제를 줄일 수 있다.
- 컬렉션 페치 조인과 페이징은 함께 사용할 때 주의해야 한다.
- 경로 표현식의 묵시적 조인을 주의해야 한다.
- Named Query는 로딩 시점에 문법 검증이 가능하다.
- 벌크 연산은 영속성 컨텍스트를 무시하므로 실행 후 초기화가 필요하다.
- 복잡한 동적 쿼리는 QueryDSL을 고려할 수 있다.
