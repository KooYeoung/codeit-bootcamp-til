# Spring 고급편 프록시, 동적 프록시, ProxyFactory, 빈 후처리기 정리

## 학습 날짜

2026-05-19

## 학습 주제

- 프록시 패턴
- 데코레이터 패턴
- 인터페이스 기반 프록시
- 구체 클래스 기반 프록시
- 리플렉션
- JDK 동적 프록시
- CGLIB
- ProxyFactory
- Pointcut, Advice, Advisor
- BeanPostProcessor
- 자동 프록시 생성기
- @Aspect 기반 프록시

---

## 1. 프록시가 필요한 이유

로그 추적기 같은 부가 기능을 모든 Controller, Service, Repository에 직접 넣으면 코드가 복잡해진다.

핵심 기능 코드는 그대로 두고, 부가 기능을 바깥에서 적용할 수 있는 방법이 필요하다.

이때 사용할 수 있는 대표적인 방법이 프록시이다.

프록시는 실제 대상 객체 대신 클라이언트의 요청을 먼저 받는 대리 객체이다.

클라이언트는 프록시를 실제 객체처럼 사용한다.

프록시는 요청을 받으면 필요한 부가 기능을 수행한 뒤 실제 대상 객체를 호출한다.

```text
클라이언트 → 프록시 → 실제 대상 객체
```

---

## 2. 프록시의 주요 목적

프록시는 같은 구조를 사용하더라도 목적에 따라 다르게 부를 수 있다.

## 프록시 패턴

프록시 패턴은 접근 제어가 목적이다.

예를 들어 다음과 같은 상황에서 사용할 수 있다.

- 권한 확인
- 캐싱
- 지연 로딩
- 실제 객체 호출 여부 제어

## 데코레이터 패턴

데코레이터 패턴은 기능 확장이 목적이다.

예를 들어 다음과 같은 상황에서 사용할 수 있다.

- 로그 추가
- 실행 시간 측정
- 부가 메시지 추가
- 기존 기능에 새로운 기능 조합

두 패턴은 구조가 비슷하지만, 의도가 다르다.

정리하면 다음과 같다.

- 프록시 패턴: 접근 제어
- 데코레이터 패턴: 기능 추가

---

## 3. 인터페이스 기반 프록시

인터페이스가 있는 경우 프록시는 인터페이스를 구현하는 방식으로 만들 수 있다.

```java
public interface OrderService {
    void orderItem(String itemId);
}
```

프록시는 같은 인터페이스를 구현한다.

```java
public class OrderServiceProxy implements OrderService {

    private final OrderService target;
    private final LogTrace trace;

    public OrderServiceProxy(OrderService target, LogTrace trace) {
        this.target = target;
        this.trace = trace;
    }

    @Override
    public void orderItem(String itemId) {
        TraceStatus status = trace.begin("OrderService.orderItem()");
        try {
            target.orderItem(itemId);
            trace.end(status);
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }
}
```

클라이언트는 `OrderService` 인터페이스에 의존하므로 실제 구현체가 원본인지 프록시인지 알 필요가 없다.

---

## 4. 구체 클래스 기반 프록시

인터페이스가 없는 클래스도 프록시를 만들 수 있다.

이 경우 상속을 사용해 프록시를 만든다.

```java
public class OrderServiceProxy extends OrderService {
}
```

구체 클래스 기반 프록시는 인터페이스 없이도 적용할 수 있다는 장점이 있다.

하지만 상속 기반이기 때문에 제약이 있다.

- `final` 클래스는 상속할 수 없다.
- `final` 메서드는 오버라이딩할 수 없다.
- 생성자 호출 관련 제약을 고려해야 한다.

---

## 5. 직접 프록시를 만들 때의 문제

프록시를 직접 만들면 대상 클래스마다 프록시 클래스를 만들어야 한다.

예를 들어 다음과 같은 프록시가 필요할 수 있다.

- `OrderControllerProxy`
- `OrderServiceProxy`
- `OrderRepositoryProxy`

프록시 클래스의 구조는 대부분 비슷하다.

차이는 실제 호출 대상과 메서드 이름 정도이다.

이런 문제를 해결하기 위해 동적 프록시 기술을 사용할 수 있다.

---

## 6. 리플렉션

JDK 동적 프록시를 이해하려면 리플렉션을 알아야 한다.

리플렉션은 클래스나 메서드의 메타정보를 런타임에 조회하고 실행할 수 있는 기능이다.

예를 들어 다음처럼 메서드 정보를 가져올 수 있다.

```java
Class<?> classHello = Class.forName("hello.proxy.Hello");
Method method = classHello.getMethod("callA");
```

그리고 메서드를 동적으로 호출할 수 있다.

```java
Object result = method.invoke(target);
```

리플렉션을 사용하면 호출할 메서드를 코드에 직접 고정하지 않고, 런타임에 결정할 수 있다.

다만 리플렉션은 컴파일 시점에 오류를 잡기 어렵고, 일반 메서드 호출보다 이해하기 어렵다.

따라서 꼭 필요한 곳에서 제한적으로 사용하는 것이 좋다.

---

## 7. JDK 동적 프록시

JDK 동적 프록시는 Java가 기본으로 제공하는 동적 프록시 기술이다.

인터페이스를 기반으로 프록시를 생성한다.

핵심은 `InvocationHandler`이다.

```java
public class LogTraceBasicHandler implements InvocationHandler {

    private final Object target;
    private final LogTrace logTrace;

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        TraceStatus status = logTrace.begin(method.getName());
        try {
            Object result = method.invoke(target, args);
            logTrace.end(status);
            return result;
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

JDK 동적 프록시를 사용하면 인터페이스가 있는 대상에 대해 프록시 클래스를 직접 만들지 않아도 된다.

하지만 인터페이스가 반드시 필요하다는 한계가 있다.

---

## 8. CGLIB

CGLIB는 구체 클래스를 상속해서 프록시를 생성하는 기술이다.

인터페이스가 없어도 프록시를 만들 수 있다.

CGLIB는 대상 클래스를 상속한 하위 클래스를 동적으로 만들어서 부가 기능을 적용한다.

스프링은 CGLIB를 내부에 포함하고 있어 별도의 라이브러리 없이 사용할 수 있다.

CGLIB의 한계는 상속 기반이라는 점이다.

- `final` 클래스는 프록시를 만들 수 없다.
- `final` 메서드는 오버라이딩할 수 없어 프록시 적용이 어렵다.

---

## 9. JDK 동적 프록시와 CGLIB 비교

## JDK 동적 프록시

- 인터페이스 기반이다.
- Java 기본 기능이다.
- 인터페이스가 필요하다.
- 프록시를 구체 클래스 타입으로 캐스팅할 수 없다.

## CGLIB

- 구체 클래스 기반이다.
- 상속을 사용한다.
- 인터페이스가 없어도 된다.
- `final` 클래스와 `final` 메서드에 제약이 있다.

직접 이 두 기술을 상황에 따라 선택하고 관리하면 복잡하다.

스프링은 이 문제를 `ProxyFactory`로 해결한다.

---

## 10. ProxyFactory

`ProxyFactory`는 스프링이 제공하는 프록시 생성 추상화 기술이다.

개발자는 JDK 동적 프록시와 CGLIB를 직접 구분해서 사용하지 않아도 된다.

스프링의 프록시 팩토리는 기본적으로 다음 방식으로 동작한다.

- 인터페이스가 있으면 JDK 동적 프록시 사용
- 인터페이스가 없으면 CGLIB 사용
- 설정에 따라 CGLIB 강제 사용 가능

`ProxyFactory`를 사용하면 프록시 기술 선택을 스프링이 처리해준다.

개발자는 부가 기능을 `Advice`로 작성하면 된다.

---

## 11. Advice

`Advice`는 프록시가 실행할 부가 기능 로직이다.

JDK 동적 프록시의 `InvocationHandler`, CGLIB의 `MethodInterceptor`를 각각 만들 필요 없이 스프링의 `Advice`를 작성하면 된다.

대표적으로 `MethodInterceptor`를 사용할 수 있다.

```java
public class TimeAdvice implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        long startTime = System.currentTimeMillis();

        Object result = invocation.proceed();

        long endTime = System.currentTimeMillis();
        System.out.println("time=" + (endTime - startTime));
        return result;
    }
}
```

`invocation.proceed()`를 호출하면 실제 대상 메서드가 실행된다.

---

## 12. Pointcut

`Pointcut`은 부가 기능을 적용할지 말지 판단하는 조건이다.

모든 메서드에 로그를 적용하고 싶지 않을 수 있다.

예를 들어 `noLog()` 같은 메서드는 제외하고 싶을 수 있다.

이때 포인트컷이 필요하다.

Pointcut은 크게 두 가지를 판단한다.

- 클래스가 적용 대상인지
- 메서드가 적용 대상인지

즉, Pointcut은 “어디에 적용할 것인가”를 담당한다.

---

## 13. Advisor

`Advisor`는 `Pointcut`과 `Advice`를 하나로 묶은 것이다.

```text
Advisor = Pointcut + Advice
```

- Pointcut: 어디에 적용할지 결정
- Advice: 어떤 부가 기능을 적용할지 결정

스프링 AOP의 핵심 구조는 Advisor를 이해하면 훨씬 쉬워진다.

프록시 팩토리는 Advisor 정보를 보고 특정 메서드 호출에 Advice를 적용할지 판단한다.

---

## 14. 여러 Advisor 적용

하나의 프록시에 여러 Advisor를 적용할 수 있다.

예를 들어 다음과 같은 부가 기능을 함께 적용할 수 있다.

- 로그 추적 Advisor
- 시간 측정 Advisor
- 권한 체크 Advisor

여러 Advisor가 적용될 때는 순서도 중요하다.

스프링은 여러 Advisor를 하나의 프록시에 적용할 수 있도록 지원한다.

---

## 15. 빈 후처리기

빈 후처리기(`BeanPostProcessor`)는 스프링이 빈 객체를 생성한 뒤, 빈 저장소에 등록하기 직전에 객체를 조작할 수 있는 기능이다.

빈 등록 흐름은 다음과 같다.

1. 스프링이 빈 대상 객체를 생성한다.
2. 생성된 객체를 빈 후처리기에 전달한다.
3. 빈 후처리기가 객체를 조작하거나 다른 객체로 바꾼다.
4. 반환된 객체가 스프링 빈 저장소에 등록된다.

빈 후처리기는 객체를 그대로 반환할 수도 있고, 프록시 객체로 바꿔서 반환할 수도 있다.

이 기능 덕분에 스프링 빈을 자동으로 프록시로 교체할 수 있다.

---

## 16. 자동 프록시 생성기

스프링은 자동 프록시 생성기를 제공한다.

자동 프록시 생성기는 스프링 빈으로 등록된 Advisor를 찾아서, 포인트컷 조건에 맞는 빈을 자동으로 프록시로 만들어준다.

흐름은 다음과 같다.

1. 스프링이 빈 객체를 생성한다.
2. 자동 프록시 생성기가 Advisor를 조회한다.
3. Advisor의 Pointcut으로 해당 빈이 적용 대상인지 확인한다.
4. 적용 대상이면 프록시를 생성한다.
5. 프록시가 스프링 빈으로 등록된다.

이 구조 덕분에 개발자가 각 빈마다 직접 프록시를 등록하지 않아도 된다.

---

## 17. @Aspect 기반 프록시

Advisor를 직접 만들 수도 있지만, 스프링은 `@Aspect`를 통해 더 편리하게 AOP를 적용할 수 있게 해준다.

`@Aspect`는 포인트컷과 어드바이스를 애노테이션으로 선언하는 방식이다.

```java
@Aspect
public class LogTraceAspect {

    @Around("execution(* hello.proxy.app..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        TraceStatus status = logTrace.begin(joinPoint.getSignature().toShortString());

        try {
            Object result = joinPoint.proceed();
            logTrace.end(status);
            return result;
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

`@Around`의 값에는 포인트컷 표현식을 작성한다.

`@Around`가 붙은 메서드는 Advice가 된다.

스프링은 `@Aspect` 정보를 읽어서 Advisor로 변환하고, 자동 프록시 생성기를 통해 프록시를 적용한다.

---

## 18. 오늘 정리

오늘 학습한 프록시 흐름은 다음과 같이 발전한다.

1. 직접 프록시 클래스 작성
2. JDK 동적 프록시
3. CGLIB
4. ProxyFactory
5. Advisor, Advice, Pointcut
6. BeanPostProcessor
7. 자동 프록시 생성기
8. `@Aspect`

처음에는 부가 기능을 적용하기 위해 직접 프록시를 만들었지만, 점점 스프링이 제공하는 추상화와 자동화 기능을 활용하는 방향으로 발전한다.

핵심 정리는 다음과 같다.

- 프록시는 기존 코드를 변경하지 않고 부가 기능을 적용할 수 있게 해준다.
- JDK 동적 프록시는 인터페이스 기반이다.
- CGLIB는 구체 클래스 기반이다.
- ProxyFactory는 두 프록시 기술을 통합해서 사용할 수 있게 해준다.
- Advice는 부가 기능, Pointcut은 적용 조건, Advisor는 둘을 묶은 것이다.
- BeanPostProcessor는 빈 등록 직전에 객체를 바꿔치기할 수 있다.
- 자동 프록시 생성기는 Advisor를 기준으로 프록시 적용 대상을 자동으로 찾는다.
- `@Aspect`는 포인트컷과 어드바이스를 편리하게 작성하는 방법이다.

앞으로 AOP를 이해할 때는 `@Aspect`만 외우지 말고, 그 아래에 ProxyFactory, Advisor, BeanPostProcessor, 자동 프록시 생성기가 있다는 흐름을 함께 기억해야겠다.
