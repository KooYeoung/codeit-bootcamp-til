# JPA, Spring Data JPA, Querydsl 정리

## 학습 날짜

2026-05-18

## 학습 주제

- SQL 중심 개발의 문제점
- ORM과 JPA
- Hibernate
- 영속성 컨텍스트 관점
- 스프링 데이터 JPA
- JpaRepository
- 쿼리 메서드
- Querydsl
- 타입 안전한 쿼리
- 동적 쿼리
- 실용적인 JPA 조합

---

## 1. SQL 중심 개발의 문제점

관계형 데이터베이스를 사용하면 데이터를 저장하고 조회하기 위해 SQL이 필요하다.

문제는 애플리케이션이 객체지향 언어로 작성되어도, 데이터를 저장하려면 결국 SQL 중심으로 개발하게 되는 경우가 많다는 점이다.

예를 들어 회원 객체가 있다고 하자.

```java
public class Member {
    private String memberId;
    private String name;
}
```

이 객체를 저장하려면 SQL을 작성해야 한다.

```sql
insert into member(member_id, name) values (?, ?)
```

조회할 때도 SQL을 작성하고, SQL 결과를 다시 객체에 담아야 한다.

```sql
select member_id, name from member where member_id = ?
```

필드가 추가되면 문제가 더 커진다.

```java
public class Member {
    private String memberId;
    private String name;
    private String tel;
}
```

이 경우 insert, select, update SQL과 결과 매핑 코드를 모두 수정해야 한다.

즉, 객체를 수정했는데 SQL도 함께 계속 수정해야 하는 반복이 생긴다.

---

## 2. 객체와 관계형 DB의 차이

객체와 관계형 데이터베이스는 바라보는 방식이 다르다.

객체는 다음과 같은 특징을 가진다.

- 상속
- 연관관계
- 객체 그래프 탐색
- 참조 비교

반면 관계형 데이터베이스는 테이블 중심으로 데이터를 관리한다.

객체의 상속 구조를 테이블에 저장하려면 슈퍼타입, 서브타입 테이블 구조를 고민해야 한다.

객체의 연관관계는 참조로 표현하지만, 테이블은 외래 키로 표현한다.

이 차이를 개발자가 직접 SQL로 해결하려면 코드가 복잡해진다.

JPA는 이런 객체와 관계형 DB의 차이를 줄여주는 ORM 기술이다.

---

## 3. ORM과 JPA

ORM은 Object Relational Mapping의 약자이다.

객체와 관계형 데이터베이스를 매핑하는 기술이다.

JPA는 Java Persistence API의 약자이며, 자바 ORM 표준이다.

중요한 점은 JPA는 표준 인터페이스이고, Hibernate는 대표적인 구현체라는 점이다.

즉, 개발자는 JPA라는 표준을 사용하고, 실제 동작은 Hibernate가 처리하는 구조로 이해할 수 있다.

---

## 4. JPA를 사용하는 이유

JPA를 사용하는 이유는 다음과 같다.

- SQL 중심 개발에서 객체 중심 개발로 이동할 수 있다.
- 반복적인 CRUD SQL 작성이 줄어든다.
- 필드 변경 시 유지보수 부담이 줄어든다.
- 객체와 관계형 DB의 패러다임 차이를 해결해준다.
- 1차 캐시, 쓰기 지연, 지연 로딩 같은 성능 최적화 기능을 제공한다.
- 데이터 접근 기술에 대한 추상화와 벤더 독립성을 얻을 수 있다.

JPA의 기본 CRUD는 다음처럼 사용할 수 있다.

```java
entityManager.persist(member);
Member member = entityManager.find(Member.class, memberId);
member.setName("변경 이름");
entityManager.remove(member);
```

수정의 경우 별도 update SQL을 직접 호출하지 않아도, 트랜잭션 안에서 엔티티 변경을 감지해서 반영할 수 있다.

---

## 5. 엔티티

JPA에서 데이터베이스 테이블과 매핑되는 객체를 엔티티라고 한다.

```java
@Entity
public class Item {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String itemName;
    private Integer price;
    private Integer quantity;
}
```

`@Entity`는 JPA가 관리하는 객체라는 뜻이다.

`@Id`는 기본 키를 의미한다.

`@GeneratedValue`는 기본 키 값을 자동 생성하는 전략을 지정할 때 사용한다.

---

## 6. EntityManager

`EntityManager`는 JPA에서 엔티티를 저장, 조회, 수정, 삭제하는 핵심 객체이다.

```java
em.persist(item);
Item findItem = em.find(Item.class, id);
em.remove(item);
```

`EntityManager`를 통해 엔티티를 영속성 컨텍스트에 저장하고 관리한다.

영속성 컨텍스트는 JPA가 엔티티를 관리하는 공간으로 이해할 수 있다.

---

## 7. 1차 캐시와 동일성 보장

JPA는 같은 트랜잭션 안에서 조회한 같은 엔티티의 동일성을 보장한다.

```java
Member m1 = em.find(Member.class, memberId);
Member m2 = em.find(Member.class, memberId);

System.out.println(m1 == m2); // true
```

처음 조회할 때 DB에서 가져온 엔티티를 영속성 컨텍스트의 1차 캐시에 보관한다.

같은 엔티티를 다시 조회하면 DB를 다시 조회하지 않고 1차 캐시에서 반환할 수 있다.

---

## 8. 쓰기 지연

JPA는 트랜잭션을 커밋할 때까지 insert SQL을 모아둘 수 있다.

```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
```

이 시점에 바로 DB에 insert SQL을 보내지 않고, 쓰기 지연 저장소에 모아둘 수 있다.

트랜잭션 커밋 시점에 SQL을 데이터베이스에 전달한다.

이 기능은 JDBC batch와 함께 성능 최적화에 도움이 될 수 있다.

---

## 9. 변경 감지

JPA에서는 영속 상태의 엔티티 값을 변경하면 트랜잭션 커밋 시점에 변경을 감지해서 update SQL을 실행한다.

```java
Item item = em.find(Item.class, id);
item.setItemName("newName");
```

개발자가 직접 update 메서드를 호출하지 않아도 된다.

이것을 변경 감지라고 한다.

단, 엔티티가 영속 상태이고 트랜잭션 안에서 변경되어야 한다.

---

## 10. 지연 로딩

JPA는 연관된 객체를 실제로 사용할 때 조회하는 지연 로딩을 지원한다.

예를 들어 회원을 조회했을 때 팀 정보가 항상 필요한 것은 아니다.

지연 로딩을 사용하면 팀을 실제로 접근할 때 SQL을 실행할 수 있다.

```java
Member member = em.find(Member.class, memberId);
Team team = member.getTeam();
```

지연 로딩은 성능 최적화에 도움이 되지만, 언제 SQL이 실행되는지 이해하지 못하면 N+1 문제 같은 성능 문제가 생길 수 있다.

---

## 11. 스프링 데이터 JPA

스프링 데이터 JPA는 JPA를 더 편리하게 사용할 수 있게 도와주는 라이브러리이다.

가장 대표적인 기능은 다음과 같다.

- 공통 인터페이스 기능
- 쿼리 메서드 기능

공통 인터페이스 기능은 `JpaRepository`를 상속하면 기본 CRUD 기능을 제공하는 것이다.

```java
public interface ItemRepository extends JpaRepository<Item, Long> {
}
```

이렇게 인터페이스만 작성하면 스프링 데이터 JPA가 구현체를 프록시로 만들어 스프링 빈으로 등록한다.

개발자는 반복적인 CRUD 구현 클래스를 만들 필요가 없다.

---

## 12. JpaRepository

`JpaRepository`는 다양한 기본 기능을 제공한다.

대표적으로 다음 기능을 사용할 수 있다.

- 저장
- 단건 조회
- 전체 조회
- 삭제
- 페이징
- 정렬

예를 들어 다음처럼 사용할 수 있다.

```java
itemRepository.save(item);
Optional<Item> item = itemRepository.findById(id);
List<Item> items = itemRepository.findAll();
itemRepository.delete(item);
```

단순 CRUD는 대부분 `JpaRepository`만으로 처리할 수 있다.

---

## 13. 쿼리 메서드

스프링 데이터 JPA는 메서드 이름을 분석해서 쿼리를 생성한다.

```java
List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
```

이 메서드는 username이 일치하고 age가 특정 값보다 큰 회원을 조회하는 쿼리로 해석된다.

대표적인 규칙은 다음과 같다.

- `find...By`
- `read...By`
- `query...By`
- `count...By`
- `exists...By`

단순 조건 조회에서는 매우 편리하다.

하지만 조건이 많아지고 동적 쿼리가 필요해지면 메서드 이름이 길어지고 관리가 어려워진다.

이때 Querydsl을 함께 사용하면 좋다.

---

## 14. Querydsl이 필요한 이유

JPA에서 JPQL을 문자열로 작성하면 오타를 컴파일 시점에 잡기 어렵다.

```java
String jpql = "select m from Member m where m.username = :username";
```

문자열 안의 필드명이나 엔티티명이 틀려도 컴파일은 성공할 수 있다.

실행 시점에야 오류가 발견된다.

Querydsl은 자바 코드로 쿼리를 작성하게 해서 이런 문제를 줄여준다.

Q 타입을 사용하므로 필드명 오타를 컴파일 시점에 잡을 수 있다.

---

## 15. Querydsl 기본 구조

Querydsl에서는 `JPAQueryFactory`를 사용한다.

```java
List<Item> result = queryFactory
        .select(item)
        .from(item)
        .where(item.itemName.like("%" + itemName + "%"))
        .fetch();
```

이 코드는 JPQL을 자바 코드로 작성하는 방식이라고 이해할 수 있다.

장점은 다음과 같다.

- 타입 안전하다.
- IDE 자동완성을 활용할 수 있다.
- 컴파일 시점에 오류를 잡을 수 있다.
- 동적 쿼리를 깔끔하게 작성할 수 있다.

---

## 16. Querydsl 동적 쿼리

검색 조건은 있을 수도 있고 없을 수도 있다.

예를 들어 상품명 조건과 최대 가격 조건이 선택적으로 들어올 수 있다.

Querydsl에서는 조건 메서드를 분리해서 동적 쿼리를 깔끔하게 작성할 수 있다.

```java
List<Item> result = query
        .select(item)
        .from(item)
        .where(likeItemName(itemName), maxPrice(maxPrice))
        .fetch();
```

조건 메서드는 값이 없으면 `null`을 반환한다.

```java
private BooleanExpression likeItemName(String itemName) {
    if (StringUtils.hasText(itemName)) {
        return item.itemName.like("%" + itemName + "%");
    }
    return null;
}

private BooleanExpression maxPrice(Integer maxPrice) {
    if (maxPrice != null) {
        return item.price.loe(maxPrice);
    }
    return null;
}
```

Querydsl의 `where()`는 `null` 조건을 무시한다.

따라서 조건이 있을 때만 자동으로 적용된다.

---

## 17. BooleanBuilder

동적 쿼리는 `BooleanBuilder`로도 작성할 수 있다.

```java
BooleanBuilder builder = new BooleanBuilder();

if (StringUtils.hasText(itemName)) {
    builder.and(item.itemName.like("%" + itemName + "%"));
}

if (maxPrice != null) {
    builder.and(item.price.loe(maxPrice));
}
```

`BooleanBuilder`는 조건을 하나씩 조립할 때 유용하다.

다만 조건 메서드 분리 방식이 더 읽기 좋고 재사용하기 쉬운 경우가 많다.

---

## 18. 스프링 데이터 JPA와 Querydsl 조합

실용적인 구조는 다음과 같다.

- 기본 CRUD와 단순 조회: 스프링 데이터 JPA
- 복잡한 동적 조회: Querydsl

예를 들어 다음처럼 리포지토리를 분리할 수 있다.

- `ItemRepositoryV2`: `JpaRepository`를 상속해서 기본 CRUD 담당
- `ItemQueryRepositoryV2`: Querydsl로 복잡한 조회 담당

이렇게 하면 스프링 데이터 JPA의 편리함과 Querydsl의 동적 쿼리 장점을 함께 사용할 수 있다.

---

## 19. 복잡한 SQL은 어떻게 할까

JPA와 Querydsl이 대부분의 조회를 해결할 수 있지만, 모든 쿼리에 적합한 것은 아니다.

아주 복잡한 통계 쿼리나 DB 벤더 특화 기능을 많이 사용하는 경우에는 직접 SQL이 더 적합할 수 있다.

이런 경우 다음 기술을 함께 사용할 수 있다.

- JdbcTemplate
- MyBatis
- 네이티브 SQL

중요한 것은 모든 상황을 한 기술로 해결하려고 하지 않는 것이다.

---

## 20. 오늘 정리

JPA는 SQL 중심 개발을 객체 중심 개발로 바꾸기 위한 핵심 기술이다.

스프링 데이터 JPA는 JPA의 반복적인 리포지토리 코드를 줄여주고, Querydsl은 복잡한 동적 쿼리를 타입 안전하게 작성할 수 있게 도와준다.

정리하면 다음과 같다.

- JPA는 자바 ORM 표준이다.
- Hibernate는 대표적인 JPA 구현체이다.
- JPA는 객체와 관계형 DB의 차이를 줄여준다.
- JPA는 1차 캐시, 동일성 보장, 쓰기 지연, 변경 감지, 지연 로딩을 제공한다.
- 스프링 데이터 JPA는 `JpaRepository`로 기본 CRUD를 제공한다.
- 쿼리 메서드는 단순 조회에 편리하지만 복잡한 동적 조건에는 한계가 있다.
- Querydsl은 타입 안전하고 동적 쿼리에 강하다.
- 실무에서는 JPA, 스프링 데이터 JPA, Querydsl을 기본으로 사용하고, 매우 복잡한 SQL은 JdbcTemplate이나 MyBatis로 보완할 수 있다.

앞으로 JPA를 사용할 때는 단순히 SQL이 줄어드는 것만 보지 말고, 영속성 컨텍스트와 트랜잭션 안에서 엔티티가 어떻게 관리되는지 함께 이해해야겠다.
