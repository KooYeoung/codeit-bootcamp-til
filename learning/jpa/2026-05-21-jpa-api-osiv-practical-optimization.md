# Spring Boot JPA 활용2 - OSIV와 실무 필수 최적화 정리

## 학습 날짜

2026-05-21

## 학습 주제

- OSIV
- Open Session In View
- Open EntityManager In View
- OSIV ON
- OSIV OFF
- 영속성 컨텍스트 생존 범위
- DB 커넥션 유지 문제
- 지연 로딩과 트랜잭션 범위
- Command와 Query 분리
- 조회 전용 서비스

---

## 1. OSIV란

OSIV는 Open Session In View의 약자이다.

하이버네이트에서는 Open Session In View라고 부르고, JPA에서는 Open EntityManager In View라고 볼 수 있다.

관례적으로 OSIV라고 부른다.

Spring Boot에서는 기본적으로 OSIV가 켜져 있다.

```yaml
spring:
  jpa:
    open-in-view: true
```

애플리케이션 시작 시점에 OSIV 관련 warn 로그가 출력되는 이유는 이 설정이 성능과 커넥션 사용에 중요한 영향을 주기 때문이다.

---

## 2. OSIV ON

OSIV가 켜져 있으면 영속성 컨텍스트가 트랜잭션 범위를 넘어 API 응답이 끝날 때까지 유지된다.

흐름은 다음과 같다.

```text
요청 시작
→ 필터/인터셉터
→ 컨트롤러
→ 서비스 트랜잭션 시작
→ 리포지토리
→ 서비스 트랜잭션 종료
→ 컨트롤러 응답 생성
→ API 응답 완료
→ 영속성 컨텍스트 종료
```

OSIV ON 상태에서는 서비스 트랜잭션이 끝난 뒤에도 영속성 컨텍스트가 살아있다.

그래서 컨트롤러나 뷰 템플릿에서 지연 로딩이 가능하다.

---

## 3. OSIV ON의 장점

OSIV ON의 가장 큰 장점은 편리함이다.

컨트롤러나 뷰에서 지연 로딩을 사용할 수 있다.

예를 들어 서비스에서 `Order`만 조회하고 컨트롤러에서 `order.getMember().getName()`을 호출해도 영속성 컨텍스트가 살아있으면 지연 로딩이 가능하다.

이 방식은 개발 초기에는 편리하다.

특히 뷰 템플릿을 사용하는 서버 사이드 렌더링에서는 화면을 만들면서 필요한 연관 데이터를 접근하기 쉽다.

---

## 4. OSIV ON의 단점

OSIV ON의 문제는 DB 커넥션을 오래 잡고 있을 수 있다는 점이다.

OSIV 전략은 최초 데이터베이스 커넥션을 사용하는 시점부터 API 응답이 끝날 때까지 영속성 컨텍스트와 DB 커넥션을 유지한다.

예를 들어 컨트롤러에서 외부 API를 호출한다고 하자.

```text
DB 조회
→ 외부 API 호출 대기
→ 응답 DTO 생성
→ API 응답
```

외부 API 응답을 기다리는 동안에도 DB 커넥션을 반환하지 못할 수 있다.

트래픽이 많은 서비스에서는 커넥션 풀이 빠르게 고갈될 수 있다.

결국 DB 커넥션 부족으로 장애가 발생할 수 있다.

---

## 5. OSIV OFF

OSIV를 끄려면 다음처럼 설정한다.

```yaml
spring:
  jpa:
    open-in-view: false
```

OSIV OFF 상태에서는 트랜잭션이 종료될 때 영속성 컨텍스트가 닫히고 DB 커넥션도 반환된다.

흐름은 다음과 같다.

```text
요청 시작
→ 컨트롤러
→ 서비스 트랜잭션 시작
→ 리포지토리
→ 서비스 트랜잭션 종료
→ 영속성 컨텍스트 종료
→ DB 커넥션 반환
→ 컨트롤러 응답 생성
→ API 응답 완료
```

DB 커넥션을 빨리 반환하므로 실시간 트래픽이 중요한 서비스에서 유리하다.

---

## 6. OSIV OFF의 장점

OSIV OFF의 장점은 커넥션 리소스를 효율적으로 사용할 수 있다는 점이다.

트랜잭션이 끝나면 DB 커넥션을 바로 반환한다.

따라서 API 응답을 만드는 시간이나 외부 API 대기 시간 동안 DB 커넥션을 붙잡고 있지 않는다.

실시간 트래픽이 많고 DB 커넥션 풀이 중요한 서비스에서는 OSIV OFF가 더 안전하다.

---

## 7. OSIV OFF의 단점

OSIV OFF 상태에서는 트랜잭션 밖에서 지연 로딩을 사용할 수 없다.

서비스 트랜잭션이 끝난 후 컨트롤러에서 지연 로딩을 시도하면 영속성 컨텍스트가 닫혀 있으므로 예외가 발생할 수 있다.

대표적으로 다음과 같은 문제가 생긴다.

```java
@GetMapping("/api/orders")
public List<OrderDto> orders() {
    List<Order> orders = orderService.findOrders();
    return orders.stream()
            .map(order -> new OrderDto(order)) // 여기서 지연 로딩 발생 가능
            .collect(Collectors.toList());
}
```

DTO 변환 과정에서 지연 로딩이 발생하면 트랜잭션이 이미 끝난 상태일 수 있다.

따라서 OSIV OFF에서는 필요한 지연 로딩을 트랜잭션 안에서 모두 처리해야 한다.

---

## 8. OSIV OFF에서의 해결 방향

OSIV OFF에서는 조회 로직을 서비스 계층 안으로 이동하는 것이 좋다.

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderQueryService {

    private final OrderRepository orderRepository;

    public List<OrderDto> orders() {
        List<Order> orders = orderRepository.findAllWithMemberDelivery();
        return orders.stream()
                .map(OrderDto::new)
                .collect(Collectors.toList());
    }
}
```

이렇게 하면 트랜잭션 안에서 엔티티 조회와 DTO 변환이 모두 끝난다.

컨트롤러는 이미 완성된 DTO를 반환하기만 한다.

```java
@GetMapping("/api/orders")
public List<OrderDto> orders() {
    return orderQueryService.orders();
}
```

---

## 9. Command와 Query 분리

OSIV를 끈 상태에서 복잡성을 관리하는 좋은 방법은 Command와 Query를 분리하는 것이다.

Command는 등록, 수정, 삭제 같은 비즈니스 변경 작업이다.

Query는 화면이나 API 응답을 위한 조회 작업이다.

두 작업은 성격이 다르다.

## Command

- 핵심 비즈니스 로직 중심
- 엔티티 상태 변경
- 트랜잭션 필수
- 도메인 규칙 중요
- 성능보다 정합성과 비즈니스 의미가 중요

## Query

- 화면이나 API 응답에 맞춘 조회
- 복잡한 조인과 최적화 필요
- 주로 읽기 전용 트랜잭션 사용
- DTO 직접 조회나 페치 조인 사용 가능
- 성능과 응답 스펙이 중요

---

## 10. 서비스 분리 예시

강의 자료에서는 다음처럼 서비스를 분리하는 방식을 제안한다.

```text
OrderService
OrderQueryService
```

## OrderService

`OrderService`는 핵심 비즈니스 로직을 담당한다.

예를 들어 주문 생성, 주문 취소 같은 기능이다.

```java
@Service
@Transactional
public class OrderService {
    public Long order(Long memberId, Long itemId, int count) {
        // 주문 비즈니스 로직
    }

    public void cancelOrder(Long orderId) {
        // 주문 취소 비즈니스 로직
    }
}
```

## OrderQueryService

`OrderQueryService`는 화면이나 API에 맞춘 조회를 담당한다.

```java
@Service
@Transactional(readOnly = true)
public class OrderQueryService {
    public List<OrderDto> findOrderDtos() {
        // 조회 최적화 로직
    }
}
```

이렇게 분리하면 비즈니스 로직과 조회 최적화 로직이 서로 섞이지 않는다.

---

## 11. 읽기 전용 트랜잭션

조회 서비스에는 읽기 전용 트랜잭션을 적용하는 것이 좋다.

```java
@Transactional(readOnly = true)
public List<OrderDto> findOrders() {
    // 조회 로직
}
```

읽기 전용 트랜잭션은 다음 장점이 있다.

- 조회 로직이라는 의도를 명확히 표현한다.
- 변경 감지 부담을 줄일 수 있다.
- OSIV OFF 상태에서도 트랜잭션 안에서 지연 로딩을 처리할 수 있다.

단, 읽기 전용 트랜잭션 안에서 엔티티를 수정하면 안 된다.

---

## 12. OSIV ON/OFF 선택 기준

OSIV는 무조건 켜거나 끄는 정답이 있는 것은 아니다.

서비스 성격에 따라 선택할 수 있다.

## OSIV ON이 유리할 수 있는 경우

- 관리자 페이지
- 트래픽이 많지 않은 서비스
- 개발 편의성이 중요한 내부 시스템
- 뷰 템플릿에서 지연 로딩 편의성이 필요한 경우

## OSIV OFF가 유리한 경우

- 실시간 트래픽이 많은 고객 서비스
- API 서버
- 외부 API 호출이 많은 서비스
- DB 커넥션 자원이 중요한 서비스
- 성능과 안정성이 중요한 서비스

강의 자료에서도 고객 서비스의 실시간 API는 OSIV를 끄고, ADMIN처럼 커넥션을 많이 사용하지 않는 곳에서는 OSIV를 켤 수 있다고 설명한다.

---

## 13. 보충: OSIV OFF에서 컨트롤러의 역할

OSIV OFF에서는 컨트롤러에서 엔티티를 만지며 지연 로딩을 발생시키면 안 된다.

컨트롤러는 다음 역할에 집중하는 것이 좋다.

- 요청 파라미터 받기
- 요청 DTO 검증
- 서비스 호출
- 서비스가 반환한 응답 DTO 반환

지연 로딩이 필요한 DTO 변환은 서비스 트랜잭션 안에서 처리하는 것이 안전하다.

---

## 14. 보충: 조회 최적화 체크리스트

API 조회 성능을 확인할 때는 다음을 점검한다.

- 엔티티가 API 응답으로 직접 노출되고 있지 않은가?
- DTO 변환 시 지연 로딩이 발생하는가?
- N+1 쿼리가 발생하는가?
- xToOne 관계는 페치 조인으로 해결 가능한가?
- 컬렉션 조회에서 페이징이 필요한가?
- 컬렉션 페치 조인으로 메모리 페이징이 발생하지 않는가?
- batch fetch size가 적절히 설정되어 있는가?
- DTO 직접 조회가 필요한 상황인가?
- OSIV 설정이 서비스 트래픽 성격에 맞는가?
- 외부 API 호출 중 DB 커넥션을 잡고 있지 않은가?

---

## 15. 보충: DTO 변환 위치

DTO 변환 위치는 OSIV 설정에 따라 달라질 수 있다.

## OSIV ON

컨트롤러에서 DTO 변환 중 지연 로딩이 가능하다.

하지만 커넥션을 오래 잡을 수 있고, 컨트롤러가 엔티티 내부 구조를 많이 알게 된다.

## OSIV OFF

컨트롤러에서 지연 로딩이 불가능하다.

따라서 서비스 또는 조회 전용 서비스 안에서 DTO 변환을 완료해야 한다.

실무 API 서버에서는 OSIV OFF와 조회 전용 서비스를 함께 사용하는 구조가 더 안정적일 수 있다.

---

## 16. 오늘 정리

OSIV는 JPA 애플리케이션의 영속성 컨텍스트 생존 범위를 결정하는 중요한 설정이다.

OSIV ON은 편리하지만 DB 커넥션을 오래 잡아 성능과 안정성 문제가 생길 수 있다.

OSIV OFF는 커넥션을 빨리 반환하지만, 모든 지연 로딩을 트랜잭션 안에서 처리해야 한다.

중요한 정리는 다음과 같다.

- OSIV ON은 API 응답 완료까지 영속성 컨텍스트를 유지한다.
- OSIV ON에서는 컨트롤러나 뷰에서도 지연 로딩이 가능하다.
- OSIV ON은 DB 커넥션을 오래 잡아 커넥션 부족 문제를 만들 수 있다.
- OSIV OFF는 트랜잭션 종료 시 영속성 컨텍스트와 DB 커넥션을 반환한다.
- OSIV OFF에서는 트랜잭션 안에서 필요한 데이터를 모두 조회해야 한다.
- Command와 Query를 분리하면 OSIV OFF의 복잡성을 관리하기 쉽다.
- 조회 전용 서비스는 `@Transactional(readOnly = true)`를 사용해 필요한 DTO를 완성해서 반환하는 것이 좋다.
- 실시간 고객 API는 OSIV OFF를 고려하고, 트래픽이 적은 관리자 화면은 OSIV ON도 선택할 수 있다.

앞으로 API 서버를 설계할 때는 단순히 지연 로딩이 되도록 만드는 것보다, 커넥션을 얼마나 오래 잡고 있는지와 트랜잭션 경계를 함께 고려해야겠다.
