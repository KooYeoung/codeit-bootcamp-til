# Spring 고급편 ThreadLocal, 템플릿 메서드, 콜백 패턴 정리

## 학습 날짜

2026-05-19

## 학습 주제

- 로그 추적기 예제
- 핵심 기능과 부가 기능
- ThreadLocal
- 동시성 문제
- 템플릿 메서드 패턴
- 전략 패턴
- 템플릿 콜백 패턴

---

## 1. 예제 프로젝트의 목적

스프링 핵심 원리 고급편에서는 먼저 간단한 주문 예제 프로젝트를 만든다.

구조는 일반적인 웹 애플리케이션처럼 Controller, Service, Repository로 나누어진다.

- `OrderController`
- `OrderService`
- `OrderRepository`

처음에는 각 계층이 자신의 핵심 기능만 수행한다.

예를 들어 Controller는 요청을 받고, Service는 주문 로직을 처리하고, Repository는 저장 로직을 수행한다.

이 상태에서는 코드가 단순하다.

하지만 여기에 로그 추적 기능이 요구사항으로 추가되면 문제가 시작된다.

---

## 2. 로그 추적기 요구사항

로그 추적기는 요청이 어떤 흐름으로 실행되는지 로그로 남기기 위한 기능이다.

요구사항은 다음과 같이 이해할 수 있다.

- 모든 public 메서드의 호출과 응답 정보를 로그로 남긴다.
- 애플리케이션의 흐름을 한눈에 파악할 수 있어야 한다.
- 메서드 호출 깊이를 표현할 수 있어야 한다.
- 정상 종료와 예외 발생을 구분할 수 있어야 한다.
- 요청 하나는 같은 트랜잭션 ID로 묶여야 한다.

예를 들어 하나의 요청이 Controller, Service, Repository로 이어진다면 다음처럼 표현할 수 있다.

```text
[abc123] OrderController.request()
[abc123] |-->OrderService.orderItem()
[abc123] | |-->OrderRepository.save()
[abc123] | |<--OrderRepository.save() time=1000ms
[abc123] |<--OrderService.orderItem() time=1001ms
[abc123] OrderController.request() time=1002ms
```

이런 로그가 있으면 장애가 발생했을 때 어떤 계층에서 문제가 생겼는지 추적하기 쉽다.

---

## 3. 핵심 기능과 부가 기능

애플리케이션 로직은 크게 핵심 기능과 부가 기능으로 나눌 수 있다.

## 핵심 기능

핵심 기능은 해당 객체가 원래 해야 하는 고유한 기능이다.

예를 들어 주문 서비스의 핵심 기능은 주문을 처리하는 것이다.

```java
public void orderItem(String itemId) {
    orderRepository.save(itemId);
}
```

## 부가 기능

부가 기능은 핵심 기능을 보조하기 위한 기능이다.

예를 들어 로그 추적, 트랜잭션, 권한 체크, 성능 측정 등이 있다.

부가 기능은 보통 여러 계층에 반복적으로 적용된다.

문제는 부가 기능 코드가 핵심 기능 코드 안에 섞이면 코드가 복잡해진다는 점이다.

---

## 4. 로그 추적기 직접 적용의 문제

로그 추적기를 직접 적용하면 다음과 같은 코드가 반복된다.

```java
TraceStatus status = null;

try {
    status = trace.begin("OrderService.orderItem()");
    orderRepository.save(itemId);
    trace.end(status);
} catch (Exception e) {
    trace.exception(status, e);
    throw e;
}
```

여기서 실제 핵심 기능은 다음 한 줄이다.

```java
orderRepository.save(itemId);
```

하지만 로그 추적을 위해 `try-catch`, `trace.begin()`, `trace.end()`, `trace.exception()` 같은 코드가 추가된다.

이렇게 되면 핵심 기능보다 부가 기능 코드가 더 많아진다.

클래스가 많아질수록 같은 패턴이 계속 반복되므로 유지보수가 어려워진다.

---

## 5. TraceId 파라미터 전달 방식의 한계

처음에는 로그 깊이와 트랜잭션 ID를 맞추기 위해 `TraceId`를 파라미터로 전달할 수 있다.

```java
public void orderItem(TraceId traceId, String itemId) {
    TraceStatus status = trace.beginSync(traceId, "OrderService.orderItem()");
    orderRepository.save(status.getTraceId(), itemId);
    trace.end(status);
}
```

이 방식은 로그 동기화는 가능하다.

하지만 모든 메서드에 `TraceId` 파라미터가 추가된다.

비즈니스 로직과 관계없는 파라미터가 계층 전체에 퍼지는 문제가 생긴다.

즉, 로그 추적이라는 부가 기능 때문에 핵심 기능의 메서드 시그니처가 오염된다.

---

## 6. 필드 동기화 방식과 동시성 문제

파라미터 전달 문제를 해결하기 위해 로그 추적기 내부 필드에 `TraceId`를 저장할 수 있다.

```java
private TraceId traceIdHolder;
```

이렇게 하면 메서드마다 `TraceId`를 전달하지 않아도 된다.

하지만 스프링 빈은 기본적으로 싱글톤이다.

싱글톤 객체의 필드는 여러 쓰레드가 함께 접근한다.

웹 애플리케이션에서 동시에 여러 요청이 들어오면 여러 쓰레드가 같은 `traceIdHolder` 필드를 변경할 수 있다.

그 결과 요청 A의 로그 ID가 요청 B에 섞이거나, 로그 level이 꼬이는 동시성 문제가 발생한다.

---

## 7. ThreadLocal

`ThreadLocal`은 쓰레드마다 별도의 저장 공간을 제공하는 기능이다.

일반 필드는 여러 쓰레드가 하나의 값을 공유한다.

반면 `ThreadLocal`은 같은 객체 안에 있어도 쓰레드별로 다른 값을 보관한다.

```java
private ThreadLocal<TraceId> traceIdHolder = new ThreadLocal<>();
```

값을 저장할 때는 `set()`을 사용한다.

```java
traceIdHolder.set(new TraceId());
```

값을 조회할 때는 `get()`을 사용한다.

```java
TraceId traceId = traceIdHolder.get();
```

값을 제거할 때는 `remove()`를 사용한다.

```java
traceIdHolder.remove();
```

이렇게 하면 하나의 로그 추적기 객체를 여러 쓰레드가 사용하더라도 각 요청의 로그 정보가 서로 섞이지 않는다.

---

## 8. ThreadLocal 사용 시 주의사항

ThreadLocal을 사용할 때 가장 중요한 것은 사용 후 값을 반드시 제거해야 한다는 점이다.

웹 애플리케이션 서버는 보통 쓰레드 풀을 사용한다.

쓰레드 풀은 요청이 끝났다고 쓰레드를 제거하지 않는다.

기존 쓰레드를 재사용한다.

만약 요청 A에서 ThreadLocal에 값을 저장하고 제거하지 않았다면, 같은 쓰레드가 요청 B를 처리할 때 요청 A의 값이 남아있을 수 있다.

따라서 요청 처리가 끝나면 반드시 `remove()`를 호출해야 한다.

```java
private void releaseTraceId() {
    if (traceIdHolder.get().isFirstLevel()) {
        traceIdHolder.remove();
    } else {
        traceIdHolder.set(traceIdHolder.get().createPreviousId());
    }
}
```

ThreadLocal은 동시성 문제를 해결하는 데 유용하지만, 쓰레드 풀 환경에서는 값 제거를 반드시 신경 써야 한다.

---

## 9. 변하는 것과 변하지 않는 것 분리

로그 추적기 적용 코드에는 반복되는 구조가 있다.

```java
try {
    trace.begin();
    핵심 기능 실행
    trace.end();
} catch (Exception e) {
    trace.exception();
    throw e;
}
```

여기서 변하지 않는 부분은 로그 추적 흐름이다.

변하는 부분은 실제 핵심 기능 실행 코드이다.

좋은 설계는 변하는 부분과 변하지 않는 부분을 분리하는 것이다.

이 문제를 해결하기 위해 템플릿 메서드 패턴, 전략 패턴, 템플릿 콜백 패턴을 사용할 수 있다.

---

## 10. 템플릿 메서드 패턴

템플릿 메서드 패턴은 부모 클래스에 변하지 않는 전체 흐름을 정의하고, 자식 클래스에서 변하는 부분을 구현하는 방식이다.

```java
public abstract class AbstractTemplate {

    public void execute() {
        long startTime = System.currentTimeMillis();

        call();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        System.out.println("resultTime=" + resultTime);
    }

    protected abstract void call();
}
```

자식 클래스는 변하는 부분만 구현한다.

```java
public class SubClassLogic1 extends AbstractTemplate {
    @Override
    protected void call() {
        System.out.println("비즈니스 로직1 실행");
    }
}
```

템플릿 메서드 패턴의 장점은 공통 흐름을 부모 클래스에서 재사용할 수 있다는 것이다.

단점은 상속을 사용하기 때문에 부모와 자식의 결합도가 높아질 수 있다는 점이다.

---

## 11. 전략 패턴

전략 패턴은 변하는 부분을 별도의 전략 객체로 분리하는 방식이다.

템플릿 메서드 패턴이 상속을 사용한다면, 전략 패턴은 위임을 사용한다.

```java
public interface Strategy {
    void call();
}
```

Context는 변하지 않는 흐름을 담당하고, 실제 핵심 로직은 Strategy에게 위임한다.

```java
public class Context {
    private final Strategy strategy;

    public Context(Strategy strategy) {
        this.strategy = strategy;
    }

    public void execute() {
        long startTime = System.currentTimeMillis();

        strategy.call();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        System.out.println("resultTime=" + resultTime);
    }
}
```

전략은 외부에서 주입할 수 있다.

```java
Context context = new Context(() -> System.out.println("비즈니스 로직 실행"));
context.execute();
```

전략 패턴은 상속보다 유연하게 동작을 교체할 수 있다.

---

## 12. 템플릿 콜백 패턴

템플릿 콜백 패턴은 전략 패턴의 변형으로 볼 수 있다.

Context를 실행할 때마다 콜백을 전달한다.

```java
public class TimeLogTemplate {

    public void execute(Callback callback) {
        long startTime = System.currentTimeMillis();

        callback.call();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        System.out.println("resultTime=" + resultTime);
    }
}
```

콜백 인터페이스는 다음과 같다.

```java
public interface Callback {
    void call();
}
```

사용할 때는 실행할 로직을 콜백으로 전달한다.

```java
template.execute(() -> System.out.println("비즈니스 로직 실행"));
```

템플릿 콜백 패턴은 변하지 않는 흐름은 템플릿이 가지고, 변하는 로직은 콜백으로 전달하는 방식이다.

스프링에서 `JdbcTemplate` 같은 구조도 이 패턴과 관련이 있다.

---

## 13. 세 패턴 비교

## 템플릿 메서드 패턴

- 상속 기반이다.
- 부모 클래스가 전체 흐름을 가진다.
- 자식 클래스가 변하는 부분을 구현한다.
- 상속으로 인한 결합도가 생긴다.

## 전략 패턴

- 위임 기반이다.
- Context가 전체 흐름을 가진다.
- Strategy 객체가 변하는 부분을 담당한다.
- 실행 로직을 객체로 교체할 수 있다.

## 템플릿 콜백 패턴

- 전략 패턴의 변형이다.
- 실행 시점에 콜백을 전달한다.
- 람다와 함께 사용하면 코드가 간결해진다.
- 스프링에서 자주 사용되는 방식이다.

---

## 14. 오늘 정리

오늘 학습한 내용의 핵심은 핵심 기능과 부가 기능을 분리하는 것이다.

처음에는 로그 추적기 요구사항을 단순히 로그를 출력하는 기능으로 볼 수 있다.

하지만 실제로 적용해보면 로그 추적 코드가 핵심 비즈니스 코드에 섞이고, 같은 코드가 여러 계층에 반복된다.

이 문제를 해결하기 위해 다음 흐름으로 발전한다.

1. 직접 로그 추적기 적용
2. `TraceId` 파라미터 전달
3. 필드 동기화
4. ThreadLocal 적용
5. 템플릿 메서드 패턴
6. 전략 패턴
7. 템플릿 콜백 패턴

ThreadLocal은 쓰레드별 값을 보관해 동시성 문제를 해결할 수 있지만, 쓰레드 풀 환경에서는 반드시 `remove()`를 호출해야 한다.

템플릿 메서드, 전략, 콜백 패턴은 모두 변하는 부분과 변하지 않는 부분을 분리하기 위한 방법이다.

앞으로 공통 로직이 반복될 때는 단순히 메서드 추출만 생각하지 말고, 변하는 부분과 변하지 않는 부분을 나누어 설계할 수 있는지 확인해야겠다.
