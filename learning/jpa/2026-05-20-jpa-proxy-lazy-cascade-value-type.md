# JPA 프록시, 지연 로딩, 영속성 전이, 값 타입 정리

## 학습 날짜

2026-05-20

## 학습 주제

- 프록시
- `em.find()`와 `em.getReference()`
- 지연 로딩과 즉시 로딩
- 영속성 전이
- 고아 객체
- 값 타입
- 임베디드 타입
- 값 타입과 불변 객체
- 값 타입 비교
- 값 타입 컬렉션

---

## 1. 프록시가 필요한 이유

엔티티를 조회할 때 연관된 엔티티를 항상 함께 조회해야 하는 것은 아니다.

예를 들어 회원과 팀이 있다고 하자.

회원 이름만 출력하는 기능에서는 팀 정보가 필요하지 않다.

```java
Member member = em.find(Member.class, memberId);
System.out.println(member.getUsername());
```

하지만 회원 이름과 팀 이름을 함께 출력하는 기능에서는 팀 정보가 필요하다.

```java
Member member = em.find(Member.class, memberId);
Team team = member.getTeam();
System.out.println(team.getName());
```

항상 팀을 함께 조회하면 필요 없는 경우에도 SQL이 더 실행될 수 있다.

JPA는 이런 문제를 해결하기 위해 프록시와 지연 로딩을 제공한다.

---

## 2. em.find()와 em.getReference()

`em.find()`는 데이터베이스를 조회해서 실제 엔티티 객체를 반환한다.

```java
Member member = em.find(Member.class, memberId);
```

반면 `em.getReference()`는 데이터베이스 조회를 미루고 가짜 엔티티 객체인 프록시를 반환한다.

```java
Member member = em.getReference(Member.class, memberId);
```

프록시는 실제 객체처럼 보이지만, 내부에는 실제 엔티티를 나중에 조회할 수 있는 정보가 들어있다.

프록시의 실제 데이터가 필요한 순간 DB를 조회해서 초기화한다.

---

## 3. 프록시 특징

프록시는 실제 클래스를 상속받아 만들어진다.

따라서 겉모양은 실제 엔티티와 비슷하다.

주요 특징은 다음과 같다.

- 프록시 객체는 처음 사용할 때 한 번만 초기화된다.
- 프록시가 초기화된다고 프록시 객체가 실제 엔티티 객체로 바뀌는 것은 아니다.
- 초기화 이후 프록시를 통해 실제 엔티티에 접근할 수 있다.
- 프록시는 원본 엔티티를 상속받으므로 타입 비교 시 `==`보다 `instanceof`를 사용하는 것이 안전하다.
- 영속성 컨텍스트에 이미 실제 엔티티가 있으면 `getReference()`를 호출해도 실제 엔티티가 반환될 수 있다.
- 준영속 상태에서 프록시를 초기화하려고 하면 `LazyInitializationException`이 발생할 수 있다.

---

## 4. 프록시 초기화

프록시는 실제 데이터가 필요한 시점에 초기화된다.

```java
Member member = em.getReference(Member.class, memberId);
member.getUsername(); // 이 시점에 초기화 가능
```

초기화 흐름은 다음과 같다.

1. 프록시 객체에 메서드를 호출한다.
2. 프록시가 영속성 컨텍스트에 초기화를 요청한다.
3. 영속성 컨텍스트가 DB를 조회한다.
4. 실제 엔티티가 생성된다.
5. 프록시가 실제 엔티티에 메서드 호출을 위임한다.

중요한 점은 프록시 초기화에는 영속성 컨텍스트가 필요하다는 것이다.

따라서 트랜잭션이 끝나거나 엔티티 매니저가 닫힌 뒤 프록시를 초기화하면 문제가 생길 수 있다.

---

## 5. 지연 로딩

지연 로딩은 연관된 엔티티를 실제 사용할 때 조회하는 방식이다.

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "TEAM_ID")
private Team team;
```

회원을 조회할 때 팀은 실제 엔티티가 아니라 프록시로 들어온다.

```java
Member member = em.find(Member.class, memberId);
Team team = member.getTeam(); // 프록시
team.getName();               // 이때 팀 조회
```

지연 로딩을 사용하면 불필요한 연관 엔티티 조회를 줄일 수 있다.

실무에서는 기본적으로 지연 로딩을 사용하는 것이 좋다.

---

## 6. 즉시 로딩

즉시 로딩은 엔티티를 조회할 때 연관된 엔티티도 함께 조회하는 방식이다.

```java
@ManyToOne(fetch = FetchType.EAGER)
@JoinColumn(name = "TEAM_ID")
private Team team;
```

즉시 로딩은 편해 보이지만 실무에서는 주의해야 한다.

이유는 다음과 같다.

- 예상하지 못한 SQL이 실행될 수 있다.
- 여러 연관관계가 즉시 로딩이면 조회 범위가 커질 수 있다.
- JPQL에서 N+1 문제가 발생할 수 있다.
- 성능 튜닝이 어려워진다.

따라서 실무에서는 모든 연관관계를 LAZY로 설정하고, 필요한 조회에서 페치 조인을 사용하는 것이 좋다.

---

## 7. N+1 문제와 페치 조인 보충

즉시 로딩 또는 지연 로딩을 잘못 사용하면 N+1 문제가 발생할 수 있다.

예를 들어 회원 목록을 조회한 뒤 각 회원의 팀을 접근하면 다음 문제가 생길 수 있다.

1. 회원 목록 조회 SQL 1번
2. 각 회원의 팀 조회 SQL N번

이를 N+1 문제라고 한다.

해결 방법 중 하나는 페치 조인이다.

```java
select m from Member m join fetch m.team
```

페치 조인을 사용하면 회원과 팀을 한 번의 쿼리로 함께 조회할 수 있다.

하지만 컬렉션 페치 조인과 페이징을 함께 사용할 때는 주의해야 한다.

---

## 8. 영속성 전이 CASCADE

영속성 전이는 특정 엔티티를 영속 상태로 만들 때, 연관된 엔티티도 함께 영속 상태로 만드는 기능이다.

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
private List<Child> children = new ArrayList<>();
```

부모를 저장할 때 자식도 함께 저장할 수 있다.

```java
Parent parent = new Parent();
Child child1 = new Child();
Child child2 = new Child();

parent.addChild(child1);
parent.addChild(child2);

em.persist(parent);
```

cascade를 사용하면 연관된 엔티티의 생명주기를 함께 관리할 수 있다.

하지만 모든 연관관계에 무분별하게 사용하면 위험하다.

---

## 9. CASCADE 사용 기준

CASCADE는 부모와 자식의 생명주기가 거의 같을 때 사용한다.

예를 들어 게시글과 첨부파일처럼 부모가 사라지면 자식도 함께 사라지는 관계라면 고려할 수 있다.

하지만 다음 관계에는 신중해야 한다.

- 여러 부모가 같은 자식을 공유하는 경우
- 자식 엔티티가 독립적으로 관리되는 경우
- 다른 aggregate에서 참조하는 경우

예를 들어 회원과 팀 관계에서 팀은 여러 회원이 공유할 수 있으므로 회원 저장 시 팀까지 cascade 하는 것은 일반적으로 적절하지 않다.

---

## 10. 고아 객체

고아 객체 제거는 부모 엔티티와의 관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능이다.

```java
@OneToMany(mappedBy = "parent", orphanRemoval = true)
private List<Child> children = new ArrayList<>();
```

컬렉션에서 자식을 제거하면 DB에서도 삭제된다.

```java
parent.getChildren().remove(0);
```

고아 객체 제거는 참조가 제거된 엔티티를 삭제하는 기능이므로, 특정 부모가 자식을 완전히 소유하는 관계에서 사용해야 한다.

---

## 11. 값 타입

JPA의 데이터 타입은 크게 엔티티 타입과 값 타입으로 나눌 수 있다.

### 엔티티 타입

엔티티 타입은 `@Entity`로 정의하는 객체이다.

식별자가 있으므로 데이터가 변경되어도 식별자로 계속 추적할 수 있다.

### 값 타입

값 타입은 식별자가 없고 값 자체가 중요한 타입이다.

예를 들어 `int`, `Integer`, `String`, `Address` 같은 값이 있다.

값 타입은 값이 변경되면 완전히 다른 값으로 대체된다고 이해하는 것이 좋다.

---

## 12. 임베디드 타입

임베디드 타입은 여러 값을 묶어서 의미 있는 값 객체로 만든 것이다.

예를 들어 회원 엔티티에 주소 관련 필드가 있다고 하자.

```java
private String city;
private String street;
private String zipcode;
```

이 필드들을 `Address`라는 값 타입으로 묶을 수 있다.

```java
@Embeddable
public class Address {
    private String city;
    private String street;
    private String zipcode;
}
```

사용하는 엔티티에서는 `@Embedded`를 붙인다.

```java
@Embedded
private Address homeAddress;
```

임베디드 타입을 사용하면 객체 모델을 더 의미 있게 만들 수 있다.

---

## 13. 값 타입과 불변 객체

값 타입은 공유하면 위험하다.

예를 들어 두 회원이 같은 `Address` 인스턴스를 공유한다고 하자.

한 회원의 주소를 변경했는데 다른 회원의 주소도 함께 바뀌면 큰 문제가 된다.

값 타입은 가급적 불변 객체로 만들어야 한다.

불변 객체 설계 방법은 다음과 같다.

- setter를 만들지 않는다.
- 필드를 final로 둔다.
- 생성자에서 값을 모두 받는다.
- 변경이 필요하면 새 객체를 만들어 교체한다.

```java
Address newAddress = new Address("newCity", address.getStreet(), address.getZipcode());
member.setHomeAddress(newAddress);
```

값 타입은 변경하는 것이 아니라 교체하는 방식으로 다루는 것이 안전하다.

---

## 14. 값 타입 비교

값 타입은 식별자가 없으므로 값 자체를 비교해야 한다.

따라서 `equals()`와 `hashCode()`를 적절히 재정의해야 한다.

엔티티는 식별자로 구분하지만, 값 타입은 모든 값이 같으면 같은 값으로 보는 것이 자연스럽다.

---

## 15. 값 타입 컬렉션

값 타입 컬렉션은 값 타입을 컬렉션으로 가지는 구조이다.

```java
@ElementCollection
@CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name = "MEMBER_ID"))
@Column(name = "FOOD_NAME")
private Set<String> favoriteFoods = new HashSet<>();
```

값 타입 컬렉션은 별도 테이블에 저장된다.

값 타입은 엔티티가 아니므로 식별자가 없다.

따라서 값 타입 컬렉션을 수정할 때는 전체 삭제 후 다시 삽입되는 방식으로 동작할 수 있어 주의가 필요하다.

실무에서는 값 타입 컬렉션 대신 일대다 관계의 엔티티로 풀어내는 것을 고려하는 경우가 많다.

---

## 16. 보충 정리

- 모든 연관관계는 기본적으로 지연 로딩을 사용하는 것이 좋다.
- 즉시 로딩은 예상하지 못한 SQL과 N+1 문제를 만들 수 있다.
- 프록시 초기화는 영속성 컨텍스트가 살아 있을 때만 가능하다.
- OSIV를 켜면 뷰에서도 지연 로딩이 가능하지만, 트랜잭션 범위와 성능 문제를 함께 고려해야 한다.
- cascade는 소유 관계가 명확할 때만 사용한다.
- orphanRemoval은 부모가 자식을 완전히 소유할 때만 사용한다.
- 값 타입은 불변으로 설계한다.
- 값 타입 컬렉션은 편리하지만 실무에서는 엔티티로 승격하는 것이 더 나은 경우가 많다.

---

## 17. 오늘 정리

프록시는 지연 로딩을 가능하게 해주지만, 영속성 컨텍스트 범위를 벗어나면 문제가 생길 수 있다.

값 타입은 단순한 값으로 보이지만 공유 참조와 변경 가능성 때문에 불변 설계가 중요하다.

중요한 정리는 다음과 같다.

- `em.find()`는 실제 엔티티를 조회한다.
- `em.getReference()`는 프록시 객체를 반환한다.
- 프록시는 실제 사용할 때 초기화된다.
- 지연 로딩은 필요한 시점에 연관 엔티티를 조회한다.
- 즉시 로딩은 실무에서 예측하기 어려우므로 주의해야 한다.
- N+1 문제는 페치 조인으로 해결할 수 있다.
- cascade는 생명주기가 같은 엔티티 사이에서 사용한다.
- orphanRemoval은 부모가 자식을 완전히 소유할 때 사용한다.
- 값 타입은 식별자가 없고 값 자체가 중요하다.
- 임베디드 타입은 의미 있는 값 객체를 만들 때 사용한다.
- 값 타입은 불변 객체로 설계하는 것이 안전하다.
