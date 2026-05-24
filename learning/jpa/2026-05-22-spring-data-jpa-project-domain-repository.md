# Spring Data JPA 프로젝트 환경설정, 도메인 모델, 공통 인터페이스 정리

## 학습 날짜

2026-05-22

## 학습 주제

- 프로젝트 환경설정
- Spring Boot 3.x 주의사항
- H2 데이터베이스 설정
- 예제 도메인 모델
- Member, Team 연관관계
- 순수 JPA 리포지토리
- Spring Data JPA 공통 인터페이스
- `JpaRepository`
- 기본 CRUD 메서드

---

## 1. 프로젝트 환경설정

Spring Data JPA 학습 프로젝트는 Spring Boot, Spring Data JPA, H2, Lombok을 사용한다.

Spring Boot 3.x 기준으로는 다음 사항을 확인해야 한다.

- Java 17 이상 필요
- `javax` 패키지 대신 `jakarta` 패키지 사용
- H2 데이터베이스 2.1.214 이상 사용 권장
- Gradle 빌드와 테스트 실행도 Gradle을 사용하도록 설정 권장

예를 들어 JPA 애노테이션 패키지는 다음처럼 변경된다.

```java
// 기존
import javax.persistence.Entity;

// Spring Boot 3.x 이후
import jakarta.persistence.Entity;
```

`@PostConstruct`, Validation 관련 패키지도 동일하게 `jakarta` 계열로 변경된다.

---

## 2. 주요 라이브러리

프로젝트에서 사용하는 주요 라이브러리는 다음과 같다.

### Spring Web

웹 MVC와 내장 톰캣을 제공한다.

### Spring Data JPA

Spring Data JPA, Spring ORM, Hibernate, JDBC, HikariCP를 포함해 JPA 기반 데이터 접근을 쉽게 할 수 있게 해준다.

### H2 Database

개발과 테스트에 사용하기 좋은 가벼운 데이터베이스이다.

### Lombok

Getter, Setter, 생성자 등을 애노테이션으로 줄여준다.

다만 엔티티에서 Lombok을 사용할 때는 주의해야 한다. 특히 `@Setter`를 무분별하게 열어두면 엔티티의 변경 지점이 많아지므로 실무에서는 필요한 메서드만 제공하는 것이 좋다.

---

## 3. H2 데이터베이스 설정

H2는 개발과 테스트에 적합한 데이터베이스이다.

Spring Boot 3.x를 사용한다면 H2 2.1.214 이상을 사용하는 것이 좋다.

일반적인 접속 흐름은 다음과 같다.

```text
jdbc:h2:~/datajpa
```

최초 한 번 파일을 생성한 뒤에는 TCP 모드로 접속할 수 있다.

```text
jdbc:h2:tcp://localhost/~/datajpa
```

`application.yml` 예시는 다음과 같다.

```yaml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/datajpa
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        format_sql: true

logging.level:
  org.hibernate.SQL: debug
```

학습 환경에서는 `ddl-auto: create`로 테이블을 매번 새로 만들 수 있다.

다만 운영 환경에서는 `create`, `create-drop`, `update`를 사용하는 것은 위험하다.

---

## 4. 예제 도메인 모델

예제 도메인은 `Member`와 `Team`으로 구성된다.

관계는 다음과 같다.

- 하나의 팀에는 여러 회원이 속할 수 있다.
- 한 명의 회원은 하나의 팀에 속할 수 있다.
- 객체 관계는 양방향이다.
- 테이블 관계는 `member.team_id` 외래 키로 연결된다.

객체에서는 다음 관계가 있다.

```text
Member -> Team
Team -> List<Member>
```

테이블에서는 다음 관계가 있다.

```text
member.team_id -> team.team_id
```

---

## 5. Member 엔티티

`Member` 엔티티는 회원 정보를 표현한다.

```java
@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id", "username", "age"})
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String username;

    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;

    public Member(String username, int age, Team team) {
        this.username = username;
        this.age = age;
        if (team != null) {
            changeTeam(team);
        }
    }

    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}
```

중요한 부분은 다음과 같다.

- `@ManyToOne(fetch = FetchType.LAZY)`로 팀을 지연 로딩한다.
- `@JoinColumn(name = "team_id")`로 외래 키 컬럼을 지정한다.
- `Member.team`이 연관관계의 주인이다.
- `changeTeam()`으로 양방향 연관관계를 한 번에 맞춘다.
- `@ToString`에는 연관관계 필드를 제외한다.

---

## 6. Team 엔티티

`Team` 엔티티는 팀 정보를 표현한다.

```java
@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id", "name"})
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "team_id")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    public Team(String name) {
        this.name = name;
    }
}
```

`Team.members`는 연관관계의 주인이 아니다.

`mappedBy = "team"`은 `Member` 엔티티의 `team` 필드가 외래 키를 관리한다는 의미이다.

따라서 외래 키 변경은 `Member.team`을 통해 이루어진다.

---

## 7. 연관관계 편의 메서드

양방향 연관관계에서는 양쪽 객체 참조를 함께 맞춰주는 것이 좋다.

```java
public void changeTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
}
```

이 메서드를 사용하면 회원의 팀도 변경되고, 팀의 회원 목록에도 현재 회원이 추가된다.

다만 실무에서는 기존 팀에서 제거하는 로직까지 고려해야 한다.

```java
public void changeTeam(Team newTeam) {
    if (this.team != null) {
        this.team.getMembers().remove(this);
    }

    this.team = newTeam;

    if (newTeam != null) {
        newTeam.getMembers().add(this);
    }
}
```

이렇게 하면 팀 변경 시 양쪽 컬렉션 상태가 더 안전하게 유지된다.

---

## 8. 순수 JPA 리포지토리

Spring Data JPA를 사용하기 전에는 `EntityManager`를 직접 사용해 리포지토리를 구현할 수 있다.

```java
@Repository
public class MemberJpaRepository {

    @PersistenceContext
    private EntityManager em;

    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    public void delete(Member member) {
        em.remove(member);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    public long count() {
        return em.createQuery("select count(m) from Member m", Long.class)
                .getSingleResult();
    }
}
```

순수 JPA 리포지토리는 동작 원리를 이해하는 데 좋다.

하지만 엔티티마다 비슷한 CRUD 코드가 반복된다.

---

## 9. Spring Data JPA 공통 인터페이스

Spring Data JPA를 사용하면 반복되는 CRUD 코드를 직접 작성하지 않아도 된다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

`JpaRepository<Member, Long>`에서 제네릭 의미는 다음과 같다.

- `Member`: 엔티티 타입
- `Long`: 엔티티 식별자 타입

이 인터페이스만 작성해도 Spring Data JPA가 구현체를 자동으로 만들어 스프링 빈으로 등록한다.

---

## 10. Spring Data JPA 리포지토리가 동작하는 이유

Spring Data JPA는 리포지토리 인터페이스를 보고 구현체를 동적으로 생성한다.

실제 객체는 프록시 객체이다.

Spring Data JPA는 다음을 자동으로 처리한다.

- 리포지토리 인터페이스 스캔
- 구현체 프록시 생성
- 스프링 빈 등록
- JPA 예외를 스프링 데이터 접근 예외로 변환

따라서 `@Repository`를 직접 붙이지 않아도 동작한다.

---

## 11. JpaRepository 주요 메서드

`JpaRepository`는 공통 CRUD 메서드를 제공한다.

### save

```java
memberRepository.save(member);
```

새로운 엔티티는 저장하고, 이미 존재하는 엔티티는 병합한다.

### findById

```java
Optional<Member> member = memberRepository.findById(id);
```

식별자로 엔티티를 조회하고, 결과가 없을 수 있으므로 `Optional`을 반환한다.

### findAll

```java
List<Member> members = memberRepository.findAll();
```

전체 엔티티를 조회한다.

### delete

```java
memberRepository.delete(member);
```

엔티티를 삭제한다.

### count

```java
long count = memberRepository.count();
```

전체 엔티티 수를 조회한다.

### getReferenceById

```java
Member reference = memberRepository.getReferenceById(id);
```

프록시로 엔티티를 조회한다.

과거에는 `getOne()`을 많이 사용했지만, 최신 흐름에서는 `getReferenceById()`를 사용하는 것으로 이해하는 것이 좋다.

---

## 12. save 사용 시 주의

`save()`는 항상 INSERT만 실행하는 메서드가 아니다.

Spring Data JPA 내부에서는 새로운 엔티티인지 확인한 뒤 다음처럼 동작한다.

- 새로운 엔티티: `em.persist()`
- 기존 엔티티: `em.merge()`

따라서 수정할 때 무조건 `save()`를 호출하기보다, 트랜잭션 안에서 엔티티를 조회한 후 값을 변경하고 변경 감지를 사용하는 것이 JPA다운 방식이다.

```java
@Transactional
public void updateMember(Long id, String username) {
    Member member = memberRepository.findById(id)
            .orElseThrow();
    member.setUsername(username);
}
```

트랜잭션 커밋 시점에 변경 감지가 동작해 UPDATE SQL이 실행된다.

---

## 13. 보충 정리

Spring Data JPA는 JPA를 대체하는 기술이 아니라, JPA 리포지토리 구현을 편리하게 만들어주는 기술이다.

따라서 내부 동작을 이해하려면 JPA 기본 개념이 필요하다.

특히 다음 내용을 함께 이해해야 한다.

- 영속성 컨텍스트
- 변경 감지
- 트랜잭션
- 프록시
- 지연 로딩
- `persist`와 `merge` 차이

Spring Data JPA는 CRUD를 쉽게 만들어주지만, 잘못 사용하면 `save()`를 수정용 메서드처럼 남용하거나, 프록시/지연 로딩 문제를 놓칠 수 있다.

---

## 14. 오늘 정리

프로젝트 환경설정에서는 Spring Boot 3.x 기준으로 Java 17, `jakarta` 패키지, H2 버전 호환성을 확인해야 한다.

예제 도메인에서는 `Member`와 `Team`의 양방향 연관관계를 통해 JPA의 연관관계 주인 개념을 다시 복습했다.

공통 인터페이스 기능에서는 `JpaRepository`를 상속하는 것만으로 기본 CRUD 구현체를 사용할 수 있음을 확인했다.

중요한 정리는 다음과 같다.

- Spring Data JPA는 리포지토리 구현체를 자동 생성한다.
- `JpaRepository<Entity, ID>`를 상속하면 공통 CRUD를 사용할 수 있다.
- `Member.team`이 외래 키를 관리하는 연관관계의 주인이다.
- 양방향 연관관계에서는 편의 메서드로 양쪽 참조를 맞춰주는 것이 좋다.
- 수정은 `save()`보다 변경 감지를 사용하는 것이 JPA답다.
- 최신 프록시 조회 메서드는 `getReferenceById()`로 이해하면 된다.
- Spring Data JPA를 잘 사용하려면 JPA 기본 동작 원리를 함께 알아야 한다.
