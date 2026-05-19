# Spring 트랜잭션 이해와 전파 정리

## 학습 날짜

2026-05-18

## 학습 주제

- 스프링 트랜잭션 추상화
- 선언적 트랜잭션 관리
- 프로그래밍 방식 트랜잭션 관리
- 트랜잭션 AOP
- `@Transactional`
- 트랜잭션 적용 위치
- 트랜잭션 옵션
- 예외와 커밋, 롤백
- 트랜잭션 전파
- REQUIRED
- REQUIRES_NEW
- rollback-only
- 트랜잭션 전파 활용

---

## 1. 스프링 트랜잭션을 다시 학습하는 이유

트랜잭션은 데이터 정합성을 지키기 위한 핵심 기능이다.

계좌이체처럼 여러 작업이 하나의 단위로 처리되어야 하는 경우, 모든 작업이 성공하면 커밋하고 하나라도 실패하면 롤백해야 한다.

스프링은 트랜잭션을 더 편리하고 일관되게 사용할 수 있도록 다양한 기능을 제공한다.

특히 `@Transactional`을 사용하면 트랜잭션 시작, 커밋, 롤백 코드를 직접 작성하지 않아도 된다.

하지만 내부 동작을 이해하지 못하면 예상과 다르게 커밋되거나 롤백되는 상황을 만날 수 있다.

따라서 트랜잭션 AOP, 적용 위치, 예외 규칙, 전파 옵션을 함께 이해해야 한다.

---

## 2. 트랜잭션 추상화

데이터 접근 기술마다 트랜잭션을 다루는 방법이 다르다.

JDBC에서는 다음처럼 커넥션을 직접 다룬다.

```java
con.setAutoCommit(false);
con.commit();
con.rollback();
```

JPA에서는 `EntityTransaction`이나 `JpaTransactionManager`를 통해 트랜잭션을 다룬다.

기술마다 트랜잭션 사용법이 다르면 서비스 계층이 특정 기술에 의존하게 된다.

스프링은 이 문제를 해결하기 위해 `PlatformTransactionManager`를 제공한다.

```java
public interface PlatformTransactionManager {
    TransactionStatus getTransaction(TransactionDefinition definition);
    void commit(TransactionStatus status);
    void rollback(TransactionStatus status);
}
```

트랜잭션은 결국 시작, 커밋, 롤백으로 추상화할 수 있다.

스프링은 JDBC를 사용하면 `DataSourceTransactionManager` 또는 `JdbcTransactionManager`, JPA를 사용하면 `JpaTransactionManager`를 사용한다.

스프링 부트는 사용하는 데이터 접근 기술을 보고 적절한 트랜잭션 매니저를 자동으로 등록해줄 수 있다.

---

## 3. 프로그래밍 방식 트랜잭션 관리

프로그래밍 방식 트랜잭션 관리는 개발자가 코드로 직접 트랜잭션을 시작하고 커밋, 롤백하는 방식이다.

```java
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

try {
    bizLogic();
    transactionManager.commit(status);
} catch (Exception e) {
    transactionManager.rollback(status);
    throw e;
}
```

이 방식은 동작이 명확하지만, 서비스 코드에 트랜잭션 처리 코드가 섞인다.

비즈니스 로직보다 트랜잭션 기술 코드가 더 많이 보일 수 있다.

그래서 실무에서는 대부분 선언적 트랜잭션 관리를 사용한다.

---

## 4. 선언적 트랜잭션 관리

선언적 트랜잭션 관리는 `@Transactional` 애노테이션을 선언해서 트랜잭션을 적용하는 방식이다.

```java
@Transactional
public void order() {
    bizLogic();
}
```

개발자는 비즈니스 메서드에 트랜잭션이 필요하다고 선언만 한다.

트랜잭션 시작, 커밋, 롤백은 스프링이 처리한다.

이 방식은 코드가 간결하고, 비즈니스 로직과 트랜잭션 처리 로직을 분리할 수 있다.

---

## 5. 트랜잭션 AOP

`@Transactional`은 기본적으로 프록시 기반 AOP로 동작한다.

흐름은 다음과 같다.

1. 클라이언트가 서비스 메서드를 호출한다.
2. 실제 서비스 객체가 아니라 트랜잭션 프록시가 먼저 호출된다.
3. 프록시가 트랜잭션을 시작한다.
4. 프록시가 실제 서비스 메서드를 호출한다.
5. 정상 종료되면 커밋한다.
6. 예외가 발생하면 롤백 여부를 판단한다.
7. 트랜잭션을 종료한다.

중요한 점은 프록시를 통해 호출되어야 트랜잭션이 적용된다는 것이다.

같은 클래스 내부에서 자기 자신의 `@Transactional` 메서드를 직접 호출하면 프록시를 거치지 않아 트랜잭션이 적용되지 않을 수 있다.

---

## 6. 트랜잭션 적용 위치

`@Transactional`은 클래스나 메서드에 붙일 수 있다.

```java
@Transactional
public class OrderService {
}
```

```java
@Transactional
public void order() {
}
```

보통은 서비스 계층의 비즈니스 작업 단위에 적용한다.

트랜잭션은 단순 DB 저장 메서드 하나보다, 비즈니스적으로 하나의 작업 단위가 되는 메서드에 적용하는 것이 자연스럽다.

예를 들어 회원 가입과 가입 로그 저장을 하나의 비즈니스 작업으로 본다면 서비스 메서드에 트랜잭션을 적용할 수 있다.

---

## 7. 트랜잭션 옵션

`@Transactional`에는 여러 옵션이 있다.

대표적으로 다음이 있다.

- `value`, `transactionManager`
- `rollbackFor`
- `noRollbackFor`
- `propagation`
- `isolation`
- `timeout`
- `readOnly`

초기 학습 단계에서는 다음을 우선 이해하면 좋다.

- 롤백 규칙: 어떤 예외에서 롤백할 것인가
- 전파 옵션: 기존 트랜잭션이 있을 때 어떻게 동작할 것인가
- 읽기 전용 옵션: 조회 전용 트랜잭션인지

---

## 8. 예외와 트랜잭션 롤백 규칙

스프링 트랜잭션의 기본 롤백 규칙은 다음과 같다.

- 런타임 예외 발생: 롤백
- 체크 예외 발생: 커밋

런타임 예외 예시는 다음과 같다.

```java
@Transactional
public void runtimeException() {
    throw new RuntimeException();
}
```

이 경우 트랜잭션은 롤백된다.

체크 예외 예시는 다음과 같다.

```java
@Transactional
public void checkedException() throws Exception {
    throw new Exception();
}
```

이 경우 기본 정책으로는 커밋된다.

체크 예외도 롤백하고 싶다면 `rollbackFor`를 사용한다.

```java
@Transactional(rollbackFor = Exception.class)
public void checkedException() throws Exception {
    throw new Exception();
}
```

---

## 9. 트랜잭션 전파란

트랜잭션 전파는 이미 트랜잭션이 진행 중일 때, 새로 호출되는 메서드가 어떤 트랜잭션을 사용할지 결정하는 규칙이다.

예를 들어 서비스 메서드에 트랜잭션이 있고, 그 안에서 리포지토리 메서드를 호출한다고 하자.

이때 리포지토리 메서드도 `@Transactional`이라면 다음을 결정해야 한다.

- 기존 트랜잭션에 참여할 것인가?
- 새로운 트랜잭션을 만들 것인가?
- 트랜잭션 없이 실행할 것인가?

이 규칙이 전파 옵션이다.

---

## 10. 물리 트랜잭션과 논리 트랜잭션

트랜잭션 전파를 이해할 때 물리 트랜잭션과 논리 트랜잭션을 구분해야 한다.

## 물리 트랜잭션

실제 DB 커넥션을 통해 시작되는 진짜 트랜잭션이다.

커밋과 롤백도 실제 DB 커넥션에서 일어난다.

## 논리 트랜잭션

스프링이 트랜잭션 전파를 관리하기 위해 사용하는 논리적인 트랜잭션 범위이다.

여러 `@Transactional` 메서드가 하나의 물리 트랜잭션에 참여할 수 있다.

이때 각각의 메서드는 논리 트랜잭션처럼 보이지만, 실제 커밋은 가장 바깥쪽 물리 트랜잭션에서 일어난다.

---

## 11. REQUIRED

`REQUIRED`는 기본 전파 옵션이다.

기존 트랜잭션이 없으면 새 트랜잭션을 만든다.

기존 트랜잭션이 있으면 기존 트랜잭션에 참여한다.

```java
@Transactional(propagation = Propagation.REQUIRED)
public void save() {
}
```

대부분의 비즈니스 로직에서는 `REQUIRED`를 사용한다.

예를 들어 회원 저장과 로그 저장이 같은 트랜잭션으로 묶이면, 둘 중 하나가 실패했을 때 전체를 함께 롤백할 수 있다.

---

## 12. REQUIRED에서 내부 트랜잭션 롤백

외부 트랜잭션이 있고 내부 트랜잭션이 `REQUIRED`로 참여한다고 하자.

내부 트랜잭션에서 예외가 발생해 롤백되면, 실제 물리 트랜잭션을 바로 롤백하는 것이 아니라 rollback-only 표시가 될 수 있다.

이후 외부 트랜잭션이 정상적으로 커밋하려고 해도, 이미 rollback-only가 설정되어 있기 때문에 최종적으로 롤백된다.

이 흐름은 처음에는 헷갈릴 수 있다.

핵심은 하나의 물리 트랜잭션에 참여한 논리 트랜잭션 중 하나라도 롤백되어야 한다고 표시되면, 전체 물리 트랜잭션은 커밋할 수 없다는 점이다.

---

## 13. REQUIRES_NEW

`REQUIRES_NEW`는 항상 새로운 트랜잭션을 만든다.

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveLog() {
}
```

기존 트랜잭션이 있어도 새 물리 트랜잭션을 생성한다.

따라서 기존 트랜잭션과 분리되어 커밋, 롤백된다.

예를 들어 회원 저장은 성공해야 하지만 로그 저장은 실패해도 회원 저장은 유지되어야 하는 요구사항이 있을 수 있다.

이때 로그 저장을 `REQUIRES_NEW`로 분리하면 로그 트랜잭션만 롤백하고, 회원 저장 트랜잭션은 커밋할 수 있다.

---

## 14. REQUIRES_NEW 주의점

`REQUIRES_NEW`는 물리 트랜잭션을 분리한다.

따라서 DB 커넥션도 별도로 사용될 수 있다.

외부 트랜잭션이 커넥션을 가지고 있는 상태에서 내부 `REQUIRES_NEW`가 새로운 커넥션을 추가로 사용할 수 있다.

즉, 동시에 커넥션이 2개 필요할 수 있다.

커넥션 풀이 작거나 많은 요청이 동시에 들어오는 환경에서는 주의해야 한다.

`REQUIRES_NEW`는 유용하지만 남용하면 커넥션 사용량과 트랜잭션 구조가 복잡해질 수 있다.

---

## 15. 트랜잭션 전파 활용 예제

예제 요구사항은 다음과 같다.

- 회원을 등록한다.
- 회원 변경 이력을 DB 로그 테이블에 남긴다.
- 예제를 단순화하기 위해 회원 등록 시점에 로그를 남긴다.

구성은 다음과 같다.

- `Member`: 회원 엔티티
- `Log`: 로그 엔티티
- `MemberRepository`: 회원 저장
- `LogRepository`: 로그 저장
- `MemberService`: 회원 저장과 로그 저장을 함께 수행

로그 저장에서 예외가 발생하는 상황을 만들어 트랜잭션 전파를 확인한다.

---

## 16. 로그 실패 시 전체 롤백이 필요한 경우

회원 저장과 로그 저장이 반드시 함께 성공해야 한다면 같은 트랜잭션으로 묶으면 된다.

이 경우 로그 저장 실패 시 회원 저장도 함께 롤백된다.

기본 `REQUIRED` 전파를 사용하면 기존 트랜잭션에 참여한다.

이 방식은 데이터 정합성을 강하게 유지할 수 있다.

하지만 로그는 부가 기능이고, 로그 실패 때문에 회원 가입 자체가 실패하면 안 되는 요구사항이라면 다른 방식이 필요하다.

---

## 17. 로그 실패를 복구하고 회원 저장은 유지해야 하는 경우

로그 저장 실패가 회원 가입 실패로 이어지면 안 된다면 로그 트랜잭션을 분리할 수 있다.

`LogRepository`에 `REQUIRES_NEW`를 적용한다.

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void save(Log logMessage) {
    em.persist(logMessage);
}
```

서비스에서는 로그 저장 예외를 잡고 정상 흐름으로 변환한다.

```java
try {
    logRepository.save(logMessage);
} catch (RuntimeException e) {
    log.info("로그 저장 실패");
}
```

이렇게 하면 로그 저장 트랜잭션만 롤백되고, 회원 저장 트랜잭션은 커밋될 수 있다.

---

## 18. 다양한 전파 옵션

스프링은 여러 전파 옵션을 제공한다.

대표적으로 다음이 있다.

- `REQUIRED`
- `REQUIRES_NEW`
- `SUPPORTS`
- `NOT_SUPPORTED`
- `MANDATORY`
- `NEVER`
- `NESTED`

실무에서는 대부분 `REQUIRED`를 사용하고, 아주 가끔 `REQUIRES_NEW`를 사용한다.

나머지 옵션은 이런 것이 있다는 정도로 알아두고 필요할 때 찾아보면 된다.

---

## 19. 트랜잭션 설계 기준

트랜잭션을 설계할 때는 다음 질문을 해보면 좋다.

- 어디까지가 하나의 비즈니스 작업인가?
- 어떤 작업은 반드시 함께 성공해야 하는가?
- 어떤 작업은 실패해도 전체 비즈니스는 성공해야 하는가?
- 예외가 발생했을 때 롤백해야 하는가, 커밋해야 하는가?
- 외부 트랜잭션과 내부 트랜잭션을 분리해야 하는가?
- 커넥션 사용량이 늘어나는 것을 감당할 수 있는가?

트랜잭션은 단순히 애노테이션을 붙이는 문제가 아니라, 비즈니스 요구사항과 데이터 정합성을 함께 고려해야 한다.

---

## 20. 오늘 정리

스프링 트랜잭션은 서비스 계층에서 비즈니스 작업 단위를 안전하게 처리하기 위한 핵심 기능이다.

`@Transactional`을 사용하면 트랜잭션 처리를 선언적으로 적용할 수 있고, 스프링 AOP가 실제 트랜잭션 시작과 종료를 담당한다.

중요한 정리는 다음과 같다.

- 스프링은 `PlatformTransactionManager`로 트랜잭션을 추상화한다.
- 실무에서는 대부분 `@Transactional`을 사용하는 선언적 트랜잭션 관리를 사용한다.
- `@Transactional`은 프록시 기반 AOP로 동작한다.
- 프록시 내부 호출은 트랜잭션이 적용되지 않을 수 있으므로 주의해야 한다.
- 기본적으로 런타임 예외는 롤백, 체크 예외는 커밋이다.
- 체크 예외도 롤백하려면 `rollbackFor`를 사용한다.
- 기본 전파 옵션은 `REQUIRED`이다.
- `REQUIRED`는 기존 트랜잭션이 있으면 참여하고, 없으면 새로 만든다.
- `REQUIRES_NEW`는 항상 새 트랜잭션을 만든다.
- `REQUIRES_NEW`는 물리 트랜잭션이 분리되므로 커넥션 사용량을 주의해야 한다.
- 일부 작업 실패를 전체 실패로 볼지, 독립적으로 복구할지는 비즈니스 요구사항에 따라 결정해야 한다.

앞으로 트랜잭션을 사용할 때는 단순히 `@Transactional`을 붙이는 것에서 끝내지 않고, 적용 위치와 전파 옵션, 예외 롤백 규칙을 함께 고려해야겠다.
