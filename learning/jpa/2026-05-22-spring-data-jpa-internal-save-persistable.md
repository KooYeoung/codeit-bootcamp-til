# Spring Data JPA 내부 분석: SimpleJpaRepository, 트랜잭션, save, Persistable 정리

## 학습 날짜

2026-05-22

## 학습 주제

- `SimpleJpaRepository`
- `@Repository`
- `@Transactional(readOnly = true)`
- 리포지토리 트랜잭션 전파
- `save()` 내부 동작
- `persist`
- `merge`
- 새로운 엔티티 판단 전략
- 직접 할당 ID 문제
- `Persistable`
- `@CreatedDate`를 활용한 `isNew()` 판단

---

## 1. Spring Data JPA 구현체

Spring Data JPA가 제공하는 공통 인터페이스의 기본 구현체는 `SimpleJpaRepository`이다.

리포지토리 인터페이스만 작성해도 동작하는 이유는 Spring Data JPA가 내부적으로 이 구현체와 프록시를 사용하기 때문이다.

구조를 단순화하면 다음과 같다.

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> {

    @Transactional
    public <S extends T> S save(S entity) {
        if (entityInformation.isNew(entity)) {
            em.persist(entity);
            return entity;
        } else {
            return em.merge(entity);
        }
    }
}
```

여기서 중요한 포인트는 세 가지이다.

- `@Repository`
- `@Transactional(readOnly = true)`
- `save()`의 `persist`와 `merge` 분기

---

## 2. @Repository의 역할

`SimpleJpaRepository`에는 `@Repository`가 적용되어 있다.

`@Repository`는 단순히 컴포넌트 스캔 대상이라는 의미만 있는 것이 아니다.

Spring은 데이터 접근 계층에서 발생한 예외를 스프링이 추상화한 데이터 접근 예외로 변환한다.

예를 들어 JPA 구현체나 JDBC에서 발생한 구체적인 예외를 Spring의 `DataAccessException` 계층으로 변환할 수 있다.

이 덕분에 서비스 계층은 특정 데이터 접근 기술의 예외에 덜 의존할 수 있다.

---

## 3. @Transactional(readOnly = true)

`SimpleJpaRepository` 클래스에는 기본적으로 읽기 전용 트랜잭션이 적용된다.

```java
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> {
}
```

조회 메서드는 기본적으로 readOnly 트랜잭션으로 동작한다.

읽기 전용 트랜잭션은 변경을 하지 않는다는 의도를 표현하고, JPA에서는 플러시를 생략하는 방식으로 약간의 성능 최적화를 얻을 수 있다.

다만 모든 상황에서 큰 성능 차이가 나는 것은 아니므로, 조회 로직의 의도를 명확히 하는 의미도 함께 가진다.

---

## 4. 변경 메서드의 트랜잭션

`save`, `delete` 같은 변경 메서드에는 별도의 `@Transactional`이 적용된다.

```java
@Transactional
public <S extends T> S save(S entity) {
}
```

JPA의 모든 변경 작업은 트랜잭션 안에서 실행되어야 한다.

서비스 계층에서 트랜잭션이 시작되지 않았다면 리포지토리 계층에서 트랜잭션이 시작된다.

서비스 계층에서 이미 트랜잭션이 시작되었다면 리포지토리는 그 트랜잭션에 참여한다.

즉, 트랜잭션 전파가 적용된다.

---

## 5. 트랜잭션은 서비스 계층에서 시작하는 것이 좋다

리포지토리에도 트랜잭션이 걸려 있기 때문에 단순 저장은 서비스 트랜잭션 없이도 동작할 수 있다.

하지만 실무에서는 보통 서비스 계층에 트랜잭션을 둔다.

이유는 하나의 비즈니스 작업이 여러 리포지토리 호출로 구성될 수 있기 때문이다.

예를 들어 주문 생성은 다음 작업을 포함할 수 있다.

- 회원 조회
- 상품 조회
- 재고 차감
- 주문 생성
- 배송 생성
- 주문 저장

이 작업들은 하나의 트랜잭션으로 묶여야 한다.

따라서 서비스 계층에서 트랜잭션 경계를 잡는 것이 자연스럽다.

---

## 6. save() 내부 동작

Spring Data JPA의 `save()`는 새로운 엔티티인지 여부를 판단한다.

```java
if (entityInformation.isNew(entity)) {
    em.persist(entity);
    return entity;
} else {
    return em.merge(entity);
}
```

새로운 엔티티면 `persist()`를 호출한다.

이미 존재하는 엔티티라고 판단되면 `merge()`를 호출한다.

따라서 `save()`를 단순히 INSERT 메서드로 이해하면 안 된다.

---

## 7. persist

`persist()`는 새로운 엔티티를 영속성 컨텍스트에 저장한다.

```java
em.persist(entity);
```

영속 상태가 된 엔티티는 JPA가 관리한다.

트랜잭션 커밋 시점에 INSERT SQL이 실행된다.

새로운 엔티티를 저장할 때는 `persist()`가 적절하다.

---

## 8. merge

`merge()`는 준영속 상태의 엔티티를 병합할 때 사용한다.

```java
Member mergedMember = em.merge(detachedMember);
```

`merge()`는 전달받은 객체를 그대로 영속 상태로 만드는 것이 아니다.

동작 흐름은 다음과 같이 이해할 수 있다.

1. 식별자로 영속성 컨텍스트를 조회한다.
2. 없으면 DB에서 조회한다.
3. 그래도 없으면 새로운 엔티티를 생성한다.
4. 전달받은 엔티티의 값을 영속 엔티티에 복사한다.
5. 영속 엔티티를 반환한다.

중요한 점은 `merge()`의 반환값이 영속 엔티티라는 것이다.

전달한 객체 자체가 영속 상태가 되는 것이 아니다.

---

## 9. 수정에는 merge보다 변경 감지

실무에서 엔티티 수정은 `merge()`보다 변경 감지를 사용하는 것이 좋다.

```java
@Transactional
public void updateMember(Long id, String username) {
    Member member = memberRepository.findById(id)
            .orElseThrow();

    member.setUsername(username);
}
```

영속 상태의 엔티티를 조회한 뒤 값을 변경하면, 트랜잭션 커밋 시점에 변경 감지가 동작한다.

변경 감지를 사용하면 수정 대상과 변경 값이 더 명확하다.

반면 `merge()`는 모든 필드를 복사하기 때문에 null 값이 의도치 않게 반영될 위험이 있다.

---

## 10. 새로운 엔티티 판단 기본 전략

Spring Data JPA는 새로운 엔티티인지 판단하기 위해 식별자 값을 확인한다.

기본 전략은 다음과 같다.

- 식별자가 객체 타입이면 `null`이면 새로운 엔티티
- 식별자가 자바 기본 타입이면 `0`이면 새로운 엔티티

예를 들어 `Long id`를 사용하고 `@GeneratedValue`를 사용하면 저장 전 ID가 null이다.

따라서 새로운 엔티티로 판단되어 `persist()`가 호출된다.

```java
@Id
@GeneratedValue
private Long id;
```

---

## 11. 직접 할당 ID 문제

문제는 ID를 직접 할당하는 경우이다.

```java
@Id
private String id;
```

엔티티를 생성할 때 이미 ID를 넣는다.

```java
Item item = new Item("A");
itemRepository.save(item);
```

Spring Data JPA는 ID 값이 null이 아니므로 새로운 엔티티가 아니라고 판단할 수 있다.

그 결과 `persist()`가 아니라 `merge()`가 호출된다.

`merge()`는 먼저 DB를 조회해서 해당 ID가 존재하는지 확인한다.

DB에 없으면 그때 새로운 엔티티로 판단해 저장한다.

즉, 불필요한 SELECT가 먼저 발생할 수 있다.

---

## 12. Persistable

직접 할당 ID를 사용하는 경우 `Persistable`을 구현해 새로운 엔티티 판단 로직을 직접 제공할 수 있다.

```java
public interface Persistable<ID> {

    ID getId();

    boolean isNew();
}
```

예제는 다음과 같다.

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable<String> {

    @Id
    private String id;

    @CreatedDate
    private LocalDateTime createdDate;

    public Item(String id) {
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }

    @Override
    public boolean isNew() {
        return createdDate == null;
    }
}
```

`createdDate`가 null이면 아직 저장되지 않은 새로운 엔티티로 판단한다.

---

## 13. @CreatedDate와 isNew

`@CreatedDate`는 엔티티가 저장될 때 값이 채워진다.

따라서 저장 전에는 null이다.

이 특성을 활용하면 `isNew()`를 다음처럼 구현할 수 있다.

```java
@Override
public boolean isNew() {
    return createdDate == null;
}
```

직접 할당 ID가 있어도 저장 전에는 `createdDate`가 null이므로 새로운 엔티티로 판단된다.

이렇게 하면 `save()` 호출 시 `merge()`가 아니라 `persist()`가 호출된다.

---

## 14. 보충: save를 언제 사용해야 할까

Spring Data JPA에서 `save()`는 저장과 병합을 모두 처리한다.

하지만 실무에서는 다음 기준을 추천한다.

### 신규 등록

새로운 엔티티를 등록할 때 `save()`를 사용한다.

```java
memberRepository.save(new Member("memberA"));
```

### 수정

수정은 `save()`보다 변경 감지를 사용한다.

```java
@Transactional
public void update(Long id, String username) {
    Member member = memberRepository.findById(id)
            .orElseThrow();
    member.changeUsername(username);
}
```

### 직접 할당 ID

직접 할당 ID를 사용한다면 `Persistable` 구현을 검토한다.

```java
public class Item implements Persistable<String> {
}
```

---

## 15. 보충: readOnly 트랜잭션과 성능

조회 전용 서비스에는 `@Transactional(readOnly = true)`를 사용할 수 있다.

```java
@Transactional(readOnly = true)
public List<Member> findMembers() {
    return memberRepository.findAll();
}
```

이 설정은 다음 의미를 가진다.

- 이 메서드는 조회 전용이라는 의도를 표현한다.
- JPA 플러시 관련 비용을 줄일 수 있다.
- 실수로 변경을 기대하는 로직을 넣지 않도록 코드 의도를 명확히 한다.

다만 readOnly라고 해서 모든 변경이 물리적으로 항상 차단되는 것은 아니다.

따라서 읽기 전용 메서드에서는 엔티티 변경 로직을 넣지 않는 습관이 중요하다.

---

## 16. 보충: 엔티티 생명주기와 save 이해

`save()`의 동작을 정확히 이해하려면 엔티티 생명주기를 함께 봐야 한다.

- 비영속: 아직 JPA가 관리하지 않는 새 객체
- 영속: 영속성 컨텍스트가 관리하는 객체
- 준영속: 과거에 영속이었지만 현재는 분리된 객체
- 삭제: 삭제 대상으로 관리되는 객체

`persist()`는 비영속 엔티티를 영속 상태로 만든다.

`merge()`는 준영속 엔티티의 값을 영속 엔티티에 복사한다.

따라서 신규 저장과 수정은 같은 `save()`로 보일 수 있지만, 내부 동작은 매우 다르다.

---

## 17. 오늘 정리

Spring Data JPA를 잘 사용하려면 내부 구현을 어느 정도 이해해야 한다.

특히 `save()`의 동작은 매우 중요하다.

중요한 정리는 다음과 같다.

- Spring Data JPA의 기본 구현체는 `SimpleJpaRepository`이다.
- 조회 메서드는 기본적으로 `@Transactional(readOnly = true)`가 적용된다.
- 변경 메서드는 별도 트랜잭션으로 동작한다.
- 서비스 계층 트랜잭션이 있으면 리포지토리는 해당 트랜잭션에 참여한다.
- `save()`는 새로운 엔티티면 `persist`, 기존 엔티티면 `merge`를 호출한다.
- 수정은 `merge`보다 변경 감지를 사용하는 것이 좋다.
- 직접 할당 ID는 새로운 엔티티 판단이 어려울 수 있다.
- 직접 할당 ID에서는 `Persistable` 구현을 고려한다.
- `@CreatedDate`를 활용하면 `isNew()`를 편리하게 구현할 수 있다.

앞으로 Spring Data JPA를 사용할 때는 `save()`를 단순 저장 메서드로만 생각하지 말고, 내부에서 `persist`와 `merge` 중 무엇이 호출되는지 확인하며 사용해야겠다.
