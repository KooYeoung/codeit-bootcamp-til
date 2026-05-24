# Spring Boot JPA 활용2 - 컬렉션 조회 최적화 정리

## 학습 날짜

2026-05-21

## 학습 주제

- 주문 조회 API
- 컬렉션 조회 최적화
- 엔티티 직접 노출 V1
- 엔티티를 DTO로 변환 V2
- 컬렉션 페치 조인 V3
- 페이징과 한계 돌파 V3.1
- DTO 직접 조회 V4
- DTO 직접 조회 컬렉션 최적화 V5
- 플랫 데이터 최적화 V6
- batch fetch size
- `array_contains`

---

## 1. 컬렉션 조회 최적화가 필요한 이유

단순 주문 조회에서는 `Order -> Member`, `Order -> Delivery` 같은 xToOne 관계만 다루었다.

이번에는 주문 내역에 주문 상품 정보까지 포함한다.

필요한 관계는 다음과 같다.

```text
Order -> Member      ManyToOne
Order -> Delivery    OneToOne
Order -> OrderItems  OneToMany
OrderItem -> Item    ManyToOne
```

여기서 핵심은 `Order -> OrderItems`가 컬렉션이라는 점이다.

컬렉션은 일대다 관계이므로 조인하면 row 수가 증가한다.

이 때문에 페치 조인, 페이징, DTO 조회 방식에서 많은 주의가 필요하다.

---

## 2. V1 - 엔티티 직접 노출

V1은 엔티티를 직접 반환한다.

```java
@GetMapping("/api/v1/orders")
public List<Order> ordersV1() {
    List<Order> all = orderRepository.findAllByString(new OrderSearch());
    for (Order order : all) {
        order.getMember().getName();
        order.getDelivery().getAddress();
        List<OrderItem> orderItems = order.getOrderItems();
        orderItems.forEach(o -> o.getItem().getName());
    }
    return all;
}
```

지연 로딩된 연관관계를 강제로 초기화한 뒤 엔티티를 반환한다.

하지만 이 방식은 좋지 않다.

문제는 다음과 같다.

- 엔티티가 API 스펙으로 노출된다.
- 엔티티 변경이 API 변경으로 이어진다.
- 양방향 연관관계에서 무한 루프가 발생할 수 있다.
- `@JsonIgnore` 같은 API 응답 제어 코드가 엔티티에 들어간다.
- 지연 로딩 초기화 위치가 컨트롤러에 섞인다.

---

## 3. V2 - 엔티티를 DTO로 변환

V2는 엔티티를 조회한 뒤 DTO로 변환한다.

```java
@GetMapping("/api/v2/orders")
public List<OrderDto> ordersV2() {
    List<Order> orders = orderRepository.findAllByString(new OrderSearch());
    return orders.stream()
            .map(OrderDto::new)
            .collect(Collectors.toList());
}
```

주문 DTO는 주문 기본 정보와 주문 상품 목록을 가진다.

```java
@Data
static class OrderDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemDto> orderItems;

    public OrderDto(Order order) {
        orderId = order.getId();
        name = order.getMember().getName();
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        address = order.getDelivery().getAddress();
        orderItems = order.getOrderItems().stream()
                .map(OrderItemDto::new)
                .collect(Collectors.toList());
    }
}
```

이 방식은 엔티티 직접 노출 문제는 해결한다.

하지만 지연 로딩으로 많은 SQL이 실행된다.

---

## 4. V2의 SQL 실행 문제

V2는 다음과 같은 SQL이 실행될 수 있다.

```text
Order 조회 1번
Member 조회 N번
Delivery 조회 N번
OrderItem 조회 N번
Item 조회 N번
```

주문 수와 주문 상품 수가 많아질수록 쿼리 수가 급격히 증가한다.

이를 N+1 문제라고 부를 수 있다.

지연 로딩은 영속성 컨텍스트에 이미 로딩된 엔티티가 있으면 SQL을 생략한다.

하지만 운영 환경에서는 데이터가 다양하므로 최악의 쿼리 수를 기준으로 성능을 고려해야 한다.

---

## 5. V3 - 컬렉션 페치 조인

V3는 페치 조인으로 모든 연관관계를 한 번에 조회한다.

```java
public List<Order> findAllWithItem() {
    return em.createQuery(
            "select distinct o from Order o" +
            " join fetch o.member m" +
            " join fetch o.delivery d" +
            " join fetch o.orderItems oi" +
            " join fetch oi.item i", Order.class)
            .getResultList();
}
```

이 방식은 SQL 한 번으로 주문, 회원, 배송, 주문상품, 상품을 모두 조회한다.

`distinct`를 사용하는 이유는 일대다 조인 때문이다.

`Order` 하나에 `OrderItem`이 여러 개 있으면 DB row가 주문상품 수만큼 증가한다.

JPA의 `distinct`는 SQL에 `distinct`를 추가하고, 추가로 애플리케이션에서 같은 엔티티 중복을 제거한다.

---

## 6. V3의 장점과 단점

V3의 장점은 명확하다.

- SQL이 한 번만 실행된다.
- DTO 변환 시 추가 지연 로딩이 발생하지 않는다.
- 코드가 비교적 단순하다.

하지만 단점도 크다.

- 컬렉션 페치 조인은 페이징이 불가능하다.
- 일대다 조인으로 데이터 row 수가 증가한다.
- 하이버네이트가 모든 데이터를 DB에서 읽고 메모리에서 페이징할 수 있다.
- 컬렉션 페치 조인은 하나만 사용하는 것이 안전하다.

특히 페이징이 필요한 API에서 컬렉션 페치 조인은 매우 위험하다.

운영 데이터가 많으면 장애로 이어질 수 있다.

---

## 7. V3.1 - 페이징과 한계 돌파

컬렉션 조회에서 페이징이 필요하면 V3 방식 대신 V3.1 방식을 사용한다.

핵심 전략은 다음과 같다.

1. ToOne 관계는 모두 페치 조인한다.
2. 컬렉션은 지연 로딩으로 남긴다.
3. 컬렉션 지연 로딩은 `hibernate.default_batch_fetch_size` 또는 `@BatchSize`로 최적화한다.

ToOne 관계는 조인해도 row 수가 증가하지 않으므로 페이징에 영향을 주지 않는다.

```java
public List<Order> findAllWithMemberDelivery(int offset, int limit) {
    return em.createQuery(
            "select o from Order o" +
            " join fetch o.member m" +
            " join fetch o.delivery d", Order.class)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();
}
```

---

## 8. batch fetch size

컬렉션 지연 로딩을 최적화하려면 batch fetch size를 적용한다.

전역 설정은 다음과 같다.

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 1000
```

개별 설정은 `@BatchSize`를 사용할 수 있다.

- 컬렉션은 컬렉션 필드에 적용
- 엔티티 프록시는 엔티티 클래스에 적용

이 설정을 사용하면 지연 로딩 대상들을 하나씩 조회하지 않고, IN 쿼리로 묶어서 조회한다.

예를 들어 주문 100개에 대한 주문상품 컬렉션을 각각 조회하는 대신, order_id를 IN 절로 묶어 한 번에 조회할 수 있다.

---

## 9. V3.1의 장점

V3.1 방식의 장점은 다음과 같다.

- 페이징이 가능하다.
- ToOne 관계는 페치 조인으로 쿼리 수를 줄인다.
- 컬렉션은 batch fetch size로 `1 + N`을 `1 + 1` 수준으로 줄일 수 있다.
- 컬렉션 페치 조인보다 중복 데이터 전송량이 줄어든다.
- 코드 변경이 크지 않고 설정으로 최적화할 수 있다.

실무에서 페이징이 필요한 컬렉션 조회는 이 방식이 매우 유용하다.

자료에서는 batch size를 100~1000 사이에서 선택하는 것을 권장한다.

데이터베이스 IN 절 제한과 순간 부하를 함께 고려해야 한다.

---

## 10. Spring Boot 3.1과 Hibernate 6.2의 array_contains

Spring Boot 3.1부터 Hibernate 6.2를 사용한다.

이 버전에서는 batch fetch 최적화 시 `where in` 대신 `array_contains`를 사용할 수 있다.

기존 방식은 IN 절 파라미터 개수에 따라 SQL 모양이 달라진다.

```sql
where item_id in (?)
where item_id in (?, ?)
where item_id in (?, ?, ?, ?)
```

SQL 모양이 달라지면 DB가 SQL 구문 캐시를 재사용하기 어렵다.

`array_contains`는 배열 하나를 바인딩하므로 SQL 모양을 일정하게 유지할 수 있다.

```sql
where array_contains(?, item_id)
```

결과는 IN 절과 같지만, SQL 구문 캐싱 측면에서 유리할 수 있다.

---

## 11. V4 - JPA에서 DTO 직접 조회

V4는 엔티티를 조회하지 않고 DTO를 직접 조회한다.

핵심 흐름은 다음과 같다.

1. ToOne 관계를 포함한 주문 기본 정보를 DTO로 먼저 조회한다.
2. 각 주문 DTO마다 주문상품 컬렉션을 별도 쿼리로 조회한다.

```java
public List<OrderQueryDto> findOrderQueryDtos() {
    List<OrderQueryDto> result = findOrders();

    result.forEach(o -> {
        List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
        o.setOrderItems(orderItems);
    });

    return result;
}
```

이 방식은 쿼리가 루트 1번, 컬렉션 N번 실행된다.

단건 조회에서는 괜찮을 수 있지만, 여러 주문을 한 번에 조회하면 성능 문제가 생긴다.

---

## 12. V5 - DTO 직접 조회 컬렉션 최적화

V5는 V4의 컬렉션 N번 조회 문제를 최적화한다.

핵심 흐름은 다음과 같다.

1. 주문 기본 정보 DTO를 먼저 조회한다.
2. 주문 ID 목록을 추출한다.
3. 주문 ID 목록으로 주문상품 DTO를 IN 절로 한 번에 조회한다.
4. 주문 ID 기준으로 Map을 만든다.
5. 각 주문 DTO에 주문상품 DTO 목록을 매칭한다.

```java
public List<OrderQueryDto> findAllByDto_optimization() {
    List<OrderQueryDto> result = findOrders();

    Map<Long, List<OrderItemQueryDto>> orderItemMap =
            findOrderItemMap(toOrderIds(result));

    result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));

    return result;
}
```

컬렉션 조회 쿼리는 한 번만 실행된다.

```java
private Map<Long, List<OrderItemQueryDto>> findOrderItemMap(List<Long> orderIds) {
    List<OrderItemQueryDto> orderItems = em.createQuery(
            "select new ...OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
            " from OrderItem oi" +
            " join oi.item i" +
            " where oi.order.id in :orderIds", OrderItemQueryDto.class)
            .setParameter("orderIds", orderIds)
            .getResultList();

    return orderItems.stream()
            .collect(Collectors.groupingBy(OrderItemQueryDto::getOrderId));
}
```

---

## 13. V5의 장점

V5의 장점은 다음과 같다.

- 루트 1번, 컬렉션 1번으로 조회한다.
- 여러 주문을 한꺼번에 조회할 때 V4보다 훨씬 효율적이다.
- Map을 사용해 주문 ID 기준으로 O(1)에 가깝게 매칭할 수 있다.
- 페이징도 가능하다.

단점은 코드가 V4보다 복잡해진다는 점이다.

하지만 운영에서 여러 건을 조회하는 API라면 V5 방식이 성능상 큰 차이를 만들 수 있다.

---

## 14. V6 - 플랫 데이터 최적화

V6는 조인 결과를 플랫하게 DTO로 한 번에 조회한다.

```java
public List<OrderFlatDto> findAllByDto_flat() {
    return em.createQuery(
            "select new ...OrderFlatDto(o.id, m.name, o.orderDate, o.status, d.address, i.name, oi.orderPrice, oi.count)" +
            " from Order o" +
            " join o.member m" +
            " join o.delivery d" +
            " join o.orderItems oi" +
            " join oi.item i", OrderFlatDto.class)
            .getResultList();
}
```

조회 결과는 주문 기준 계층 구조가 아니라 row 단위 플랫 구조이다.

따라서 애플리케이션에서 다시 그룹핑해야 한다.

```java
flats.stream()
    .collect(groupingBy(..., mapping(..., toList())))
```

---

## 15. V6의 장점과 단점

V6의 장점은 쿼리가 한 번만 실행된다는 것이다.

하지만 단점이 많다.

- 조인으로 인해 중복 데이터가 증가한다.
- 애플리케이션에서 그룹핑 작업이 필요하다.
- 코드가 복잡하다.
- Order 기준 페이징이 불가능하다.
- 데이터가 많으면 V5보다 항상 빠르다고 볼 수 없다.

쿼리 수만 보면 V6가 좋아 보이지만, 실무에서는 페이징이 중요하므로 선택하기 어려운 경우가 많다.

---

## 16. 권장 순서

컬렉션 조회 최적화 권장 순서는 다음과 같다.

1. 엔티티 조회 방식으로 우선 접근한다.
2. ToOne 관계는 페치 조인으로 최적화한다.
3. 컬렉션이 있고 페이징이 필요하면 batch fetch size로 최적화한다.
4. 페이징이 필요 없으면 컬렉션 페치 조인을 고려할 수 있다.
5. 엔티티 조회 방식으로 해결이 안 되면 DTO 직접 조회를 사용한다.
6. DTO 직접 조회에서도 필요하면 V5처럼 컬렉션을 IN 절로 한 번에 조회한다.
7. 그래도 안 되면 네이티브 SQL이나 Spring JdbcTemplate을 고려한다.

---

## 17. V4, V5, V6 비교

## V4

- 루트 1번, 컬렉션 N번 조회
- 코드가 단순하다.
- 단건 조회에는 괜찮다.
- 여러 건 조회에서는 성능 문제가 크다.

## V5

- 루트 1번, 컬렉션 1번 조회
- IN 절과 Map으로 최적화한다.
- 여러 건 조회에 적합하다.
- 코드가 V4보다 복잡하다.
- 페이징이 가능하다.

## V6

- 쿼리 1번 조회
- 플랫 데이터로 받아 애플리케이션에서 그룹핑한다.
- 중복 데이터 전송이 많을 수 있다.
- 페이징이 어렵다.
- 실무에서 항상 좋은 선택은 아니다.

---

## 18. 보충: 성능 최적화에서 확인할 것

성능 최적화는 쿼리 수만 보고 판단하면 안 된다.

다음 기준을 함께 확인해야 한다.

- 실행 쿼리 수
- DB에서 애플리케이션으로 전송되는 데이터 양
- 중복 row 발생 여부
- 페이징 가능 여부
- 애플리케이션 메모리 사용량
- 코드 복잡도
- 리포지토리 재사용성
- API 스펙 변경 시 영향 범위

특히 컬렉션 조인은 쿼리 수가 적어도 중복 데이터가 많아질 수 있다.

따라서 실제 SQL과 실행 계획, 응답 시간, 데이터 양을 함께 확인해야 한다.

---

## 19. 오늘 정리

컬렉션 조회 최적화는 JPA API 성능 최적화에서 가장 중요한 부분이다.

중요한 정리는 다음과 같다.

- 컬렉션 관계는 일대다 조인으로 row 수가 증가한다.
- 엔티티 직접 노출은 피한다.
- 엔티티를 DTO로 변환해도 지연 로딩으로 N+1 문제가 발생할 수 있다.
- 컬렉션 페치 조인은 쿼리 수를 줄이지만 페이징이 불가능하다.
- 페이징이 필요하면 ToOne만 페치 조인하고 컬렉션은 batch fetch size로 최적화한다.
- DTO 직접 조회 V4는 단건 조회에는 괜찮지만 여러 건 조회에서는 N 문제가 생긴다.
- V5는 IN 절과 Map으로 컬렉션을 한 번에 조회해 최적화한다.
- V6는 쿼리 한 번이지만 중복 데이터와 페이징 한계가 있다.
- 최적화는 성능과 코드 복잡도 사이의 균형을 고려해야 한다.

앞으로 컬렉션 조회 API를 만들 때는 페이징이 필요한지 먼저 판단하고, 필요한 경우 V3.1 또는 V5 방식으로 접근해야겠다.
