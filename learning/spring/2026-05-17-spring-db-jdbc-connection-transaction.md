# Spring DB JDBC, 커넥션 풀, 트랜잭션 이해 정리

## 학습 날짜

2026-05-17

## 학습 주제

- JDBC 등장 이유
- JDBC 표준 인터페이스
- H2 데이터베이스 설정
- JDBC 등록, 조회, 수정, 삭제
- 커넥션 풀
- DataSource
- DriverManagerDataSource
- HikariCP
- 트랜잭션
- ACID
- 자동 커밋과 수동 커밋
- DB 세션과 락

---

## 1. JDBC를 학습하는 이유

애플리케이션에서 중요한 데이터는 대부분 데이터베이스에 저장한다.

Java 애플리케이션이 데이터베이스를 사용하려면 다음 작업이 필요하다.

1. 데이터베이스와 커넥션을 연결한다.
2. SQL을 데이터베이스에 전달한다.
3. 데이터베이스가 응답한 결과를 애플리케이션에서 사용한다.

문제는 데이터베이스 종류마다 연결 방법, SQL 전달 방법, 결과를 받는 방법이 다를 수 있다는 점이다.

만약 애플리케이션 코드가 특정 데이터베이스 사용 방식에 직접 묶여 있다면, 데이터베이스를 변경할 때 많은 코드를 수정해야 한다.

JDBC는 이런 문제를 해결하기 위해 등장한 Java의 데이터베이스 접근 표준 인터페이스이다.

JDBC 덕분에 개발자는 데이터베이스 종류가 달라도 비슷한 방식으로 DB에 접근할 수 있다.

---

## 2. JDBC의 기본 구성

JDBC를 직접 사용할 때 자주 등장하는 객체는 다음과 같다.

- `Connection`
- `PreparedStatement`
- `ResultSet`
- `DriverManager`

## Connection

`Connection`은 애플리케이션과 데이터베이스 사이의 연결을 의미한다.

SQL을 실행하려면 먼저 DB와 연결된 커넥션이 필요하다.

```java
Connection con = DriverManager.getConnection(url, username, password);
```

커넥션은 단순한 객체처럼 보이지만, 실제로는 DB와 연결된 중요한 자원이다.

사용이 끝나면 반드시 정리해야 한다.

## PreparedStatement

`PreparedStatement`는 SQL을 데이터베이스에 전달할 때 사용한다.

```java
String sql = "insert into member(member_id, money) values(?, ?)";
PreparedStatement pstmt = con.prepareStatement(sql);
pstmt.setString(1, member.getMemberId());
pstmt.setInt(2, member.getMoney());
pstmt.executeUpdate();
```

`?` 위치에 값을 바인딩해서 SQL을 실행한다.

이 방식은 문자열을 직접 이어붙이는 것보다 안전하고, 파라미터를 명확히 분리할 수 있다.

## ResultSet

`ResultSet`은 select 쿼리의 결과를 담는다.

```java
ResultSet rs = pstmt.executeQuery();

if (rs.next()) {
    String memberId = rs.getString("member_id");
    int money = rs.getInt("money");
}
```

`rs.next()`를 호출하면 다음 행으로 이동한다.

조회 결과가 있으면 값을 꺼내서 객체에 매핑할 수 있다.

---

## 3. H2 데이터베이스 설정

H2 데이터베이스는 개발과 테스트에 사용하기 좋은 가벼운 데이터베이스이다.

이번 학습에서는 `member` 테이블을 생성해 JDBC 실습을 진행한다.

```sql
drop table member if exists cascade;

create table member (
    member_id varchar(10),
    money integer not null default 0,
    primary key (member_id)
);

insert into member(member_id, money) values ('hi1', 10000);
insert into member(member_id, money) values ('hi2', 20000);
```

이 테이블은 회원 ID와 보유 금액을 저장한다.

Java에서는 다음과 같은 `Member` 객체로 데이터를 표현할 수 있다.

```java
public class Member {
    private String memberId;
    private int money;
}
```

---

## 4. JDBC 등록 흐름

회원 저장은 insert SQL을 사용한다.

흐름은 다음과 같다.

1. 커넥션을 획득한다.
2. SQL을 작성한다.
3. `PreparedStatement`를 생성한다.
4. SQL 파라미터를 바인딩한다.
5. SQL을 실행한다.
6. 사용한 리소스를 정리한다.

예시는 다음과 같다.

```java
String sql = "insert into member(member_id, money) values(?, ?)";

Connection con = getConnection();
PreparedStatement pstmt = con.prepareStatement(sql);

pstmt.setString(1, member.getMemberId());
pstmt.setInt(2, member.getMoney());

pstmt.executeUpdate();
```

`executeUpdate()`는 insert, update, delete처럼 데이터를 변경하는 SQL을 실행할 때 사용한다.

---

## 5. JDBC 조회 흐름

회원 조회는 select SQL을 사용한다.

흐름은 다음과 같다.

1. 커넥션을 획득한다.
2. SQL을 작성한다.
3. `PreparedStatement`를 생성한다.
4. 파라미터를 바인딩한다.
5. `executeQuery()`로 조회한다.
6. `ResultSet`에서 값을 꺼낸다.
7. 객체로 변환한다.
8. 리소스를 정리한다.

예시는 다음과 같다.

```java
String sql = "select * from member where member_id = ?";

Connection con = getConnection();
PreparedStatement pstmt = con.prepareStatement(sql);

pstmt.setString(1, memberId);

ResultSet rs = pstmt.executeQuery();

if (rs.next()) {
    Member member = new Member();
    member.setMemberId(rs.getString("member_id"));
    member.setMoney(rs.getInt("money"));
    return member;
}
```

조회 결과는 `ResultSet`으로 받고, 이 결과를 애플리케이션에서 사용할 객체로 직접 매핑해야 한다.

---

## 6. JDBC 수정과 삭제 흐름

수정은 update SQL을 사용한다.

```java
String sql = "update member set money=? where member_id=?";

PreparedStatement pstmt = con.prepareStatement(sql);
pstmt.setInt(1, money);
pstmt.setString(2, memberId);

pstmt.executeUpdate();
```

삭제는 delete SQL을 사용한다.

```java
String sql = "delete from member where member_id=?";

PreparedStatement pstmt = con.prepareStatement(sql);
pstmt.setString(1, memberId);

pstmt.executeUpdate();
```

등록, 수정, 삭제는 모두 `executeUpdate()`를 사용한다.

조회만 `executeQuery()`를 사용한다.

---

## 7. JDBC 직접 사용의 불편함

JDBC를 직접 사용하면 데이터베이스 동작 원리를 이해하기 좋다.

하지만 실제 코드에서는 반복이 많다.

반복되는 부분은 다음과 같다.

- 커넥션 획득
- `PreparedStatement` 생성
- 파라미터 바인딩
- SQL 실행
- `ResultSet` 처리
- 예외 처리
- 리소스 정리

특히 `try-catch-finally` 구조가 계속 반복된다.

이런 반복 문제는 나중에 `JdbcTemplate`이 해결해준다.

---

## 8. 커넥션을 매번 새로 만들 때의 문제

데이터베이스 커넥션을 새로 만들 때는 생각보다 많은 과정이 필요하다.

1. 애플리케이션이 DB 드라이버를 통해 커넥션을 요청한다.
2. DB 드라이버가 DB와 TCP/IP 연결을 맺는다.
3. ID, 비밀번호 등 인증 정보를 전달한다.
4. DB는 내부 인증을 수행한다.
5. DB 내부에 세션을 생성한다.
6. 커넥션 객체를 애플리케이션에 반환한다.

이 과정을 요청마다 반복하면 성능에 좋지 않다.

SQL 실행 시간뿐 아니라 커넥션 생성 시간까지 응답 시간에 포함되기 때문이다.

---

## 9. 커넥션 풀

커넥션 풀은 커넥션을 미리 만들어두고 재사용하는 방식이다.

애플리케이션 시작 시점에 필요한 만큼 커넥션을 생성해 풀에 보관한다.

요청이 들어오면 새 커넥션을 만들지 않고, 풀에 있는 커넥션을 빌려서 사용한다.

사용이 끝나면 커넥션을 닫는 것이 아니라 풀에 반환한다.

흐름은 다음과 같다.

1. 애플리케이션 시작 시점에 커넥션을 미리 생성한다.
2. 요청이 들어오면 풀에서 커넥션을 빌린다.
3. SQL을 실행한다.
4. 사용이 끝나면 커넥션을 풀에 반환한다.
5. 다음 요청에서 다시 재사용한다.

커넥션 풀을 사용하면 커넥션 생성 비용을 줄이고 응답 속도를 개선할 수 있다.

---

## 10. HikariCP

HikariCP는 스프링 부트에서 기본으로 많이 사용하는 커넥션 풀이다.

커넥션 풀의 최대 크기, 풀 이름 등을 설정할 수 있다.

```java
HikariDataSource dataSource = new HikariDataSource();
dataSource.setJdbcUrl(URL);
dataSource.setUsername(USERNAME);
dataSource.setPassword(PASSWORD);
dataSource.setMaximumPoolSize(10);
dataSource.setPoolName("MyPool");
```

커넥션 풀은 별도 쓰레드를 사용해서 커넥션을 채울 수 있다.

커넥션 생성은 상대적으로 시간이 걸릴 수 있으므로, 애플리케이션 시작을 오래 막지 않도록 별도 쓰레드에서 커넥션을 준비한다.

---

## 11. DataSource

`DataSource`는 커넥션을 획득하는 방법을 추상화한 인터페이스이다.

핵심 기능은 다음 메서드이다.

```java
Connection getConnection() throws SQLException;
```

기존 `DriverManager`는 커넥션을 얻을 때마다 URL, 사용자명, 비밀번호를 전달해야 했다.

```java
DriverManager.getConnection(URL, USERNAME, PASSWORD);
```

반면 `DataSource`를 사용하면 설정은 DataSource 생성 시점에 해두고, 사용하는 쪽에서는 다음처럼 호출한다.

```java
Connection con = dataSource.getConnection();
```

이렇게 하면 설정과 사용이 분리된다.

리포지토리 코드는 URL, 사용자명, 비밀번호를 알 필요 없이 `DataSource`에만 의존하면 된다.

---

## 12. DataSource의 장점

`DataSource`를 사용하면 커넥션 획득 방법을 쉽게 바꿀 수 있다.

예를 들어 처음에는 `DriverManagerDataSource`를 사용할 수 있다.

이후 성능을 위해 `HikariDataSource`로 바꾸더라도, 리포지토리 코드가 `DataSource` 인터페이스에만 의존하고 있다면 코드 변경을 줄일 수 있다.

즉, 구현체를 바꿔도 사용하는 코드는 그대로 둘 수 있다.

이것은 DI와 OCP 관점에서 장점이 있다.

- DI: 외부에서 필요한 구현체를 주입한다.
- OCP: 기존 코드를 크게 수정하지 않고 구현체를 교체할 수 있다.

---

## 13. 트랜잭션이 필요한 이유

트랜잭션은 하나의 거래를 안전하게 처리하기 위한 기능이다.

대표적인 예는 계좌이체이다.

A가 B에게 5000원을 이체한다고 하자.

1. A의 잔고를 5000원 감소시킨다.
2. B의 잔고를 5000원 증가시킨다.

이 두 작업은 반드시 하나의 작업처럼 처리되어야 한다.

첫 번째 작업만 성공하고 두 번째 작업이 실패하면 A의 돈만 사라지는 문제가 생긴다.

트랜잭션을 사용하면 두 작업이 모두 성공해야 반영하고, 하나라도 실패하면 모두 되돌릴 수 있다.

---

## 14. 커밋과 롤백

트랜잭션에서 모든 작업이 성공해 DB에 반영하는 것을 커밋이라고 한다.

```sql
commit;
```

반대로 작업 중 문제가 발생해 트랜잭션 시작 전 상태로 되돌리는 것을 롤백이라고 한다.

```sql
rollback;
```

커밋 전까지 변경 내용은 임시 상태이다.

트랜잭션을 시작한 세션에서는 변경 내용이 보이지만, 다른 세션에서는 아직 보이지 않을 수 있다.

이렇게 해야 다른 세션이 커밋되지 않은 데이터를 보고 잘못된 판단을 하는 문제를 막을 수 있다.

---

## 15. ACID

트랜잭션은 ACID를 보장해야 한다.

## 원자성

트랜잭션 안의 작업은 모두 성공하거나 모두 실패해야 한다.

계좌이체에서 출금만 성공하고 입금이 실패하면 안 된다.

## 일관성

트랜잭션이 끝난 후 데이터베이스는 일관성 있는 상태를 유지해야 한다.

예를 들어 제약 조건을 위반한 데이터가 저장되면 안 된다.

## 격리성

동시에 실행되는 트랜잭션들이 서로에게 영향을 주지 않도록 해야 한다.

다만 격리성을 너무 강하게 보장하면 성능이 떨어질 수 있어 격리 수준을 선택할 수 있다.

## 지속성

트랜잭션이 성공적으로 커밋되면 그 결과는 영구적으로 보관되어야 한다.

시스템 장애가 발생해도 커밋된 내용은 복구 가능해야 한다.

---

## 16. 자동 커밋과 수동 커밋

기본적으로 많은 데이터베이스는 자동 커밋 모드로 동작한다.

자동 커밋은 SQL 하나를 실행할 때마다 바로 커밋하는 방식이다.

```sql
set autocommit true;
```

하지만 계좌이체처럼 여러 SQL을 하나의 작업으로 묶어야 할 때는 자동 커밋이 위험하다.

중간에 실패하면 앞에서 실행한 SQL은 이미 커밋되어 되돌리기 어렵기 때문이다.

이럴 때는 수동 커밋 모드로 변경한다.

```sql
set autocommit false;
```

수동 커밋 모드에서는 개발자가 직접 `commit` 또는 `rollback`을 호출해야 한다.

보통 자동 커밋을 끄고 수동 커밋 모드로 들어가는 것을 트랜잭션을 시작한다고 표현한다.

---

## 17. DB 세션과 커넥션

클라이언트가 DB에 연결하면 DB 내부에는 세션이 생성된다.

커넥션을 통해 전달되는 SQL은 해당 세션에서 실행된다.

커넥션 풀이 10개의 커넥션을 만들면 DB 내부에도 보통 10개의 세션이 만들어진다.

트랜잭션은 세션 단위로 관리된다.

즉, 같은 트랜잭션을 유지하려면 같은 커넥션을 사용해야 한다.

다른 커넥션을 사용하면 다른 세션에서 SQL이 실행되므로 같은 트랜잭션으로 묶이지 않는다.

이 점이 스프링 트랜잭션 동기화를 이해할 때 중요하다.

---

## 18. DB 락

동시에 여러 세션이 같은 데이터를 수정하려고 하면 문제가 생길 수 있다.

예를 들어 세션1이 memberA의 금액을 500원으로 변경하고 아직 커밋하지 않았는데, 세션2도 같은 데이터를 수정하려고 한다면 데이터 정합성이 깨질 수 있다.

DB는 이런 문제를 막기 위해 락을 사용한다.

세션1이 특정 로우를 수정하면 해당 로우의 락을 가진다.

세션1이 커밋하거나 롤백하기 전까지 다른 세션은 해당 로우를 수정하지 못하고 기다릴 수 있다.

락은 동시성 환경에서 데이터 정합성을 지키기 위한 중요한 장치이다.

---

## 19. 오늘 정리

오늘 학습한 JDBC, 커넥션 풀, DataSource, 트랜잭션은 스프링 데이터 접근 기술을 이해하기 위한 기초이다.

JDBC는 Java에서 DB에 접근하기 위한 표준이고, 커넥션 풀은 DB 커넥션 생성 비용을 줄이기 위한 기술이다.

DataSource는 커넥션을 얻는 방법을 추상화해서 애플리케이션 코드가 특정 커넥션 획득 방식에 묶이지 않도록 도와준다.

트랜잭션은 여러 SQL을 하나의 작업처럼 묶어 데이터 정합성을 지키는 기능이다.

중요한 정리는 다음과 같다.

- JDBC는 Java의 DB 접근 표준이다.
- JDBC 직접 사용은 동작 원리 이해에는 좋지만 반복 코드가 많다.
- 커넥션 풀은 커넥션을 미리 만들어두고 재사용한다.
- DataSource는 커넥션 획득 방법을 추상화한다.
- 트랜잭션은 모두 성공하면 커밋하고, 실패하면 롤백한다.
- 같은 트랜잭션을 유지하려면 같은 커넥션을 사용해야 한다.
- DB 락은 동시에 같은 데이터를 수정할 때 정합성을 지키기 위해 필요하다.

앞으로 스프링 트랜잭션과 JdbcTemplate을 이해할 때 오늘 정리한 JDBC와 트랜잭션 기본 흐름을 계속 기준으로 삼아야겠다.
