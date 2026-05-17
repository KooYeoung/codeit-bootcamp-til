# Spring DB 스프링 트랜잭션, 예외 처리, JdbcTemplate 정리

## 학습 날짜

2026-05-17

## 학습 주제

- 애플리케이션 계층 구조
- 순수한 서비스 계층
- 트랜잭션 추상화
- 트랜잭션 동기화
- 트랜잭션 매니저
- 트랜잭션 템플릿
- 트랜잭션 AOP
- `@Transactional`
- 자바 예외 계층
- 체크 예외와 언체크 예외
- 예외 전환
- 스프링 데이터 접근 예외 추상화
- JdbcTemplate

---

## 1. 애플리케이션 계층 구조

일반적인 애플리케이션은 역할에 따라 계층을 나눈다.

대표적으로 다음 3가지 계층으로 나눌 수 있다.

- 프레젠테이션 계층
- 서비스 계층
- 데이터 접근 계층

## 프레젠테이션 계층

프레젠테이션 계층은 웹 요청과 응답, 사용자 입력 검증, 화면 또는 API 응답 처리와 관련된 역할을 담당한다.

주로 스프링 MVC, 서블릿, HTTP 같은 웹 기술을 사용한다.

## 서비스 계층

서비스 계층은 핵심 비즈니스 로직을 담당한다.

가급적 특정 기술에 의존하지 않고 순수한 Java 코드로 작성하는 것이 좋다.

## 데이터 접근 계층

데이터 접근 계층은 실제 데이터베이스에 접근하는 코드를 담당한다.

JDBC, JPA, MyBatis, Redis 같은 기술이 이 계층에 위치한다.

계층을 나누는 중요한 이유는 서비스 계층을 최대한 순수하게 유지하기 위해서이다.

---

## 2. 서비스 계층을 순수하게 유지해야 하는 이유

서비스 계층에는 핵심 비즈니스 로직이 들어간다.

시간이 지나면서 UI 기술이나 데이터 접근 기술은 바뀔 수 있다.

예를 들어 다음과 같은 변경이 있을 수 있다.

- HTML 화면에서 HTTP API 방식으로 변경
- JDBC에서 JPA로 변경
- DB에서 외부 API 연동으로 변경

이때 서비스 계층이 특정 기술에 강하게 의존하면 변경 범위가 커진다.

서비스 계층을 순수하게 유지하면 기술 변경이 발생해도 비즈니스 로직은 크게 바꾸지 않아도 된다.

따라서 서비스 계층은 비즈니스 로직에 집중하고, 기술적인 코드는 프레젠테이션 계층이나 데이터 접근 계층으로 분리하는 것이 좋다.

---

## 3. 순수 JDBC 트랜잭션 적용 시 문제

JDBC로 트랜잭션을 직접 적용하면 서비스 계층에 JDBC 기술이 들어온다.

예를 들어 다음과 같은 코드가 필요하다.

```java
Connection con = dataSource.getConnection();

try {
    con.setAutoCommit(false);

    bizLogic(con, fromId, toId, money);

    con.commit();
} catch (Exception e) {
    con.rollback();
    throw new IllegalStateException(e);
} finally {
    release(con);
}
```

이 코드는 트랜잭션을 처리하지만, 서비스 계층에 다음 기술이 섞인다.

- `DataSource`
- `Connection`
- `SQLException`
- `setAutoCommit(false)`
- `commit()`
- `rollback()`

문제는 서비스 계층이 더 이상 순수한 비즈니스 로직만 가지지 않는다는 점이다.

또한 JDBC에서 JPA로 기술을 바꾸면 트랜잭션 처리 코드도 바뀌어야 한다.

---

## 4. 트랜잭션 관련 문제 정리

직접 JDBC 트랜잭션을 적용하면 다음 문제가 생긴다.

## JDBC 구현 기술 누수

서비스 계층이 JDBC의 `Connection`, `SQLException` 등에 의존한다.

서비스 계층은 특정 구현 기술을 몰라야 하는데, 트랜잭션 때문에 JDBC 기술이 새어 들어온다.

## 트랜잭션 동기화 문제

같은 트랜잭션을 유지하려면 같은 커넥션을 사용해야 한다.

이를 위해 커넥션을 파라미터로 계속 넘기면 코드가 지저분해진다.

또한 트랜잭션용 메서드와 트랜잭션이 필요 없는 메서드가 분리되는 문제가 생긴다.

## 트랜잭션 반복 코드

트랜잭션 시작, 커밋, 롤백, 리소스 정리 코드가 반복된다.

## 예외 누수

데이터 접근 계층의 `SQLException`이 서비스 계층으로 전파된다.

`SQLException`은 JDBC 전용 예외이므로 서비스 계층이 JDBC 기술에 의존하게 된다.

## JDBC 반복 문제

커넥션 획득, SQL 실행, 결과 매핑, 리소스 정리 코드가 반복된다.

---

## 5. 트랜잭션 추상화

스프링은 트랜잭션 기술을 추상화하기 위해 `PlatformTransactionManager`를 제공한다.

트랜잭션을 사용하는 방법은 기술마다 다르다.

- JDBC: `con.setAutoCommit(false)`
- JPA: `transaction.begin()`

만약 서비스 계층이 각 기술별 트랜잭션 코드에 직접 의존하면, 기술 변경 시 서비스 코드도 수정해야 한다.

스프링은 `PlatformTransactionManager`라는 공통 인터페이스로 이 문제를 해결한다.

```java
public interface PlatformTransactionManager {
    TransactionStatus getTransaction(TransactionDefinition definition);
    void commit(TransactionStatus status);
    void rollback(TransactionStatus status);
}
```

서비스 계층은 구체적인 JDBC나 JPA 트랜잭션 방법을 몰라도 된다.

필요한 구현체만 주입하면 된다.

- JDBC: `DataSourceTransactionManager`
- JPA: `JpaTransactionManager`

---

## 6. 트랜잭션 매니저

트랜잭션 매니저를 사용하면 트랜잭션 시작, 커밋, 롤백을 추상화된 방식으로 처리할 수 있다.

```java
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

try {
    bizLogic(fromId, toId, money);
    transactionManager.commit(status);
} catch (Exception e) {
    transactionManager.rollback(status);
    throw new IllegalStateException(e);
}
```

이 코드는 JDBC 트랜잭션 API에 직접 의존하지 않는다.

`transactionManager`의 구현체가 JDBC용이면 JDBC 트랜잭션을 처리하고, JPA용이면 JPA 트랜잭션을 처리한다.

서비스 계층은 `PlatformTransactionManager`에 의존하므로 기술 변경에 더 유연해진다.

---

## 7. 트랜잭션 동기화

트랜잭션을 시작하면 같은 트랜잭션 안에서는 같은 커넥션을 사용해야 한다.

스프링은 트랜잭션 동기화 매니저를 통해 이 문제를 해결한다.

동작 흐름은 다음과 같다.

1. 트랜잭션 매니저가 데이터소스에서 커넥션을 얻는다.
2. 커넥션을 수동 커밋 모드로 변경한다.
3. 트랜잭션 동기화 매니저에 커넥션을 보관한다.
4. 리포지토리는 `DataSourceUtils.getConnection()`을 통해 현재 트랜잭션에 연결된 커넥션을 가져온다.
5. 트랜잭션이 끝나면 커밋 또는 롤백 후 커넥션을 정리한다.

트랜잭션 동기화 매니저는 쓰레드 로컬을 사용한다.

따라서 멀티 쓰레드 환경에서도 각 요청의 트랜잭션 커넥션을 안전하게 관리할 수 있다.

---

## 8. DataSourceUtils

트랜잭션 동기화를 사용하려면 리포지토리에서 커넥션을 얻고 반납할 때 주의해야 한다.

일반적으로는 `dataSource.getConnection()`을 사용하지만, 트랜잭션 동기화를 위해서는 `DataSourceUtils.getConnection(dataSource)`를 사용한다.

```java
Connection con = DataSourceUtils.getConnection(dataSource);
```

커넥션을 닫을 때도 직접 `con.close()`를 호출하면 안 된다.

```java
DataSourceUtils.releaseConnection(con, dataSource);
```

트랜잭션이 진행 중인 커넥션을 직접 닫아버리면 같은 트랜잭션을 유지할 수 없다.

`DataSourceUtils.releaseConnection()`은 트랜잭션에 묶인 커넥션이면 닫지 않고 유지하고, 트랜잭션과 무관한 커넥션이면 닫아준다.

---

## 9. 트랜잭션 템플릿

트랜잭션 매니저를 사용해도 여전히 반복 코드가 남는다.

반복되는 구조는 다음과 같다.

```java
try {
    비즈니스 로직
    commit
} catch {
    rollback
}
```

스프링은 이런 반복을 줄이기 위해 `TransactionTemplate`을 제공한다.

```java
transactionTemplate.executeWithoutResult(status -> {
    bizLogic(fromId, toId, money);
});
```

트랜잭션 템플릿은 트랜잭션 시작, 커밋, 롤백 처리를 대신해준다.

하지만 서비스 로직 안에 트랜잭션 템플릿 코드가 남아있기 때문에, 완전히 순수한 서비스 계층이라고 보기는 어렵다.

이 문제는 트랜잭션 AOP로 더 깔끔하게 해결할 수 있다.

---

## 10. 트랜잭션 AOP와 @Transactional

스프링은 `@Transactional`을 사용해 트랜잭션을 선언적으로 적용할 수 있다.

```java
@Transactional
public void accountTransfer(String fromId, String toId, int money) {
    bizLogic(fromId, toId, money);
}
```

이렇게 작성하면 개발자가 직접 트랜잭션 시작, 커밋, 롤백 코드를 작성하지 않아도 된다.

스프링은 AOP를 사용해 프록시 객체를 만들고, 메서드 호출 전후에 트랜잭션을 처리한다.

흐름은 다음과 같다.

1. 클라이언트가 서비스 메서드를 호출한다.
2. 실제 서비스 객체가 아니라 트랜잭션 프록시가 먼저 호출된다.
3. 프록시가 트랜잭션을 시작한다.
4. 실제 서비스 메서드를 호출한다.
5. 정상 종료되면 커밋한다.
6. 예외가 발생하면 롤백한다.

덕분에 서비스 계층은 비즈니스 로직만 남길 수 있다.

---

## 11. 스프링 부트의 자동 리소스 등록

스프링 부트는 데이터베이스 접근에 필요한 리소스를 자동으로 등록해준다.

예를 들어 `DataSource` 설정 정보가 있으면 스프링 부트가 `DataSource`를 자동으로 스프링 빈으로 등록한다.

또한 JDBC를 사용하면 `DataSourceTransactionManager`도 자동으로 등록할 수 있다.

덕분에 개발자는 필요한 의존성을 주입받아 사용하면 된다.

스프링 부트 자동 설정을 이해하면 설정 코드를 줄이고 핵심 로직에 집중할 수 있다.

---

## 12. 자바 예외 계층

자바 예외의 최상위는 `Throwable`이다.

`Throwable` 아래에는 크게 `Error`와 `Exception`이 있다.

```text
Object
└── Throwable
    ├── Error
    └── Exception
        └── RuntimeException
```

## Error

`Error`는 메모리 부족 같은 복구하기 어려운 시스템 오류이다.

애플리케이션 로직에서 잡으려고 하면 안 된다.

## Exception

`Exception`과 그 하위 예외는 기본적으로 체크 예외이다.

단, `RuntimeException`은 예외이다.

## RuntimeException

`RuntimeException`과 그 하위 예외는 언체크 예외이다.

컴파일러가 예외 처리를 강제하지 않는다.

---

## 13. 예외 기본 규칙

예외의 기본 규칙은 두 가지이다.

1. 예외는 잡아서 처리하거나 밖으로 던져야 한다.
2. 예외를 잡거나 던질 때 지정한 예외뿐 아니라 그 자식 예외도 함께 처리된다.

예를 들어 `Exception`을 catch하면 그 하위 예외도 함께 잡힌다.

하지만 너무 상위 타입을 무분별하게 잡으면 의도하지 않은 예외까지 처리할 수 있으므로 주의해야 한다.

애플리케이션에서는 보통 `Throwable`을 직접 잡지 않는다.

`Throwable`을 잡으면 `Error`까지 잡을 수 있기 때문이다.

---

## 14. 체크 예외

체크 예외는 컴파일러가 처리 여부를 확인하는 예외이다.

`Exception`을 상속받고 `RuntimeException`을 상속받지 않으면 체크 예외가 된다.

```java
class MyCheckedException extends Exception {
    public MyCheckedException(String message) {
        super(message);
    }
}
```

체크 예외는 반드시 잡거나 던져야 한다.

```java
public void callThrow() throws MyCheckedException {
    repository.call();
}
```

체크 예외는 개발자가 실수로 예외 처리를 놓치지 않게 도와준다.

하지만 처리할 수 없는 예외까지 계속 throws로 전파하면 계층 전체가 특정 예외에 의존하게 된다.

---

## 15. 언체크 예외

언체크 예외는 컴파일러가 처리 여부를 강제하지 않는 예외이다.

`RuntimeException`을 상속받으면 언체크 예외가 된다.

```java
class MyUncheckedException extends RuntimeException {
    public MyUncheckedException(String message) {
        super(message);
    }
}
```

언체크 예외는 잡지 않아도 되고, throws를 선언하지 않아도 된다.

처리할 수 없는 예외를 억지로 각 계층에서 throws로 선언하지 않아도 되므로, 서비스 계층을 특정 기술 예외로부터 보호하는 데 도움이 된다.

---

## 16. 체크 예외와 언체크 예외 사용 기준

기본적으로는 언체크 예외를 사용하는 것이 좋다.

체크 예외는 비즈니스 로직상 의도적으로 발생시키고, 호출자가 반드시 처리해야 하는 경우에 사용하는 것이 좋다.

체크 예외를 고려할 수 있는 예시는 다음과 같다.

- 계좌 이체 실패
- 결제 포인트 부족
- 로그인 ID, 비밀번호 불일치

반대로 데이터베이스 연결 실패, 네트워크 장애처럼 대부분 복구하기 어려운 시스템 예외는 언체크 예외로 전환하고 공통 처리하는 것이 더 적절할 수 있다.

---

## 17. 예외 전환과 스택 트레이스

체크 예외를 언체크 예외로 바꿀 때는 기존 예외를 포함해야 한다.

```java
try {
    runSql();
} catch (SQLException e) {
    throw new MyDbException(e);
}
```

기존 예외를 포함하지 않으면 실제 원인을 추적하기 어렵다.

예외를 포함하면 스택 트레이스에서 최초 원인 예외까지 확인할 수 있다.

실무에서 예외 전환을 할 때는 원인 예외를 꼭 함께 넘기는 습관이 중요하다.

---

## 18. 체크 예외와 인터페이스 문제

서비스 계층은 구현체가 아니라 인터페이스에 의존하는 것이 좋다.

```java
public interface MemberRepository {
    Member save(Member member);
    Member findById(String memberId);
    void update(String memberId, int money);
    void delete(String memberId);
}
```

하지만 리포지토리가 `SQLException` 같은 체크 예외를 던지면 인터페이스에도 `throws SQLException`을 선언해야 한다.

```java
public interface MemberRepositoryEx {
    Member save(Member member) throws SQLException;
}
```

이렇게 되면 인터페이스가 JDBC 기술에 의존하게 된다.

나중에 JPA로 변경해도 인터페이스에 JDBC 예외가 남아있게 된다.

이 문제를 해결하려면 데이터 접근 계층에서 체크 예외를 런타임 예외로 전환하는 방식이 필요하다.

---

## 19. 스프링 데이터 접근 예외 추상화

데이터 접근 기술마다 예외가 다르다.

- JDBC는 `SQLException`
- JPA는 다른 예외 체계 사용
- MyBatis도 별도 예외 사용 가능

이 예외들이 그대로 서비스 계층으로 올라오면 서비스 계층이 데이터 접근 기술에 의존하게 된다.

스프링은 데이터 접근 예외를 추상화해서 `DataAccessException` 계층으로 제공한다.

이 예외는 런타임 예외이다.

스프링 예외 추상화를 사용하면 데이터 접근 기술이 바뀌어도 서비스 계층은 공통 예외 체계로 다룰 수 있다.

---

## 20. JdbcTemplate

JDBC를 직접 사용하면 반복 코드가 많다.

반복되는 코드는 다음과 같다.

- 커넥션 획득
- `PreparedStatement` 생성
- SQL 실행
- `ResultSet` 반복
- 객체 매핑
- 예외 처리
- 리소스 정리

스프링은 이 반복을 줄이기 위해 `JdbcTemplate`을 제공한다.

```java
public class MemberRepositoryV5 implements MemberRepository {

    private final JdbcTemplate template;

    public MemberRepositoryV5(DataSource dataSource) {
        this.template = new JdbcTemplate(dataSource);
    }

    public Member save(Member member) {
        String sql = "insert into member(member_id, money) values(?, ?)";
        template.update(sql, member.getMemberId(), member.getMoney());
        return member;
    }
}
```

`JdbcTemplate`을 사용하면 SQL과 파라미터, 매핑 로직에 집중할 수 있다.

---

## 21. JdbcTemplate 조회

단건 조회는 `queryForObject()`를 사용할 수 있다.

```java
public Member findById(String memberId) {
    String sql = "select * from member where member_id = ?";
    return template.queryForObject(sql, memberRowMapper(), memberId);
}
```

조회 결과를 객체로 바꾸기 위해 `RowMapper`를 사용한다.

```java
private RowMapper<Member> memberRowMapper() {
    return (rs, rowNum) -> {
        Member member = new Member();
        member.setMemberId(rs.getString("member_id"));
        member.setMoney(rs.getInt("money"));
        return member;
    };
}
```

`RowMapper`는 `ResultSet`의 한 행을 도메인 객체로 변환하는 역할을 한다.

---

## 22. JdbcTemplate의 장점

`JdbcTemplate`은 JDBC 반복 문제를 대부분 해결해준다.

장점은 다음과 같다.

- 커넥션 획득과 종료를 처리한다.
- `PreparedStatement` 생성과 실행을 처리한다.
- ResultSet 반복을 도와준다.
- 트랜잭션 동기화를 지원한다.
- 예외 발생 시 스프링 예외 변환기를 실행한다.
- 개발자는 SQL과 파라미터, 매핑 로직에 집중할 수 있다.

즉, JDBC 원리를 몰라도 된다는 뜻이 아니라, JDBC의 반복 작업을 스프링이 대신 처리해준다는 의미이다.

---

## 23. 오늘 정리

오늘 학습한 내용의 핵심은 서비스 계층을 순수하게 유지하는 것이다.

처음에는 JDBC로 직접 트랜잭션을 처리하면서 데이터베이스 동작 원리를 이해할 수 있었다.

하지만 직접 처리 방식은 서비스 계층에 JDBC 기술이 섞이고, 트랜잭션 코드와 예외 처리 코드가 반복되는 문제가 있었다.

스프링은 이 문제를 다음 방식으로 해결한다.

- `PlatformTransactionManager`로 트랜잭션 기술을 추상화한다.
- 트랜잭션 동기화 매니저로 같은 트랜잭션에서 같은 커넥션을 유지한다.
- `@Transactional`과 AOP로 트랜잭션 반복 코드를 제거한다.
- 체크 예외를 런타임 예외로 전환해 특정 기술 의존을 줄인다.
- `DataAccessException`으로 데이터 접근 예외를 추상화한다.
- `JdbcTemplate`으로 JDBC 반복 코드를 줄인다.

중요한 정리는 다음과 같다.

- 서비스 계층은 비즈니스 로직에 집중해야 한다.
- 트랜잭션은 서비스 계층에서 시작하는 것이 자연스럽지만, 기술 코드는 분리해야 한다.
- 같은 트랜잭션에서는 같은 커넥션이 유지되어야 한다.
- `@Transactional`은 트랜잭션을 선언적으로 적용한다.
- 기본적으로 언체크 예외를 사용하고, 비즈니스상 반드시 처리해야 하는 경우 체크 예외를 고려한다.
- 예외 전환 시 원인 예외를 포함해야 한다.
- JdbcTemplate은 JDBC 반복 코드를 제거해준다.

앞으로 DB 접근 코드를 작성할 때는 단순히 SQL 실행만 보는 것이 아니라, 트랜잭션 경계, 예외 처리 전략, 서비스 계층의 순수성까지 함께 고려해야겠다.
