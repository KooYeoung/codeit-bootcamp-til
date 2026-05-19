# Spring DB2 데이터 접근 기술 비교 정리

## 학습 날짜

2026-05-18

## 학습 주제

- 데이터 접근 기술 큰 그림
- SQL Mapper와 ORM
- JdbcTemplate
- MyBatis
- 데이터 접근 테스트
- 데이터 접근 기술 활용 방안

---

## 1. 데이터 접근 기술을 왜 나누어 보는가

스프링 애플리케이션에서 데이터를 저장하고 조회하는 방법은 여러 가지가 있다.

대표적인 기술은 다음과 같다.

- JdbcTemplate
- MyBatis
- JPA, Hibernate
- 스프링 데이터 JPA
- Querydsl

이 기술들은 모두 데이터베이스에 접근한다는 공통점이 있지만, 개발자가 SQL을 직접 작성하는지, 객체 중심으로 개발하는지에 따라 성격이 다르다.

크게 나누면 다음과 같다.

- SQL Mapper
- ORM

---

## 2. SQL Mapper

SQL Mapper는 개발자가 SQL을 직접 작성하고, 그 결과를 객체로 매핑하는 방식이다.

대표적인 기술은 다음과 같다.

- JdbcTemplate
- MyBatis

SQL Mapper의 핵심은 SQL을 개발자가 직접 제어한다는 점이다.

장점은 다음과 같다.

- SQL을 직접 작성하므로 실행되는 쿼리를 명확히 알 수 있다.
- 복잡한 SQL을 직접 튜닝하기 좋다.
- SQL에 익숙한 개발자는 빠르게 적응할 수 있다.

단점은 다음과 같다.

- SQL을 직접 계속 작성해야 한다.
- 객체와 테이블 사이의 매핑 코드를 작성해야 한다.
- 필드가 추가되면 SQL과 매핑 코드도 함께 수정해야 할 수 있다.

---

## 3. ORM

ORM은 Object Relational Mapping의 약자이다.

객체와 관계형 데이터베이스 사이의 차이를 매핑해주는 기술이다.

대표적인 기술은 다음과 같다.

- JPA
- Hibernate
- 스프링 데이터 JPA
- Querydsl

JPA는 자바 진영의 ORM 표준이고, Hibernate는 JPA의 대표 구현체이다.

JPA를 사용하면 개발자는 객체를 저장하고 조회하는 방식으로 개발할 수 있다.

예를 들어 SQL을 직접 작성하는 대신 다음과 같은 방식으로 사용할 수 있다.

```java
entityManager.persist(item);
Item item = entityManager.find(Item.class, id);
```

JPA는 내부에서 필요한 SQL을 생성하고 실행한다.

ORM의 장점은 다음과 같다.

- SQL 중심 개발에서 객체 중심 개발로 이동할 수 있다.
- 기본 CRUD SQL 작성량이 줄어든다.
- 객체의 연관관계를 자연스럽게 표현할 수 있다.
- 생산성과 유지보수성이 좋아질 수 있다.

단점은 다음과 같다.

- 학습 곡선이 높다.
- 내부 동작 원리를 모르면 예상하지 못한 SQL이 실행될 수 있다.
- 복잡한 통계 쿼리에는 SQL Mapper가 더 적합할 수 있다.

---

## 4. JdbcTemplate

`JdbcTemplate`은 스프링이 제공하는 JDBC 편의 도구이다.

순수 JDBC를 직접 사용하면 다음 작업이 반복된다.

- 커넥션 획득
- Statement 준비
- SQL 실행
- ResultSet 반복
- 결과 객체 매핑
- 리소스 정리
- 예외 변환

`JdbcTemplate`은 이런 반복 작업을 대신 처리해준다.

개발자는 다음에 집중하면 된다.

- SQL 작성
- 파라미터 전달
- 결과 매핑

예를 들어 저장 로직은 다음처럼 작성할 수 있다.

```java
String sql = "insert into item(item_name, price, quantity) values (?, ?, ?)";
template.update(sql, item.getItemName(), item.getPrice(), item.getQuantity());
```

조회 결과를 객체로 바꿀 때는 `RowMapper`를 사용할 수 있다.

```java
private RowMapper<Item> itemRowMapper() {
    return (rs, rowNum) -> {
        Item item = new Item();
        item.setId(rs.getLong("id"));
        item.setItemName(rs.getString("item_name"));
        item.setPrice(rs.getInt("price"));
        item.setQuantity(rs.getInt("quantity"));
        return item;
    };
}
```

---

## 5. JdbcTemplate의 장점과 단점

## 장점

- 스프링에서 기본적으로 제공하므로 설정이 간단하다.
- JDBC 반복 코드를 크게 줄일 수 있다.
- SQL을 직접 작성하므로 쿼리를 명확히 제어할 수 있다.
- 단순 CRUD나 간단한 조회에 적합하다.

## 단점

- 동적 SQL을 작성하기 어렵다.
- 조건이 많아지면 문자열 조립 코드가 복잡해진다.
- SQL과 자바 코드가 섞여 가독성이 떨어질 수 있다.

예를 들어 검색 조건이 있을 때 조건에 따라 `where`, `and`를 직접 붙여야 한다.

이런 경우 조건이 늘어날수록 코드가 지저분해질 수 있다.

---

## 6. 이름 지정 파라미터

`JdbcTemplate`에서 기본 파라미터는 순서 기반이다.

```java
template.update(sql, itemName, price, quantity);
```

파라미터가 많아지면 순서가 헷갈릴 수 있다.

이 문제를 줄이기 위해 이름 지정 파라미터를 사용할 수 있다.

```java
SqlParameterSource param = new BeanPropertySqlParameterSource(item);
```

또는 Map 기반으로 파라미터를 전달할 수도 있다.

```java
MapSqlParameterSource param = new MapSqlParameterSource()
        .addValue("itemName", item.getItemName())
        .addValue("price", item.getPrice())
        .addValue("quantity", item.getQuantity());
```

이름 지정 파라미터를 사용하면 SQL의 파라미터 이름과 객체 필드를 더 명확히 연결할 수 있다.

---

## 7. SimpleJdbcInsert

`SimpleJdbcInsert`는 insert SQL을 직접 작성하지 않고 데이터를 저장할 수 있게 도와준다.

```java
SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(dataSource)
        .withTableName("item")
        .usingGeneratedKeyColumns("id");
```

자동 생성 키를 받을 때도 편리하다.

단순 insert 작업에서는 SQL 작성량을 더 줄일 수 있다.

다만 복잡한 저장 로직에서는 직접 SQL을 작성하는 편이 더 명확할 수 있다.

---

## 8. MyBatis

MyBatis는 SQL Mapper 기술이다.

JdbcTemplate보다 더 많은 기능을 제공하며, 특히 XML 기반 SQL 작성과 동적 쿼리에 강점이 있다.

JdbcTemplate에서는 긴 SQL을 문자열로 조립해야 한다.

```java
String sql = "update item " +
        "set item_name=:itemName, price=:price, quantity=:quantity " +
        "where id=:id";
```

MyBatis에서는 XML에 SQL을 자연스럽게 작성할 수 있다.

```xml
<update id="update">
    update item
    set item_name=#{itemName},
        price=#{price},
        quantity=#{quantity}
    where id = #{id}
</update>
```

SQL이 길어질수록 XML로 분리하는 방식이 더 읽기 쉬울 수 있다.

---

## 9. MyBatis 동적 쿼리

MyBatis의 큰 장점은 동적 쿼리이다.

검색 조건이 있을 때 조건에 따라 쿼리를 자동으로 조립할 수 있다.

```xml
<select id="findAll" resultType="Item">
    select id, item_name, price, quantity
    from item
    <where>
        <if test="itemName != null and itemName != ''">
            and item_name like concat('%', #{itemName}, '%')
        </if>
        <if test="maxPrice != null">
            and price &lt;= #{maxPrice}
        </if>
    </where>
</select>
```

`<where>`는 조건이 있을 때만 where를 붙여주고, 앞의 불필요한 `and`도 처리해준다.

따라서 동적 조건이 많을수록 MyBatis가 JdbcTemplate보다 편리할 수 있다.

---

## 10. JdbcTemplate과 MyBatis 선택 기준

## JdbcTemplate이 적합한 경우

- SQL이 단순하다.
- 동적 조건이 많지 않다.
- 별도 XML 설정 없이 빠르게 개발하고 싶다.
- 스프링 기본 기능만으로 충분하다.

## MyBatis가 적합한 경우

- SQL이 길고 복잡하다.
- 동적 쿼리가 많다.
- SQL을 자바 코드와 분리해서 관리하고 싶다.
- SQL 튜닝 중심의 프로젝트이다.

둘은 모두 SQL Mapper 계열이다.

즉, 개발자가 SQL을 직접 작성한다는 점은 같다.

---

## 11. 데이터 접근 테스트

데이터 접근 기술을 사용할 때는 실제 DB와 연동해서 저장과 조회가 정상 동작하는지 확인해야 한다.

테스트에서 중요한 점은 테스트 간 데이터가 서로 영향을 주지 않도록 만드는 것이다.

문제 상황은 다음과 같다.

- 테스트가 실제 로컬 DB를 사용한다.
- 이전 테스트에서 저장한 데이터가 남아 있다.
- 다음 테스트 결과에 영향을 준다.

이를 해결하는 방법은 다음과 같다.

- 테스트 전용 DB를 분리한다.
- 테스트마다 데이터를 초기화한다.
- `@Transactional`을 사용해 테스트 종료 후 롤백한다.
- 임베디드 DB를 사용한다.

테스트는 항상 반복 실행해도 같은 결과가 나와야 한다.

---

## 12. @Transactional 테스트

테스트에 `@Transactional`을 붙이면 테스트가 끝난 후 트랜잭션이 롤백된다.

```java
@Transactional
@SpringBootTest
class ItemRepositoryTest {
}
```

이렇게 하면 테스트 중 저장한 데이터가 테스트 종료 후 DB에 반영되지 않는다.

장점은 테스트 데이터를 직접 삭제하지 않아도 된다는 점이다.

주의할 점은 테스트에서 롤백되기 때문에 실제 커밋이 필요한 상황을 검증할 때는 별도 설정이 필요할 수 있다는 점이다.

---

## 13. 임베디드 DB

임베디드 DB는 애플리케이션 내부에서 실행되는 테스트용 DB이다.

별도로 H2 서버를 실행하지 않아도 테스트할 수 있다.

테스트 환경에서 임베디드 DB를 사용하면 다음 장점이 있다.

- 테스트 실행이 간편하다.
- 외부 DB 상태에 의존하지 않는다.
- 테스트 간 격리가 쉬워진다.

스프링 부트는 테스트에서 적절한 임베디드 DB 의존성이 있으면 자동으로 설정해줄 수 있다.

---

## 14. 데이터 접근 기술 활용 방안

데이터 접근 기술 선택에는 하나의 정답이 없다.

프로젝트 상황, 팀원의 역량, 쿼리 복잡도, 유지보수 전략에 따라 달라진다.

실용적인 방향은 다음과 같이 정리할 수 있다.

- 기본 CRUD와 단순 조회는 JPA, 스프링 데이터 JPA를 사용한다.
- 복잡한 동적 조회는 Querydsl을 사용한다.
- 매우 복잡한 통계 쿼리나 튜닝이 중요한 SQL은 JdbcTemplate 또는 MyBatis를 함께 사용한다.

즉, 한 가지 기술만 고집하기보다 상황에 맞게 조합할 수 있다.

다만 여러 기술을 조합할 때는 트랜잭션 매니저와 계층 구조를 잘 고려해야 한다.

---

## 15. 오늘 정리

오늘 데이터 접근 기술을 비교하면서 중요한 것은 “어떤 기술이 무조건 좋다”가 아니라 “어떤 상황에 어떤 기술이 적합한가”라는 점이다.

정리하면 다음과 같다.

- JdbcTemplate은 단순 SQL 처리에 편리하다.
- MyBatis는 복잡한 SQL과 동적 쿼리에 강하다.
- JPA는 객체 중심 개발과 생산성 향상에 강하다.
- 스프링 데이터 JPA는 JPA의 반복적인 리포지토리 코드를 줄여준다.
- Querydsl은 복잡한 동적 조회를 타입 안전하게 작성하게 해준다.
- 테스트에서는 DB 상태가 테스트 결과에 영향을 주지 않도록 격리해야 한다.
- 실무에서는 JPA 계열을 기본으로 두고, 복잡한 SQL은 SQL Mapper로 보완하는 조합이 가능하다.

앞으로 데이터 접근 코드를 작성할 때는 단순히 익숙한 기술을 쓰기보다, 쿼리 복잡도와 유지보수 비용을 함께 고려해야겠다.
