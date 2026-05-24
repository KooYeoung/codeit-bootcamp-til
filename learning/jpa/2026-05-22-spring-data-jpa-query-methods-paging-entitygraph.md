# Spring Data JPA 쿼리 메소드, @Query, 페이징, EntityGraph 정리

## 학습 날짜

2026-05-22

## 학습 주제

- 메서드 이름으로 쿼리 생성
- JPA NamedQuery
- `@Query`
- 값 조회와 DTO 조회
- 파라미터 바인딩
- 반환 타입
- 페이징과 정렬
- `Page`, `Slice`, `List`
- 벌크성 수정 쿼리
- `@EntityGraph`
- JPA Hint
- Lock

---

## 1. 쿼리 메소드 기능

Spring Data JPA는 리포지토리 메서드 이름을 분석해서 JPQL을 자동으로 생성할 수 있다.

예를 들어 이름이 `AAA`이고 나이가 15보다 큰 회원을 조회하려면 순수 JPA에서는 다음처럼 작성한다.

```java
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
    return em.createQuery(
            "select m from Member m where m.username = :username and m.age > :age",
            Member.class
    )
    .setParameter("username", username)
    .setParameter("age", age)
    .getResultList();
}
```

Spring Data JPA에서는 인터페이스에 메서드만 선언하면 된다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

Spring Data JPA가 메서드 이름을 분석해 JPQL을 생성하고 실행한다.

---

## 2. 메서드 이름 기반 쿼리의 장점

메서드 이름 기반 쿼리의 장점은 다음과 같다.

- 간단한 조건 조회를 빠르게 작성할 수 있다.
- JPQL을 직접 작성하지 않아도 된다.
- 엔티티 필드명이 틀리면 애플리케이션 로딩 시점에 오류를 확인할 수 있다.
- `findBy`, `countBy`, `existsBy`, `deleteBy` 같은 규칙을 활용할 수 있다.

대표 기능은 다음과 같다.

```java
List<Member> findByUsername(String username);

long countByAge(int age);

boolean existsByUsername(String username);

long deleteByUsername(String username);

List<Member> findTop3ByAgeOrderByUsernameDesc(int age);
```

---

## 3. 메서드 이름 기반 쿼리의 한계

메서드 이름 기반 쿼리는 조건이 단순할 때 편리하다.

하지만 조건이 많아지면 메서드 이름이 너무 길어진다.

```java
findByUsernameAndAgeGreaterThanAndTeamNameOrderByCreatedDateDesc(...)
```

이런 이름은 읽기 어렵고 유지보수하기도 어렵다.

따라서 실무에서는 기준을 나누는 것이 좋다.

- 조건이 단순하다: 메서드 이름 기반 쿼리
- 조건이 조금 복잡하거나 정적 쿼리다: `@Query`
- 동적 조건이 많다: Querydsl
- 특정 DB 기능이 필요하다: Native SQL 또는 JdbcTemplate

---

## 4. JPA NamedQuery

JPA는 엔티티에 미리 이름 있는 쿼리를 정의할 수 있다.

```java
@Entity
@NamedQuery(
    name = "Member.findByUsername",
    query = "select m from Member m where m.username = :username"
)
public class Member {
}
```

Spring Data JPA는 도메인 클래스 이름과 메서드 이름을 조합해 NamedQuery를 찾을 수 있다.

```java
List<Member> findByUsername(@Param("username") String username);
```

하지만 실무에서는 NamedQuery를 직접 등록하기보다 `@Query`를 리포지토리 메서드에 직접 작성하는 경우가 많다.

---

## 5. @Query

`@Query`는 리포지토리 메서드에 JPQL을 직접 작성하는 방법이다.

```java
@Query("select m from Member m where m.username = :username and m.age = :age")
List<Member> findUser(@Param("username") String username, @Param("age") int age);
```

장점은 다음과 같다.

- 메서드 이름이 길어지는 문제를 줄일 수 있다.
- JPQL을 명확하게 작성할 수 있다.
- 애플리케이션 실행 시점에 문법 오류를 확인할 수 있다.
- DTO 조회나 조인 조회를 명확히 표현할 수 있다.

실무에서는 조건이 조금만 복잡해져도 `@Query`를 자주 사용하게 된다.

---

## 6. 값 조회와 DTO 조회

단순 값 하나를 조회할 수 있다.

```java
@Query("select m.username from Member m")
List<String> findUsernameList();
```

DTO로 직접 조회할 수도 있다.

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
       "from Member m join m.team t")
List<MemberDto> findMemberDto();
```

DTO 조회 시 주의할 점은 다음과 같다.

- JPQL의 `new` 명령어를 사용해야 한다.
- DTO의 패키지명을 포함해야 한다.
- DTO에 맞는 생성자가 필요하다.

---

## 7. 파라미터 바인딩

JPQL 파라미터 바인딩은 위치 기반과 이름 기반이 있다.

```sql
select m from Member m where m.username = ?1
select m from Member m where m.username = :username
```

실무에서는 이름 기반 바인딩을 사용하는 것이 좋다.

```java
@Query("select m from Member m where m.username = :name")
Member findMember(@Param("name") String username);
```

컬렉션 파라미터도 사용할 수 있다.

```java
@Query("select m from Member m where m.username in :names")
List<Member> findByNames(@Param("names") List<String> names);
```

---

## 8. 반환 타입

Spring Data JPA는 다양한 반환 타입을 지원한다.

```java
List<Member> findByUsername(String username);

Member findMemberByUsername(String username);

Optional<Member> findOptionalByUsername(String username);
```

반환 타입별 특징은 다음과 같다.

### 컬렉션

조회 결과가 없으면 빈 컬렉션을 반환한다.

### 단건

결과가 없으면 `null`을 반환할 수 있다.

결과가 둘 이상이면 예외가 발생한다.

### Optional

조회 결과가 없을 수 있음을 명시적으로 표현한다.

단건 조회에서는 `Optional`을 사용하는 방식이 더 명확하다.

---

## 9. Spring Data JPA 페이징과 정렬

Spring Data JPA는 `Pageable`과 `Sort`를 제공한다.

```java
Page<Member> findByAge(int age, Pageable pageable);
```

호출할 때는 `PageRequest`를 사용한다.

```java
PageRequest pageRequest = PageRequest.of(
        0,
        3,
        Sort.by(Sort.Direction.DESC, "username")
);

Page<Member> page = memberRepository.findByAge(10, pageRequest);
```

주의할 점은 페이지 번호가 0부터 시작한다는 것이다.

```text
0번 페이지 = 첫 번째 페이지
```

---

## 10. Page, Slice, List 차이

페이징 반환 타입은 화면 요구사항에 따라 선택해야 한다.

### Page

- 데이터 조회 쿼리 실행
- 전체 count 쿼리 실행
- 전체 페이지 수 제공
- 전체 데이터 수 제공
- 페이지 번호 기반 UI에 적합

### Slice

- count 쿼리를 실행하지 않는다.
- 요청한 개수보다 1개 더 조회해서 다음 페이지 여부를 확인한다.
- 무한 스크롤에 적합하다.

### List

- count 쿼리를 실행하지 않는다.
- 조회 결과만 필요할 때 사용한다.

정리하면 다음과 같다.

- 전체 페이지 수가 필요하면 `Page`
- 다음 페이지 여부만 필요하면 `Slice`
- 단순 목록만 필요하면 `List`

---

## 11. count 쿼리 분리

복잡한 조인이 포함된 페이징 쿼리는 count 쿼리가 무거울 수 있다.

이 경우 count 쿼리를 분리할 수 있다.

```java
@Query(
    value = "select m from Member m left join m.team t",
    countQuery = "select count(m) from Member m"
)
Page<Member> findMemberAllCountBy(Pageable pageable);
```

데이터 조회에는 조인이 필요하지만, count에는 조인이 필요 없는 경우가 있다.

이런 상황에서는 count 쿼리를 분리하면 성능 개선에 도움이 된다.

Spring Boot 3.x에서 사용하는 Hibernate 6은 실제로 select나 where에서 사용하지 않는 의미 없는 left join을 제거하는 최적화를 할 수 있다.

하지만 연관 엔티티를 함께 로딩하려면 단순 join이 아니라 fetch join이 필요하다.

---

## 12. Page.map으로 DTO 변환

`Page`는 `map()`을 제공한다.

엔티티 Page를 DTO Page로 쉽게 변환할 수 있다.

```java
Page<Member> page = memberRepository.findByAge(10, pageRequest);

Page<MemberDto> dtoPage = page.map(member ->
        new MemberDto(member.getId(), member.getUsername(), member.getTeam().getName())
);
```

이 방식은 페이징 메타데이터를 유지하면서 응답 DTO로 변환할 수 있어 API 응답에서 유용하다.

다만 DTO 변환 중 지연 로딩이 발생할 수 있으므로 필요한 연관관계는 페치 조인이나 EntityGraph로 미리 조회하는 것을 고려해야 한다.

---

## 13. 벌크성 수정 쿼리

벌크성 수정 쿼리는 여러 데이터를 한 번에 수정하는 쿼리이다.

```java
@Modifying
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```

주의할 점은 벌크 연산이 영속성 컨텍스트를 거치지 않고 DB에 직접 실행된다는 것이다.

이미 영속성 컨텍스트에 올라온 엔티티는 DB 값과 다를 수 있다.

따라서 벌크 쿼리 실행 후에는 영속성 컨텍스트를 비워야 한다.

```java
@Modifying(clearAutomatically = true)
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```

또는 직접 `em.clear()`를 호출할 수 있다.

---

## 14. @EntityGraph

`@EntityGraph`는 연관 엔티티를 함께 조회할 때 사용할 수 있다.

지연 로딩으로 인해 N+1 문제가 발생할 수 있는 경우, 필요한 연관관계를 한 번에 조회하도록 지정한다.

```java
@EntityGraph(attributePaths = {"team"})
List<Member> findByUsername(String username);
```

JPQL의 fetch join과 비슷한 효과를 낼 수 있다.

간단한 연관관계 로딩 최적화에는 `@EntityGraph`가 편리하다.

복잡한 조건이나 동적 쿼리는 Querydsl이나 직접 JPQL fetch join을 고려할 수 있다.

---

## 15. JPA Hint와 Lock

JPA Hint는 JPA 구현체에 부가 설정을 전달할 때 사용한다.

예를 들어 조회 전용 최적화를 위해 read-only 힌트를 줄 수 있다.

```java
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
Member findReadOnlyByUsername(String username);
```

동시성 제어가 필요한 경우 Lock을 사용할 수 있다.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findByUsername(String username);
```

락은 성능과 대기 시간에 영향을 줄 수 있으므로 꼭 필요한 곳에만 신중하게 사용해야 한다.

---

## 16. 보충 정리

쿼리 메소드 기능은 매우 편리하지만, 모든 조회 문제를 해결하는 도구는 아니다.

실무 기준으로는 다음처럼 선택하면 좋다.

- 단순 조건 조회: 메서드 이름 기반 쿼리
- 정적 JPQL, DTO 조회: `@Query`
- 동적 조건이 많은 검색: Querydsl
- 특정 DB 기능, 복잡한 성능 최적화: Native SQL 또는 JdbcTemplate
- N+1 완화: fetch join 또는 `@EntityGraph`
- 페이지 수 필요: `Page`
- 무한 스크롤: `Slice`
- 벌크 수정: `@Modifying` + 영속성 컨텍스트 초기화

---

## 17. 오늘 정리

Spring Data JPA의 쿼리 메소드 기능은 리포지토리 인터페이스만으로 다양한 조회를 가능하게 한다.

중요한 정리는 다음과 같다.

- 메서드 이름 기반 쿼리는 단순 조건에 적합하다.
- 복잡한 정적 쿼리는 `@Query`를 사용한다.
- 파라미터 바인딩은 이름 기반을 사용한다.
- DTO 조회는 JPQL `new` 명령어와 생성자가 필요하다.
- `Page`는 count 쿼리를 포함한다.
- `Slice`는 count 없이 다음 페이지 여부만 확인한다.
- count 쿼리는 성능상 분리할 수 있다.
- 벌크 수정 쿼리는 영속성 컨텍스트와 DB 상태 불일치에 주의한다.
- `@EntityGraph`는 연관 엔티티 조회 최적화에 유용하다.
- Hint와 Lock은 필요한 상황에서 신중하게 사용한다.

앞으로 조회 기능을 만들 때는 단순히 “쿼리가 동작하는가”만 보지 않고, 반환 타입, count 쿼리 비용, N+1 가능성, 영속성 컨텍스트 영향까지 함께 고려해야겠다.
