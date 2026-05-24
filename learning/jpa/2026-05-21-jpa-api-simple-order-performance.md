# Spring Boot JPA 활용2 - 지연 로딩과 단순 주문 조회 성능 최적화 정리

## 학습 날짜

2026-05-21

## 학습 주제

- API 개발 고급 준비
- 조회용 샘플 데이터
- 지연 로딩과 API 성능 문제
- 간단한 주문 조회 V1
- 간단한 주문 조회 V2
- 간단한 주문 조회 V3
- 간단한 주문 조회 V4
- 페치 조인
- DTO 직접 조회
- 쿼리 방식 선택 권장 순서

---

## 1. API 개발 고급 준비

API 개발 고급에서는 주문 조회 API를 기준으로 성능 최적화를 학습한다.

샘플 데이터는 회원, 상품, 주문, 배송 정보를 함께 가진다.

예시는 다음 흐름이다.

- userA 회원 생성
- JPA1 BOOK, JPA2 BOOK 생성
- userA가 두 상품 주문
- userB 회원 생성
- SPRING1 BOOK, SPRING2 BOOK 생성
- userB가 두 상품 주문

이 데이터는 주문 조회 시 다음 연관관계를 테스트하기 위해 사용된다.

- `Order -> Member`
- `Order -> Delivery`
- `Order -> OrderItem`
- `OrderItem -> Item`

단순 주문 조회에서는 먼저 xToOne 관계인 `Order -> Member`, `Order -> Delivery` 최적화에 집중한다.

---

## 2. 단순 주문 조회의 대상

단순 주문 조회 API는 주문, 회원, 배송 정보를 조회한다.

필요한 값은 다음과 같다.

- 주문 ID
- 회원 이름
- 주문 일시
- 주문 상태
- 배송 주소

엔티티 관계는 다음과 같다.

```text
Order -> Member   ManyToOne
Order -> Delivery OneToOne
```

이 둘은 모두 xToOne 관계이다.

xToOne 관계는 조인해도 데이터 row 수가 증가하지 않기 때문에 페치 조인 최적화가 비교적 쉽다.

---

## 3. V1 - 엔티티 직접 노출

V1은 엔티티를 직접 반환한다.

```java
@GetMapping("/api/v1/simple-orders")
public List<Order> ordersV1() {
    List<Order> all = orderRepository.findAllByString(new OrderSearch());
    for (Order order : all) {
        order.getMember().getName();
        order.getDelivery().getAddress();
    }
    return all;
}
```

`Order -> Member`, `Order -> Delivery`는 지연 로딩이다.

따라서 실제 엔티티 대신 프록시 객체가 들어있을 수 있다.

Jackson은 기본적으로 Hibernate 프록시 객체를 JSON으로 어떻게 직렬화해야 하는지 알지 못해 예외가 발생할 수 있다.

이를 해결하기 위해 Hibernate 모듈을 등록할 수 있다.

---

## 4. Hibernate 모듈과 한계

Spring Boot 3.0 미만에서는 `Hibernate5Module`을 사용할 수 있다.

```java
@Bean
Hibernate5Module hibernate5Module() {
    return new Hibernate5Module();
}
```

Spring Boot 3.0 이상에서는 `javax`에서 `jakarta`로 패키지가 변경되었으므로 `Hibernate5JakartaModule`을 사용해야 한다.

```java
@Bean
Hibernate5JakartaModule hibernate5Module() {
    return new Hibernate5JakartaModule();
}
```

이 모듈은 초기화된 프록시는 노출하고, 초기화되지 않은 프록시는 기본적으로 노출하지 않는다.

강제로 지연 로딩을 수행하도록 설정할 수도 있다.

하지만 이 방식은 실무에서 권장되지 않는다.

이유는 다음과 같다.

- 엔티티가 API 스펙에 직접 노출된다.
- 양방향 연관관계에서 무한 루프가 발생할 수 있다.
- `@JsonIgnore`가 엔티티에 들어가 API 응답에 종속된다.
- 엔티티 변경이 API 응답 변경으로 이어진다.

따라서 엔티티를 직접 노출하기보다 DTO로 변환해서 반환하는 것이 좋다.

---

## 5. 즉시 로딩으로 바꾸면 안 되는 이유

지연 로딩 문제가 있다고 해서 즉시 로딩으로 바꾸면 안 된다.

```java
@ManyToOne(fetch = FetchType.EAGER)
private Member member;
```

즉시 로딩은 연관관계가 필요 없는 상황에서도 데이터를 항상 조회한다.

이로 인해 예상하지 못한 SQL이 실행되고 성능 튜닝이 어려워진다.

실무에서는 기본적으로 지연 로딩을 사용하고, 필요한 조회에서 페치 조인으로 최적화하는 것이 좋다.

---

## 6. V2 - 엔티티를 DTO로 변환

V2는 엔티티를 조회한 뒤 DTO로 변환한다.

```java
@GetMapping("/api/v2/simple-orders")
public List<SimpleOrderDto> ordersV2() {
    List<Order> orders = orderRepository.findAllByString(new OrderSearch());
    return orders.stream()
            .map(SimpleOrderDto::new)
            .collect(Collectors.toList());
}
```

DTO는 다음처럼 구성할 수 있다.

```java
@Data
static class SimpleOrderDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public SimpleOrderDto(Order order) {
        orderId = order.getId();
        name = order.getMember().getName();
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        address = order.getDelivery().getAddress();
    }
}
```

이 방식은 엔티티를 API 응답으로 직접 노출하지 않는다는 점에서 V1보다 좋다.

하지만 DTO 생성자에서 `order.getMember().getName()`, `order.getDelivery().getAddress()`를 호출하면서 지연 로딩이 발생한다.

---

## 7. V2의 N+1 문제

V2는 쿼리가 다음처럼 실행될 수 있다.

```text
Order 조회 1번
Order -> Member 조회 N번
Order -> Delivery 조회 N번
```

즉, 총 `1 + N + N` 쿼리가 발생할 수 있다.

예를 들어 주문 결과가 4개라면 최악의 경우 다음처럼 실행된다.

```text
1 + 4 + 4 = 9번
```

단, 지연 로딩은 영속성 컨텍스트를 먼저 확인한다.

이미 같은 회원이나 배송 엔티티가 영속성 컨텍스트에 있으면 추가 SQL을 실행하지 않을 수 있다.

하지만 최악의 경우를 기준으로 성능 문제를 고려해야 한다.

---

## 8. V3 - 페치 조인 최적화

V3는 페치 조인을 사용해 xToOne 관계를 한 번에 조회한다.

```java
@GetMapping("/api/v3/simple-orders")
public List<SimpleOrderDto> ordersV3() {
    List<Order> orders = orderRepository.findAllWithMemberDelivery();
    return orders.stream()
            .map(SimpleOrderDto::new)
            .collect(Collectors.toList());
}
```

리포지토리에서는 JPQL 페치 조인을 사용한다.

```java
public List<Order> findAllWithMemberDelivery() {
    return em.createQuery(
            "select o from Order o" +
            " join fetch o.member m" +
            " join fetch o.delivery d", Order.class)
            .getResultList();
}
```

페치 조인으로 `Order`, `Member`, `Delivery`를 SQL 한 번에 조회한다.

이미 연관 엔티티가 조회된 상태이므로 DTO 변환 과정에서 추가 지연 로딩 SQL이 발생하지 않는다.

---

## 9. V3의 장점

V3의 장점은 다음과 같다.

- 쿼리 한 번으로 필요한 엔티티를 모두 조회한다.
- 엔티티를 조회하므로 리포지토리 재사용성이 좋다.
- DTO 변환은 컨트롤러나 서비스 계층에서 수행할 수 있다.
- 대부분의 xToOne 성능 문제를 해결할 수 있다.

xToOne 관계는 페치 조인해도 row 수가 증가하지 않으므로 페이징과 함께 사용해도 상대적으로 안전하다.

따라서 단순 주문 조회에서는 V3 방식이 실무적으로 가장 많이 권장된다.

---

## 10. V4 - JPA에서 DTO로 직접 조회

V4는 엔티티를 조회하지 않고 JPQL에서 DTO를 바로 생성한다.

```java
@GetMapping("/api/v4/simple-orders")
public List<OrderSimpleQueryDto> ordersV4() {
    return orderSimpleQueryRepository.findOrderDtos();
}
```

조회 전용 리포지토리를 따로 만든다.

```java
@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {

    private final EntityManager em;

    public List<OrderSimpleQueryDto> findOrderDtos() {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto" +
                "(o.id, m.name, o.orderDate, o.status, d.address)" +
                " from Order o" +
                " join o.member m" +
                " join o.delivery d", OrderSimpleQueryDto.class)
                .getResultList();
    }
}
```

DTO는 생성자 기반으로 값을 받는다.

```java
public class OrderSimpleQueryDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public OrderSimpleQueryDto(Long orderId, String name,
                               LocalDateTime orderDate,
                               OrderStatus orderStatus,
                               Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```

---

## 11. V4의 장점과 단점

V4의 장점은 다음과 같다.

- 필요한 컬럼만 선택해서 조회한다.
- 네트워크 전송량을 줄일 수 있다.
- DTO를 바로 생성하므로 엔티티 변환 과정이 없다.

단점은 다음과 같다.

- 리포지토리 재사용성이 떨어진다.
- API 스펙에 맞춘 DTO 조회 코드가 리포지토리에 들어간다.
- DTO 생성자의 패키지명을 JPQL에 직접 작성해야 한다.
- 엔티티를 조회하는 방식보다 코드가 더 복잡해질 수 있다.

자료에서도 네트워크 용량 최적화 효과는 생각보다 미비할 수 있다고 설명한다.

따라서 기본적으로는 엔티티 조회 후 DTO 변환을 먼저 고려하는 것이 좋다.

---

## 12. 쿼리 방식 선택 권장 순서

강의 자료의 권장 순서는 다음과 같다.

1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다.
2. 필요하면 페치 조인으로 성능을 최적화한다.
3. 그래도 해결이 안 되면 DTO로 직접 조회한다.
4. 최후에는 네이티브 SQL이나 Spring JdbcTemplate을 사용한다.

이 순서가 중요한 이유는 단순성과 재사용성 때문이다.

엔티티 조회 방식은 JPA가 제공하는 최적화 기능을 활용하기 쉽고, 코드가 단순하다.

DTO 직접 조회는 성능을 더 세밀하게 제어할 수 있지만, API 스펙에 맞춘 코드가 많아진다.

---

## 13. 보충: 조회 전용 리포지토리 분리

DTO 직접 조회를 사용할 경우 일반 리포지토리와 조회 전용 리포지토리를 분리하는 것이 좋다.

예를 들어 다음처럼 나눌 수 있다.

```text
OrderRepository
OrderSimpleQueryRepository
OrderQueryRepository
```

`OrderRepository`는 엔티티 저장, 조회, 검색 등 일반적인 도메인 중심 기능을 담당한다.

`OrderSimpleQueryRepository`는 특정 API 화면에 맞춘 조회 DTO를 반환한다.

이렇게 나누면 도메인 리포지토리에 API 스펙 의존 코드가 섞이는 것을 줄일 수 있다.

---

## 14. 보충: 읽기 전용 트랜잭션

조회 API에서는 읽기 전용 트랜잭션을 사용할 수 있다.

```java
@Transactional(readOnly = true)
public List<OrderSimpleDto> findOrders() {
    // 조회 로직
}
```

읽기 전용 트랜잭션은 변경 감지를 위한 불필요한 부담을 줄이는 데 도움이 될 수 있다.

또한 OSIV를 끄는 구조에서는 서비스 계층의 읽기 전용 트랜잭션 안에서 필요한 지연 로딩을 모두 처리해야 한다.

---

## 15. 오늘 정리

단순 주문 조회 최적화의 핵심은 지연 로딩으로 발생하는 N+1 문제를 어떻게 해결할지이다.

중요한 정리는 다음과 같다.

- 엔티티를 API 응답으로 직접 노출하지 않는다.
- Hibernate 모듈로 프록시 직렬화 문제를 해결하기보다 DTO를 사용한다.
- 지연 로딩 문제를 즉시 로딩으로 해결하면 안 된다.
- V2는 DTO 변환 방식이지만 지연 로딩으로 `1 + N + N` 쿼리가 발생할 수 있다.
- V3는 페치 조인으로 xToOne 관계를 한 번에 조회해 쿼리 수를 줄인다.
- V4는 DTO를 직접 조회하지만 리포지토리 재사용성이 떨어진다.
- 기본 권장 순서는 엔티티 조회 후 DTO 변환, 페치 조인 최적화, DTO 직접 조회, 네이티브 SQL 순서이다.

앞으로 조회 API를 만들 때는 먼저 단순한 엔티티 조회와 DTO 변환으로 시작하고, 실제 SQL과 성능 문제를 확인한 뒤 필요한 만큼만 최적화해야겠다.
