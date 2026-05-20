# JPA 연관관계 매핑과 고급 매핑 정리

## 학습 날짜

2026-05-20

## 학습 주제

- 연관관계가 필요한 이유
- 단방향 연관관계
- 양방향 연관관계
- 연관관계의 주인
- 다대일, 일대다, 일대일, 다대다
- 상속관계 매핑
- 조인 전략, 단일 테이블 전략, 구현 클래스마다 테이블 전략
- `@MappedSuperclass`

---

## 1. 연관관계가 필요한 이유

객체지향 설계의 목표는 객체들이 서로 협력하는 구조를 만드는 것이다.

객체는 참조를 통해 다른 객체와 협력한다.

반면 관계형 데이터베이스는 외래 키와 조인을 통해 테이블 간 관계를 맺는다.

만약 객체를 테이블에 맞춰 모델링해서 외래 키 ID만 필드로 가지고 있으면 객체다운 탐색이 어렵다.

예를 들어 회원이 팀에 속하는 관계에서 `Member`가 `teamId`만 가지고 있다면, 팀을 조회하려면 다시 `Team`을 별도로 조회해야 한다.

```java
Member member = em.find(Member.class, memberId);
Team team = em.find(Team.class, member.getTeamId());
```

이 방식은 객체가 참조로 협력하는 구조가 아니다.

JPA의 연관관계 매핑은 객체의 참조와 테이블의 외래 키를 연결해준다.

---

## 2. 단방향 연관관계

회원과 팀이 다대일 관계라고 하자.

회원은 하나의 팀에 소속되고, 하나의 팀에는 여러 회원이 있을 수 있다.

가장 기본적인 매핑은 다대일 단방향이다.

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String username;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

`@ManyToOne`은 여러 회원이 하나의 팀을 참조한다는 의미이다.

`@JoinColumn(name = "TEAM_ID")`는 MEMBER 테이블의 TEAM_ID 외래 키와 매핑한다.

이제 회원에서 팀을 객체 참조로 탐색할 수 있다.

```java
Member member = em.find(Member.class, memberId);
Team team = member.getTeam();
```

이것을 객체 그래프 탐색이라고 볼 수 있다.

---

## 3. 연관관계 저장과 수정

연관관계를 저장할 때는 외래 키 값을 직접 넣는 것이 아니라 참조를 설정한다.

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
member.setTeam(team);
em.persist(member);
```

JPA는 `member.setTeam(team)`으로 설정된 참조를 보고 MEMBER 테이블의 TEAM_ID 외래 키 값을 저장한다.

연관관계 수정도 참조를 변경하면 된다.

```java
Team teamB = new Team();
teamB.setName("TeamB");
em.persist(teamB);

member.setTeam(teamB);
```

영속 상태의 엔티티라면 변경 감지를 통해 외래 키 변경 SQL이 실행된다.

---

## 4. 양방향 연관관계

객체에서 양쪽 방향으로 탐색하고 싶으면 양방향 연관관계를 구성할 수 있다.

회원에서 팀을 참조하고, 팀에서도 회원 목록을 참조한다.

```java
@Entity
public class Team {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```

`mappedBy = "team"`은 이 컬렉션이 연관관계의 주인이 아니라는 뜻이다.

여기서 `team`은 `Member` 클래스의 `team` 필드 이름이다.

---

## 5. 연관관계의 주인

테이블의 양방향 관계는 외래 키 하나로 양쪽 조인이 가능하다.

하지만 객체의 양방향 관계는 서로 다른 참조가 두 개 있다.

예를 들어 다음 두 참조가 있다.

- `Member.team`
- `Team.members`

둘 중 어느 참조가 외래 키를 관리할지 정해야 한다.

이것이 연관관계의 주인이다.

규칙은 다음과 같다.

- 연관관계의 주인만 외래 키를 관리한다.
- 주인이 아닌 쪽은 읽기만 가능하다.
- 주인은 `mappedBy`를 사용하지 않는다.
- 주인이 아닌 쪽에 `mappedBy`를 사용한다.
- 외래 키가 있는 쪽을 주인으로 정하는 것이 기본이다.

회원과 팀 관계에서는 MEMBER 테이블에 TEAM_ID 외래 키가 있으므로 `Member.team`이 연관관계의 주인이다.

---

## 6. 양방향 연관관계 주의사항

양방향 연관관계에서는 주인 쪽에 값을 설정해야 DB에 반영된다.

```java
member.setTeam(team);
```

반대편 컬렉션에만 값을 넣으면 외래 키가 변경되지 않는다.

```java
team.getMembers().add(member); // 이것만으로는 외래 키 관리 불가
```

객체 관점에서는 양쪽 모두 값을 맞춰주는 것이 좋다.

그래서 연관관계 편의 메서드를 만든다.

```java
public void changeTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
}
```

실무에서는 양방향 연관관계를 무조건 만들기보다, 먼저 단방향으로 설계하고 필요한 경우에만 양방향을 추가하는 것이 좋다.

---

## 7. 다대일 N:1

다대일은 가장 많이 사용하는 연관관계이다.

예를 들어 여러 회원이 하나의 팀에 속한다.

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "TEAM_ID")
private Team team;
```

다대일 관계에서는 다 쪽에 외래 키가 있다.

따라서 다 쪽인 `Member`가 연관관계의 주인이 된다.

실무에서는 다대일 단방향을 기본으로 시작하고, 반대 방향 조회가 필요하면 양방향을 추가하는 방식이 좋다.

---

## 8. 일대다 1:N

일대다 단방향은 일 쪽이 연관관계의 주인이 되는 구조이다.

하지만 테이블에서는 항상 다 쪽에 외래 키가 있다.

예를 들어 팀이 회원 목록을 관리하지만, 실제 외래 키는 MEMBER 테이블에 있다.

이 구조는 객체가 관리하는 외래 키가 다른 테이블에 있는 특이한 구조가 된다.

단점은 다음과 같다.

- 연관관계 관리를 위해 추가 UPDATE SQL이 실행될 수 있다.
- 매핑이 직관적이지 않다.
- 운영 중 SQL 흐름을 예측하기 어려워질 수 있다.

따라서 실무에서는 일대다 단방향보다 다대일 양방향 매핑을 권장한다.

---

## 9. 일대일 1:1

일대일 관계는 양쪽 중 어디에 외래 키를 둘지 선택할 수 있다.

예를 들어 회원과 락커가 일대일 관계라고 하자.

주 테이블인 MEMBER에 LOCKER_ID 외래 키를 둘 수도 있고, 대상 테이블인 LOCKER에 MEMBER_ID 외래 키를 둘 수도 있다.

객체 관계에서는 외래 키가 있는 쪽을 연관관계의 주인으로 정한다.

일대일 관계도 다대일과 마찬가지로 외래 키가 있는 곳이 주인이다.

---

## 10. 다대다 N:M

관계형 데이터베이스는 정규화된 테이블 2개만으로 다대다 관계를 표현할 수 없다.

중간 연결 테이블이 필요하다.

JPA는 `@ManyToMany`를 제공하지만 실무에서는 사용하지 않는 것이 좋다.

이유는 다음과 같다.

- 연결 테이블에 추가 컬럼을 넣기 어렵다.
- 실무의 연결 테이블은 대부분 단순하지 않다.
- 엔티티와 테이블 구조가 맞지 않는다.
- 쿼리와 운영이 복잡해질 수 있다.

따라서 다대다 관계는 연결 엔티티를 만들어 일대다, 다대일 관계로 풀어내야 한다.

---

## 11. 상속관계 매핑

관계형 데이터베이스에는 객체 상속 개념이 없다.

대신 슈퍼타입-서브타입 관계라는 모델링 기법이 객체 상속과 유사하다.

JPA의 상속관계 매핑은 객체의 상속 구조와 DB의 슈퍼타입-서브타입 구조를 매핑하는 기능이다.

예를 들어 `Item`을 부모로 두고 `Album`, `Movie`, `Book`이 자식 엔티티가 될 수 있다.

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private int price;
}
```

---

## 12. 상속 전략 비교

### 조인 전략

부모 테이블과 자식 테이블을 각각 만들고 조회할 때 조인한다.

장점은 정규화, 저장 공간 효율, 외래 키 참조 무결성이다.

단점은 조회 쿼리가 복잡하고 저장 시 INSERT가 2번 발생할 수 있다는 점이다.

### 단일 테이블 전략

부모와 자식을 하나의 테이블에 모두 저장한다.

장점은 조인이 필요 없어 조회가 단순하고 빠를 수 있다는 점이다.

단점은 자식별 컬럼이 null 허용이 되고 테이블이 커질 수 있다는 점이다.

### 구현 클래스마다 테이블 전략

자식 클래스마다 테이블을 따로 만든다.

여러 자식 테이블을 함께 조회할 때 UNION이 필요하므로 실무에서는 권장하지 않는다.

---

## 13. @MappedSuperclass

`@MappedSuperclass`는 공통 매핑 정보가 필요할 때 사용한다.

예를 들어 여러 엔티티가 공통으로 다음 필드를 가진다고 하자.

- createdBy
- createdDate
- lastModifiedBy
- lastModifiedDate

부모 클래스로 분리할 수 있다.

```java
@MappedSuperclass
public abstract class BaseEntity {
    private String createdBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
}
```

중요한 점은 `@MappedSuperclass` 자체는 엔티티가 아니고 테이블과 직접 매핑되지 않는다는 것이다.

따라서 `em.find(BaseEntity.class, id)`처럼 조회할 수 없다.

직접 생성해서 사용할 일이 거의 없으므로 추상 클래스로 만드는 것이 좋다.

---

## 14. 보충 정리

- 양방향 관계는 꼭 필요한 경우에만 사용한다.
- JSON 직렬화 시 양방향 관계는 무한 순환 문제가 생길 수 있으므로 DTO로 응답하는 것이 좋다.
- 연관관계 편의 메서드는 한쪽에만 두고 일관되게 사용한다.
- `@ManyToOne`은 기본 fetch가 EAGER이므로 실무에서는 LAZY로 명시하는 것이 좋다.
- `@OneToMany`는 컬렉션이므로 기본 fetch가 LAZY이다.
- `@ManyToMany`는 실무에서 연결 엔티티로 풀어내는 것이 안전하다.
- 상속관계 매핑은 객체 설계와 조회 성능을 함께 고려해야 한다.
- 공통 필드는 `@MappedSuperclass`로 분리할 수 있지만, 이것은 상속관계 매핑이 아니다.

---

## 15. 오늘 정리

연관관계 매핑은 JPA에서 가장 중요한 부분 중 하나이다.

객체는 참조로 관계를 맺고, 테이블은 외래 키로 관계를 맺는다.

JPA는 이 둘을 연결해주지만, 방향과 주인 개념은 개발자가 정확히 이해해야 한다.

중요한 정리는 다음과 같다.

- 연관관계는 먼저 단방향으로 충분히 설계한다.
- 양방향이 필요하면 반대편 참조를 추가한다.
- 양방향에서는 외래 키를 관리하는 연관관계의 주인이 필요하다.
- 외래 키가 있는 쪽을 주인으로 정한다.
- 다대일 단방향이 가장 기본적인 연관관계이다.
- 일대다 단방향보다 다대일 양방향을 권장한다.
- 다대다는 연결 엔티티로 풀어낸다.
- 상속관계 매핑 전략은 조인, 단일 테이블, 구현 클래스마다 테이블 전략이 있다.
- `@MappedSuperclass`는 공통 매핑 정보만 제공한다.
