# Spring AOP 개념, 구현, 포인트컷, 실무 주의사항 정리

## 학습 날짜

2026-05-19

## 학습 주제

- AOP 개념
- 핵심 기능과 부가 기능
- 애스펙트
- 조인 포인트
- 포인트컷
- 어드바이스
- 어드바이저
- `@Aspect`
- `@Around`
- 포인트컷 표현식
- AOP 실전 예제
- 프록시 내부 호출 문제
- JDK 동적 프록시와 CGLIB 한계

---

## 1. AOP가 필요한 이유

애플리케이션 로직은 핵심 기능과 부가 기능으로 나눌 수 있다.

핵심 기능은 해당 객체가 제공해야 하는 고유한 기능이다.

예를 들어 `OrderService`의 핵심 기능은 주문 로직이다.

부가 기능은 핵심 기능을 보조하는 공통 기능이다.

예를 들어 다음과 같은 기능이 있다.

- 로그 추적
- 트랜잭션
- 권한 체크
- 성능 측정
- 재시도

문제는 이런 부가 기능이 여러 계층에 반복해서 필요하다는 점이다.

부가 기능을 각 클래스에 직접 넣으면 핵심 기능과 부가 기능이 섞이고, 중복 코드가 많아진다.

AOP는 이런 부가 기능을 별도의 관점으로 분리해서 적용하는 방법이다.

---

## 2. 애스펙트

애스펙트는 부가 기능과 그 적용 위치를 함께 모듈화한 것이다.

쉽게 말하면 다음 두 가지를 하나로 묶은 것이다.

- 어디에 적용할 것인가
- 무엇을 적용할 것인가

스프링 AOP에서는 이것을 다음 개념으로 나누어 볼 수 있다.

- 포인트컷: 어디에 적용할지
- 어드바이스: 무엇을 적용할지

즉, 애스펙트는 포인트컷과 어드바이스를 포함한 부가 기능 모듈이라고 이해할 수 있다.

---

## 3. AOP 주요 용어

## 조인 포인트

조인 포인트는 AOP를 적용할 수 있는 지점이다.

스프링 AOP에서는 프록시 방식으로 동작하기 때문에 메서드 실행 지점이 조인 포인트가 된다.

즉, 스프링 AOP에서 조인 포인트는 메서드 실행이라고 이해하면 된다.

## 포인트컷

포인트컷은 조인 포인트 중에서 실제로 AOP를 적용할 대상을 선별하는 조건이다.

예를 들어 다음 표현식은 특정 패키지 하위의 모든 메서드 실행을 대상으로 한다.

```java
execution(* hello.aop.order..*(..))
```

## 어드바이스

어드바이스는 실제 부가 기능 로직이다.

예를 들어 로그 출력, 실행 시간 측정, 트랜잭션 시작과 종료 같은 코드가 어드바이스이다.

## 어드바이저

어드바이저는 포인트컷과 어드바이스를 하나로 묶은 것이다.

```text
Advisor = Pointcut + Advice
```

## 타겟

타겟은 AOP가 적용되는 실제 대상 객체이다.

## 프록시

프록시는 타겟을 대신해서 클라이언트 요청을 먼저 받는 객체이다.

스프링 AOP는 프록시를 통해 어드바이스를 실행한 뒤 타겟을 호출한다.

---

## 4. 스프링 AOP 적용 방식

스프링 AOP는 기본적으로 프록시 방식으로 동작한다.

흐름은 다음과 같다.

```text
클라이언트 → AOP 프록시 → 어드바이스 실행 → 실제 대상 객체 호출
```

클라이언트는 실제 대상 객체를 직접 호출한다고 생각하지만, 실제로는 스프링이 만든 프록시 객체를 호출한다.

프록시는 포인트컷 조건에 맞으면 어드바이스를 실행하고, 이후 실제 대상 메서드를 호출한다.

이 방식 덕분에 핵심 기능 코드를 수정하지 않고 부가 기능을 적용할 수 있다.

---

## 5. @Aspect

스프링은 `@Aspect` 애노테이션을 사용해 AOP를 편리하게 구현할 수 있게 해준다.

```java
@Slf4j
@Aspect
public class AspectV1 {

    @Around("execution(* hello.aop.order..*(..))")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }
}
```

`@Aspect`가 붙은 클래스는 AOP 설정 정보를 담는다.

`@Around`는 포인트컷과 어드바이스를 함께 표현한다.

`@Around`의 값은 포인트컷 표현식이고, 메서드 본문은 어드바이스가 된다.

---

## 6. ProceedingJoinPoint

`ProceedingJoinPoint`는 실제 대상 메서드를 호출할 수 있는 객체이다.

```java
Object result = joinPoint.proceed();
```

`proceed()`를 호출해야 실제 대상 메서드가 실행된다.

`ProceedingJoinPoint`에서는 다음 정보도 얻을 수 있다.

- 호출 대상 메서드 정보
- 전달 인자
- 실제 target 객체
- 메서드 시그니처

예를 들어 다음과 같이 사용할 수 있다.

```java
String signature = joinPoint.getSignature().toShortString();
Object[] args = joinPoint.getArgs();
```

---

## 7. 어드바이스 종류

스프링 AOP에는 여러 어드바이스 종류가 있다.

## @Around

메서드 실행 전후를 모두 감싼다.

가장 강력한 어드바이스이며, 직접 `proceed()`를 호출해야 한다.

```java
@Around("pointcut()")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    // 메서드 호출 전
    Object result = joinPoint.proceed();
    // 메서드 호출 후
    return result;
}
```

## @Before

메서드 실행 전에 호출된다.

```java
@Before("pointcut()")
public void before(JoinPoint joinPoint) {
}
```

## @AfterReturning

메서드가 정상 반환된 후 호출된다.

## @AfterThrowing

메서드에서 예외가 발생한 후 호출된다.

## @After

메서드가 정상 종료되든 예외가 발생하든 항상 호출된다.

실무에서는 대부분 `@Around`를 많이 사용하지만, 목적이 명확하면 다른 어드바이스를 사용해 의도를 더 잘 드러낼 수 있다.

---

## 8. 포인트컷 분리

포인트컷 표현식이 여러 어드바이스에서 반복되면 별도로 분리할 수 있다.

```java
@Pointcut("execution(* hello.aop.order..*(..))")
private void allOrder() {
}
```

분리한 포인트컷은 다음처럼 사용할 수 있다.

```java
@Around("allOrder()")
public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
    return joinPoint.proceed();
}
```

포인트컷을 분리하면 재사용할 수 있고, 표현식의 의미를 이름으로 드러낼 수 있다.

---

## 9. 어드바이스 순서

여러 AOP가 같은 대상에 적용되면 실행 순서가 중요할 수 있다.

스프링에서는 `@Order`를 사용해 순서를 지정할 수 있다.

```java
@Order(1)
@Aspect
public class TransactionAspect {
}
```

숫자가 작을수록 먼저 실행된다.

순서가 필요한 경우에는 하나의 Aspect 클래스 안에서 내부 클래스로 분리하거나, Aspect 자체에 `@Order`를 적용해 관리할 수 있다.

---

## 10. 포인트컷 표현식

포인트컷 표현식은 AOP를 적용할 대상을 지정하는 조건이다.

스프링 AOP에서 가장 많이 사용하는 것은 `execution`이다.

```java
execution(* hello.aop.order..*(..))
```

이 표현식은 다음처럼 이해할 수 있다.

- `*`: 모든 반환 타입
- `hello.aop.order..`: 해당 패키지와 하위 패키지
- `*`: 모든 클래스 또는 메서드 이름
- `(..)`: 모든 파라미터

포인트컷 표현식은 처음에는 어렵지만, 메서드 실행 정보를 조건으로 매칭한다고 생각하면 된다.

---

## 11. 주요 포인트컷 지시자

## execution

메서드 실행 조인 포인트를 매칭한다.

스프링 AOP에서 가장 많이 사용한다.

## within

특정 타입 내부의 조인 포인트를 매칭한다.

## args

전달된 인자의 런타임 타입을 기준으로 매칭한다.

## this

스프링 AOP 프록시 객체를 기준으로 매칭한다.

## target

프록시가 가리키는 실제 대상 객체를 기준으로 매칭한다.

## @annotation

특정 애노테이션이 붙은 메서드를 매칭한다.

## @target, @within

특정 애노테이션이 타입에 붙어 있는 경우 매칭한다.

## bean

스프링 빈 이름을 기준으로 매칭한다.

처음에는 `execution`과 `@annotation`을 중심으로 익히는 것이 좋다.

---

## 12. @annotation 활용

특정 애노테이션이 붙은 메서드에만 AOP를 적용할 수 있다.

예를 들어 로그 추적용 애노테이션을 만들 수 있다.

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Trace {
}
```

그리고 포인트컷에 `@annotation`을 사용한다.

```java
@Before("@annotation(hello.aop.exam.annotation.Trace)")
public void doTrace(JoinPoint joinPoint) {
    log.info("[trace] {}", joinPoint.getSignature());
}
```

이렇게 하면 `@Trace`가 붙은 메서드에만 로그가 출력된다.

애노테이션 기반 AOP는 적용 대상을 코드에서 명확하게 표시할 수 있다는 장점이 있다.

---

## 13. 재시도 AOP

실전 예제에서는 `@Retry` 애노테이션을 사용해 예외 발생 시 재시도하는 AOP를 만들 수 있다.

간헐적으로 실패하는 저장소가 있을 때 재시도 로직을 매번 직접 작성하면 중복이 생긴다.

재시도 AOP를 사용하면 다음처럼 표현할 수 있다.

```java
@Retry(value = 4)
public void request(String itemId) {
    repository.save(itemId);
}
```

AOP는 예외가 발생하면 지정한 횟수만큼 다시 시도하고, 계속 실패하면 예외를 던진다.

이런 기능은 네트워크 호출이나 외부 시스템 연동처럼 일시적인 실패가 발생할 수 있는 상황에서 활용할 수 있다.

---

## 14. AOP 내부 호출 문제

스프링 AOP는 프록시 방식으로 동작한다.

따라서 AOP가 적용되려면 반드시 프록시를 통해 대상 객체를 호출해야 한다.

문제는 같은 객체 내부에서 메서드를 호출하는 경우이다.

```java
public void external() {
    internal();
}

public void internal() {
}
```

`external()`을 외부에서 호출하면 프록시를 거치므로 AOP가 적용된다.

하지만 `external()` 내부에서 호출하는 `internal()`은 `this.internal()`과 같다.

여기서 `this`는 프록시가 아니라 실제 대상 객체이다.

따라서 내부 호출은 프록시를 거치지 않고, `internal()`에는 AOP가 적용되지 않는다.

---

## 15. 내부 호출 문제 해결 방법

## 자기 자신 주입

자기 자신을 주입받아 프록시를 통해 호출하는 방법이다.

```java
@Autowired
public void setCallService(CallService callService) {
    this.callService = callService;
}
```

단, 순환 참조 문제가 생길 수 있고 구조가 어색해질 수 있다.

## 지연 조회

`ObjectProvider`를 사용해 실제 호출 시점에 스프링 빈을 조회할 수 있다.

```java
private final ObjectProvider<CallService> callServiceProvider;

public void external() {
    CallService callService = callServiceProvider.getObject();
    callService.internal();
}
```

자기 자신을 직접 주입받는 것보다는 순환 참조 문제를 피할 수 있다.

## 구조 변경

가장 권장되는 방법은 내부 호출이 발생하지 않도록 구조를 변경하는 것이다.

내부 메서드를 별도 클래스로 분리한다.

```java
public class CallService {
    private final InternalService internalService;

    public void external() {
        internalService.internal();
    }
}
```

이렇게 하면 외부 빈 호출이 되므로 프록시를 거쳐 AOP가 적용될 수 있다.

---

## 16. JDK 동적 프록시 한계

JDK 동적 프록시는 인터페이스 기반으로 프록시를 생성한다.

따라서 인터페이스 타입으로는 주입받을 수 있지만, 구체 클래스 타입으로는 캐스팅할 수 없다.

예를 들어 다음과 같은 문제가 발생할 수 있다.

```java
MemberService memberServiceProxy = (MemberService) proxyFactory.getProxy();
MemberServiceImpl impl = (MemberServiceImpl) memberServiceProxy; // 실패
```

JDK 동적 프록시는 인터페이스를 구현한 프록시이지, 실제 구현 클래스를 상속한 객체가 아니기 때문이다.

따라서 JDK 동적 프록시를 사용할 때는 인터페이스 타입으로 의존관계를 주입받는 것이 중요하다.

---

## 17. CGLIB 한계

CGLIB는 구체 클래스를 상속해서 프록시를 만든다.

따라서 다음과 같은 한계가 있다.

- `final` 클래스는 상속할 수 없다.
- `final` 메서드는 오버라이딩할 수 없다.
- 상속 기반이므로 생성자 호출과 관련된 제약이 있다.

스프링은 CGLIB 관련 문제를 많이 개선해왔고, 스프링 부트에서는 기본적으로 CGLIB 기반 프록시를 사용하는 경우가 많다.

하지만 final 제약은 여전히 주의해야 한다.

---

## 18. this와 target

포인트컷에서 `this`와 `target`은 비슷해 보이지만 기준이 다르다.

## this

`this`는 스프링 AOP 프록시 객체를 기준으로 매칭한다.

## target

`target`은 프록시가 가리키는 실제 대상 객체를 기준으로 매칭한다.

JDK 동적 프록시는 인터페이스 기반이므로 구현 클래스 타입 기준의 `this` 매칭이 어려울 수 있다.

CGLIB는 구체 클래스를 상속하므로 구현 클래스 타입 기준의 `this` 매칭이 가능하다.

이 차이는 프록시 기술을 이해할 때 중요하다.

---

## 19. 오늘 정리

오늘 학습한 AOP의 핵심은 핵심 기능과 부가 기능을 분리하는 것이다.

스프링 AOP는 프록시 방식으로 동작하기 때문에 기존 코드를 크게 수정하지 않고도 로그, 트랜잭션, 재시도 같은 부가 기능을 적용할 수 있다.

중요한 정리는 다음과 같다.

- AOP는 부가 기능을 별도의 관점으로 분리하는 방식이다.
- 스프링 AOP는 프록시 방식으로 동작한다.
- `@Aspect`는 포인트컷과 어드바이스를 편리하게 정의하는 방법이다.
- `@Around`는 대상 메서드 실행 전후를 감쌀 수 있다.
- `ProceedingJoinPoint.proceed()`를 호출해야 실제 대상 메서드가 실행된다.
- 포인트컷은 어디에 AOP를 적용할지 결정한다.
- `execution`은 스프링 AOP에서 가장 많이 사용하는 포인트컷 지시자이다.
- `@annotation`을 사용하면 특정 애노테이션이 붙은 메서드에만 AOP를 적용할 수 있다.
- 스프링 AOP는 내부 호출에는 적용되지 않는다.
- 내부 호출 문제는 구조 변경으로 해결하는 것이 가장 권장된다.
- JDK 동적 프록시와 CGLIB는 각각 한계가 있으므로 의존관계 타입과 final 사용에 주의해야 한다.

앞으로 AOP를 사용할 때는 단순히 애노테이션을 붙이는 것에서 끝내지 않고, 프록시를 통해 호출되는 구조인지 반드시 확인해야겠다.
