# Querydsl 프로젝트 환경설정과 예제 도메인 모델 정리

## 학습 날짜

2026-05-23

## 학습 주제

- Querydsl 프로젝트 생성
- Spring Boot 3.x 설정
- Java 17, Jakarta 전환
- Querydsl 5.0 Gradle 설정
- Q-Type 생성과 검증
- H2 데이터베이스 설정
- 예제 도메인 모델
- Member, Team 연관관계
- 양방향 연관관계 편의 메서드

---

## 1. Querydsl 학습의 전체 흐름

실전 Querydsl 학습은 다음 흐름으로 진행된다.

1. 프로젝트 환경설정
2. 예제 도메인 모델 구성
3. Querydsl 기본 문법
4. Querydsl 중급 문법
5. 순수 JPA와 Querydsl 실무 활용
6. Spring Data JPA와 Querydsl 실무 활용
7. Spring Data JPA가 제공하는 Querydsl 기능 검토
8. Spring Boot 3.x와 Querydsl 5.0 지원 방법 확인

Querydsl은 JPA를 대체하는 기술이 아니다.

JPA와 JPQL을 더 타입 안전하고 편리하게 사용하기 위한 쿼리 빌더로 이해하는 것이 좋다.

---

## 2. Spring Boot 3.x 프로젝트 생성 기준

Spring Boot 3.x 기준 프로젝트에서는 다음 조건을 확인해야 한다.

- Java 17 이상 사용
- Spring Boot 3.x 선택
- Gradle - Groovy Project 사용 가능
- Spring Web 추가
- Spring Data JPA 추가
- H2 Database 추가
- Lombok 추가
- Querydsl 설정 추가

Spring Boot 3.x부터는 `javax` 패키지 대신 `jakarta` 패키지를 사용한다.

예를 들어 JPA 엔티티 애노테이션 import는 다음처럼 바뀐다.

```java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
```

기존 Spring Boot 2.x 자료를 참고할 때는 `javax.persistence`를 `jakarta.persistence`로 바꿔야 하는 부분을 주의해야 한다.

---

## 3. Querydsl 5.0 Gradle 설정

Spring Boot 3.x와 Querydsl 5.0을 사용할 때는 jakarta classifier가 포함된 의존성을 사용한다.

예시는 다음과 같다.

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'

    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
    annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jakarta"
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"

    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
}
```

Querydsl은 엔티티를 기반으로 Q-Type을 생성한다.

Q-Type은 쿼리 작성 시 엔티티 필드에 타입 안전하게 접근할 수 있게 해주는 클래스이다.

---

## 4. Q-Type 생성 파일 관리

Querydsl이 생성하는 Q-Type은 빌드 결과물이다.

따라서 일반적으로 Git에 포함하지 않는다.

생성 경로를 `src/main/generated`로 잡았다면 `clean` 시 삭제되도록 설정할 수 있다.

```groovy
clean {
    delete file('src/main/generated')
}
```

보충하면, 팀 프로젝트에서는 Q-Type 생성 경로와 IDE 인식 설정을 팀원들이 동일하게 맞추는 것이 중요하다.

Q-Type이 생성되지 않으면 Querydsl 코드를 작성할 수 없으므로, 빌드 설정과 annotation processing 설정을 반드시 확인해야 한다.

---

## 5. Querydsl 동작 검증

간단한 엔티티를 저장하고 Q-Type으로 조회하면 Querydsl 설정이 정상인지 확인할 수 있다.

```java
@SpringBootTest
@Transactional
class QuerydslApplicationTests {

    @Autowired
    EntityManager em;

    @Test
    void contextLoads() {
        Hello hello = new Hello();
        em.persist(hello);

        JPAQueryFactory query = new JPAQueryFactory(em);
        QHello qHello = QHello.hello;

        Hello result = query
                .selectFrom(qHello)
                .fetchOne();

        assertThat(result).isEqualTo(hello);
    }
}
```

이 테스트에서 확인할 수 있는 것은 다음과 같다.

- JPA가 정상 동작하는지
- Querydsl Q-Type이 생성되었는지
- `JPAQueryFactory`로 쿼리를 실행할 수 있는지
- Lombok이 정상 동작하는지

---

## 6. 예제 도메인 모델

예제 도메인은 `Member`와 `Team`으로 구성된다.

관계는 다음과 같다.

- 하나의 팀에는 여러 회원이 속할 수 있다.
- 한 명의 회원은 하나의 팀에 속할 수 있다.
- 객체는 양방향 관계이다.
- 테이블은 `member.team_id` 외래 키로 관계를 맺는다.

객체 관계는 다음처럼 볼 수 있다.

```text
Member N : 1 Team
Member.team
Team.members
```

테이블 관계는 다음과 같다.

```text
member.team_id -> team.team_id
```

---

## 7. Member 엔티티 핵심

`Member` 엔티티에서는 팀과 다대일 관계를 맺는다.

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "team_id")
private Team team;
```

중요한 점은 다음과 같다.

- `@ManyToOne(fetch = FetchType.LAZY)`로 지연 로딩을 설정한다.
- `@JoinColumn(name = "team_id")`로 외래 키 컬럼을 지정한다.
- `Member.team`이 연관관계의 주인이다.
- `changeTeam()`은 양방향 연관관계를 한 번에 맞추기 위한 편의 메서드이다.
- `@ToString`에는 연관관계 필드를 제외한다.

---

## 8. Team 엔티티 핵심

`Team` 엔티티는 회원 목록을 가진다.

```java
@OneToMany(mappedBy = "team")
private List<Member> members = new ArrayList<>();
```

`mappedBy = "team"`은 `Member`의 `team` 필드가 연관관계의 주인이라는 뜻이다.

즉, `Team.members`는 외래 키를 직접 변경하지 않는다.

외래 키 값은 `Member.team`이 변경될 때 반영된다.

---

## 9. 연관관계 편의 메서드 보충

기본 편의 메서드는 다음과 같다.

```java
public void changeTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
}
```

다만 실무에서는 기존 팀에서 현재 회원을 제거하는 로직까지 고려할 수 있다.

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

이렇게 하면 팀 변경 시 기존 팀의 회원 목록과 새 팀의 회원 목록이 더 일관되게 유지된다.

---

## 10. Lombok 사용 시 주의

예제에서는 학습 편의를 위해 `@Setter`를 사용한다.

하지만 실무에서는 엔티티에 setter를 무분별하게 열어두지 않는 것이 좋다.

이유는 다음과 같다.

- 어디서 값이 변경되는지 추적하기 어렵다.
- 도메인 규칙 없이 필드가 변경될 수 있다.
- 엔티티 상태 변경 로직이 분산된다.

실무에서는 의미 있는 메서드를 만들어 변경 의도를 드러내는 것이 좋다.

```java
public void changeUsername(String username) {
    this.username = username;
}
```

또한 `@ToString`은 연관관계 필드를 포함하지 않는 것이 좋다.

양방향 연관관계에서 `toString()`이 서로를 계속 호출하면 무한 순환이 발생할 수 있다.

---

## 오늘 정리

Querydsl 프로젝트를 시작할 때는 먼저 환경설정과 Q-Type 생성 여부를 확인해야 한다.

Spring Boot 3.x에서는 Java 17 이상, jakarta 패키지, Querydsl 5.0 설정이 중요하다.

예제 도메인은 `Member`와 `Team`으로 구성되며, JPA 연관관계와 Querydsl 테스트의 기반이 된다.

중요한 정리는 다음과 같다.

- Querydsl은 JPA와 함께 사용하는 타입 안전한 쿼리 빌더이다.
- Spring Boot 3.x에서는 `jakarta` 패키지와 Querydsl 5.0 설정을 맞춰야 한다.
- Q-Type은 엔티티를 기반으로 생성되는 Querydsl 전용 클래스이다.
- Q-Type은 빌드 결과물이므로 Git에 포함하지 않는 것이 일반적이다.
- `Member.team`이 연관관계의 주인이다.
- `Team.members`는 `mappedBy`로 읽기 쪽 관계를 표현한다.
- 양방향 연관관계는 편의 메서드로 양쪽 참조를 맞추는 것이 좋다.
- 엔티티에서 `@ToString`과 `@Setter` 사용은 실무에서 주의해야 한다.
