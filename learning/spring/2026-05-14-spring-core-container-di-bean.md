# 스프링 핵심 원리 정리 - 객체지향 설계, 컨테이너, DI, 빈 관리

## 학습 날짜

2026-05-14

## 학습 주제

- 객체지향 설계와 스프링
- SOLID
- 회원/주문/할인 예제
- 관심사의 분리
- AppConfig
- IoC, DI, 컨테이너
- 스프링 컨테이너와 스프링 빈
- 싱글톤 컨테이너
- 컴포넌트 스캔
- 의존관계 자동 주입
- 빈 생명주기 콜백
- 빈 스코프

---

## 1. 스프링 핵심 원리의 중심

스프링 핵심 원리는 단순히 애노테이션 사용법을 외우는 과정이 아니다.

중심은 객체지향 설계이다.

스프링은 좋은 객체지향 설계를 더 편하게 적용할 수 있도록 도와준다.

특히 중요한 개념은 다음과 같다.

- 역할과 구현의 분리
- 다형성
- OCP
- DIP
- IoC
- DI
- 스프링 컨테이너

객체지향 설계를 코드로 직접 적용하다 보면 객체 생성과 의존관계 연결을 누군가 담당해야 한다.

스프링은 이 역할을 컨테이너와 DI 기능으로 지원한다.

---

## 2. 객체지향 설계와 스프링

객체지향의 핵심은 다형성이다.

다형성을 사용하면 역할과 구현을 분리할 수 있다.

예를 들어 할인 정책을 다음과 같이 나눌 수 있다.

```text
DiscountPolicy 인터페이스
 ├─ FixDiscountPolicy
 └─ RateDiscountPolicy
```

클라이언트는 `DiscountPolicy`라는 역할에 의존하고, 실제 구현체는 고정 할인 정책이 될 수도 있고 정률 할인 정책이 될 수도 있다.

하지만 다형성만 사용한다고 해서 OCP와 DIP가 자동으로 지켜지는 것은 아니다.

클라이언트 코드가 직접 구현체를 선택하면 구현체 변경 시 클라이언트 코드도 수정해야 한다.

```java
private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
```

위 코드는 `DiscountPolicy` 인터페이스에도 의존하지만, 동시에 `FixDiscountPolicy` 구현체에도 의존한다.

따라서 DIP를 위반하게 된다.

---

## 3. OCP와 DIP

## OCP

OCP는 Open-Closed Principle의 약자이다.

확장에는 열려 있고, 변경에는 닫혀 있어야 한다는 원칙이다.

새로운 기능을 추가할 때 기존 코드를 최대한 변경하지 않고 확장할 수 있어야 한다.

## DIP

DIP는 Dependency Inversion Principle의 약자이다.

구체적인 구현 클래스에 의존하지 말고, 추상화에 의존하라는 원칙이다.

쉽게 말하면 구현 클래스가 아니라 인터페이스에 의존해야 한다.

하지만 인터페이스에 의존하더라도 클라이언트가 직접 구현체를 생성하면 DIP를 완전히 지키기 어렵다.

그래서 객체 생성과 의존관계 연결을 외부로 분리해야 한다.

---

## 4. 관심사의 분리

객체는 자신의 역할에 집중해야 한다.

서비스 객체가 비즈니스 로직도 처리하고, 어떤 구현체를 사용할지도 직접 결정하면 책임이 많아진다.

예를 들어 `OrderServiceImpl`이 주문 생성 로직도 처리하고, 할인 정책 구현체도 직접 선택한다면 관심사가 섞인다.

이를 분리하기 위해 설정 클래스인 `AppConfig`를 둘 수 있다.

`AppConfig`는 객체를 생성하고 의존관계를 연결하는 역할을 담당한다.

서비스 객체는 필요한 인터페이스에만 의존하고, 어떤 구현체가 들어올지는 알지 않아도 된다.

---

## 5. AppConfig

`AppConfig`는 애플리케이션의 전체 동작 방식을 구성하는 설정 클래스이다.

```java
public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
```

이 구조에서는 `MemberServiceImpl`, `OrderServiceImpl`이 구현체를 직접 생성하지 않는다.

필요한 의존 객체는 외부에서 주입받는다.

이를 통해 클라이언트 코드를 변경하지 않고 구현체를 교체할 수 있다.

---

## 6. IoC와 DI

## IoC

IoC는 Inversion of Control의 약자이다.

제어의 역전이라고 부른다.

기존에는 객체가 직접 필요한 객체를 생성하고 제어했다.

하지만 IoC 구조에서는 객체 생성과 의존관계 연결을 외부에서 담당한다.

즉, 객체는 자신의 로직을 실행하는 데 집중하고, 전체 제어 흐름은 외부 설정이나 컨테이너가 담당한다.

## DI

DI는 Dependency Injection의 약자이다.

의존관계 주입이라고 한다.

객체가 직접 의존 객체를 생성하지 않고, 외부에서 생성된 객체를 주입받는 방식이다.

```java
public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
```

DI를 사용하면 정적인 클래스 의존관계는 유지하면서, 실행 시점의 객체 인스턴스 의존관계를 유연하게 바꿀 수 있다.

---

## 7. 스프링 컨테이너로 전환

순수 Java로 만든 `AppConfig`를 스프링 기반으로 바꿀 수 있다.

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
```

`@Configuration`은 설정 정보라는 뜻이다.

`@Bean`이 붙은 메서드의 반환 객체는 스프링 컨테이너에 스프링 빈으로 등록된다.

스프링 컨테이너는 다음과 같이 생성할 수 있다.

```java
ApplicationContext applicationContext =
        new AnnotationConfigApplicationContext(AppConfig.class);
```

`ApplicationContext`를 일반적으로 스프링 컨테이너라고 부른다.

---

## 8. 스프링 빈

스프링 빈은 스프링 컨테이너가 관리하는 객체이다.

`@Bean`이 붙은 메서드의 이름이 기본 빈 이름이 된다.

예를 들어 다음 메서드는 `memberService`라는 이름의 빈으로 등록된다.

```java
@Bean
public MemberService memberService() {
    return new MemberServiceImpl(memberRepository());
}
```

빈은 다음처럼 조회할 수 있다.

```java
MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
```

빈 이름은 중복되면 안 된다.

같은 이름을 사용하면 설정에 따라 기존 빈이 덮어씌워지거나 오류가 발생할 수 있다.

---

## 9. BeanFactory와 ApplicationContext

`BeanFactory`는 스프링 컨테이너의 가장 기본 기능을 제공한다.

스프링 빈을 관리하고 조회하는 역할을 한다.

`ApplicationContext`는 `BeanFactory` 기능을 모두 상속받고, 추가로 애플리케이션 개발에 필요한 여러 부가 기능을 제공한다.

예를 들어 다음 기능을 제공한다.

- 메시지 소스
- 환경 변수
- 애플리케이션 이벤트
- 편리한 리소스 조회

실무에서는 `BeanFactory`를 직접 사용하기보다 대부분 `ApplicationContext`를 사용한다.

---

## 10. 싱글톤 컨테이너

웹 애플리케이션은 여러 사용자가 동시에 요청한다.

요청마다 객체를 새로 생성하면 메모리 낭비가 심해질 수 있다.

스프링은 기본적으로 빈을 싱글톤으로 관리한다.

즉, 같은 스프링 빈을 여러 번 조회해도 같은 객체 인스턴스를 반환한다.

```java
MemberService memberService1 = ac.getBean("memberService", MemberService.class);
MemberService memberService2 = ac.getBean("memberService", MemberService.class);

System.out.println(memberService1 == memberService2); // true
```

스프링 컨테이너는 싱글톤 패턴의 단점을 해결하면서 객체를 싱글톤으로 관리해준다.

개발자가 직접 싱글톤 패턴을 구현하지 않아도 된다.

---

## 11. 싱글톤 방식의 주의점

싱글톤 객체는 여러 사용자가 공유한다.

따라서 상태를 가지면 문제가 발생할 수 있다.

특히 사용자별로 달라지는 값을 필드에 저장하면 위험하다.

```java
private int price;
```

여러 요청이 동시에 들어오면 한 사용자의 값이 다른 사용자에게 영향을 줄 수 있다.

싱글톤 빈은 무상태로 설계해야 한다.

무상태 설계 기준은 다음과 같다.

- 특정 클라이언트에 의존적인 필드를 만들지 않는다.
- 특정 클라이언트가 값을 변경할 수 있는 필드를 만들지 않는다.
- 가급적 읽기 전용 값만 필드로 둔다.
- 공유 값이 필요하면 지역 변수, 파라미터, 반환값을 사용한다.

---

## 12. @Configuration과 싱글톤 보장

스프링 설정 클래스에서 `@Configuration`을 사용하면 스프링이 싱글톤을 보장하기 위해 설정 클래스를 특별하게 처리한다.

예를 들어 `memberRepository()`를 여러 번 호출하는 것처럼 보여도 실제로는 같은 스프링 빈 인스턴스가 반환되도록 보장한다.

스프링은 설정 클래스를 바이트코드 조작을 통해 CGLIB 기반의 프록시 클래스로 만들어 관리한다.

따라서 `@Bean` 메서드가 여러 번 호출되어도 스프링 컨테이너에 등록된 동일한 빈을 반환하도록 동작한다.

---

## 13. 컴포넌트 스캔

수동으로 `@Bean`을 하나씩 등록하면 빈이 많아질수록 설정 정보가 커지고 누락 가능성이 생긴다.

이를 해결하기 위해 스프링은 컴포넌트 스캔 기능을 제공한다.

```java
@Configuration
@ComponentScan
public class AutoAppConfig {
}
```

컴포넌트 스캔은 `@Component`가 붙은 클래스를 찾아 스프링 빈으로 자동 등록한다.

다음 애노테이션들도 내부적으로 `@Component`를 포함한다.

- `@Controller`
- `@Service`
- `@Repository`
- `@Configuration`

따라서 해당 애노테이션이 붙은 클래스도 컴포넌트 스캔 대상이 된다.

---

## 14. 컴포넌트 스캔 탐색 위치

컴포넌트 스캔은 지정한 위치부터 하위 패키지를 탐색한다.

스프링 부트를 사용하면 보통 메인 애플리케이션 클래스가 있는 패키지를 기준으로 하위 패키지를 스캔한다.

따라서 메인 클래스는 프로젝트 최상위 패키지에 두는 것이 좋다.

예를 들어 다음 구조가 자연스럽다.

```text
hello.core
 ├─ CoreApplication
 ├─ member
 ├─ order
 └─ discount
```

이렇게 하면 `hello.core` 하위의 컴포넌트들이 자동으로 스캔된다.

---

## 15. 의존관계 자동 주입

컴포넌트 스캔으로 빈을 자동 등록하면 의존관계도 자동으로 주입할 수 있다.

대표적으로 `@Autowired`를 사용한다.

```java
@Component
public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;

    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

`@Autowired`는 기본적으로 타입을 기준으로 빈을 조회해서 주입한다.

생성자 파라미터가 여러 개여도 스프링이 타입에 맞는 빈을 찾아 주입한다.

---

## 16. 의존관계 주입 방법

의존관계 주입 방법은 크게 네 가지가 있다.

- 생성자 주입
- 수정자 주입
- 필드 주입
- 일반 메서드 주입

## 생성자 주입

생성자를 통해 의존관계를 주입받는다.

```java
public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
```

특징은 다음과 같다.

- 생성 시점에 한 번만 호출된다.
- 필수 의존관계에 적합하다.
- `final` 필드를 사용할 수 있다.
- 불변 설계에 유리하다.
- 테스트하기 쉽다.

생성자가 하나만 있으면 `@Autowired`를 생략할 수 있다.

## 수정자 주입

setter 메서드를 통해 의존관계를 주입한다.

선택적이거나 변경 가능성이 있는 의존관계에 사용할 수 있다.

## 필드 주입

필드에 바로 `@Autowired`를 붙이는 방식이다.

```java
@Autowired
private MemberRepository memberRepository;
```

코드는 간결하지만 외부에서 변경하기 어렵고, 순수 Java 테스트가 어려워진다.

실제 애플리케이션 코드에서는 권장되지 않는다.

---

## 17. 생성자 주입을 선택해야 하는 이유

실무에서는 대부분 생성자 주입을 권장한다.

이유는 다음과 같다.

- 의존관계가 누락되는 것을 막을 수 있다.
- 객체 생성 시점에 필요한 의존관계가 모두 주입된다.
- `final` 키워드를 사용할 수 있어 불변성을 보장할 수 있다.
- 테스트 코드에서 직접 객체를 생성하기 쉽다.
- 순환 참조 문제를 더 빨리 발견할 수 있다.

따라서 기본적으로 생성자 주입을 사용하고, 특별한 이유가 있을 때만 다른 방식을 고려하는 것이 좋다.

---

## 18. 조회 빈이 2개 이상일 때

`@Autowired`는 타입으로 빈을 조회한다.

그런데 같은 타입의 빈이 2개 이상 있으면 어떤 빈을 주입해야 할지 알 수 없어 오류가 발생한다.

예를 들어 `DiscountPolicy` 구현체가 두 개일 수 있다.

- `FixDiscountPolicy`
- `RateDiscountPolicy`

이때 해결 방법은 다음과 같다.

- 필드명 또는 파라미터명 매칭
- `@Qualifier`
- `@Primary`

## @Qualifier

특정 빈을 명시적으로 지정할 때 사용한다.

```java
public OrderServiceImpl(@Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
    this.discountPolicy = discountPolicy;
}
```

## @Primary

기본 우선순위를 지정할 때 사용한다.

```java
@Primary
@Component
public class RateDiscountPolicy implements DiscountPolicy {
}
```

일반적으로 기본값은 `@Primary`로 처리하고, 특별히 다른 빈이 필요할 때 `@Qualifier`를 사용할 수 있다.

---

## 19. 자동 등록과 수동 등록 기준

실무에서는 보통 다음 기준으로 나눌 수 있다.

## 자동 등록

정형화된 계층 구조에 사용한다.

- Controller
- Service
- Repository

이런 클래스들은 개수가 많고 패턴이 일정하므로 컴포넌트 스캔이 편리하다.

## 수동 등록

기술 지원 객체나 설정이 중요한 객체에 사용한다.

예를 들어 다음과 같은 경우 수동 등록을 고려할 수 있다.

- 외부 라이브러리 객체
- 설정 값이 중요한 객체
- 같은 타입 구현체가 여러 개이고 명확히 구분해야 하는 경우
- 비즈니스 로직이 아니라 기술 지원 로직인 경우

---

## 20. 빈 생명주기 콜백

스프링 빈은 생성되고, 의존관계가 주입되고, 초기화된 뒤 사용된다.

종료 시점에는 필요한 정리 작업을 수행할 수 있다.

스프링 빈의 생명주기 흐름은 다음과 같다.

```text
스프링 컨테이너 생성
→ 스프링 빈 생성
→ 의존관계 주입
→ 초기화 콜백
→ 사용
→ 소멸 전 콜백
→ 스프링 종료
```

초기화 콜백은 의존관계 주입이 끝난 후 호출된다.

소멸 전 콜백은 빈이 종료되기 직전에 호출된다.

---

## 21. 생성자와 초기화 분리

생성자는 객체를 생성하고 필수 값을 받는 책임을 가진다.

반면 초기화는 외부 커넥션 연결처럼 무거운 작업을 수행할 수 있다.

따라서 생성자 안에서 무거운 초기화 작업까지 함께 수행하기보다, 생성과 초기화를 분리하는 것이 좋다.

예를 들어 DB 커넥션 풀이나 네트워크 소켓 연결은 의존관계 주입이 완료된 뒤 초기화 콜백에서 수행하는 것이 자연스럽다.

---

## 22. @PostConstruct와 @PreDestroy

빈 생명주기 콜백을 처리하는 가장 권장되는 방법은 애노테이션을 사용하는 것이다.

```java
@PostConstruct
public void init() {
    connect();
}

@PreDestroy
public void close() {
    disconnect();
}
```

`@PostConstruct`는 의존관계 주입이 끝난 뒤 초기화 시점에 호출된다.

`@PreDestroy`는 컨테이너가 종료되기 직전에 호출된다.

이 방식은 스프링에만 종속적인 인터페이스를 구현하지 않아도 되어 편리하다.

---

## 23. 빈 스코프

빈 스코프는 빈이 존재할 수 있는 범위를 의미한다.

스프링은 여러 스코프를 제공한다.

- singleton
- prototype
- request
- session
- application

## 싱글톤 스코프

스프링의 기본 스코프이다.

스프링 컨테이너의 시작부터 종료까지 하나의 인스턴스가 유지된다.

## 프로토타입 스코프

스프링 컨테이너에 요청할 때마다 새로운 인스턴스를 생성한다.

하지만 스프링 컨테이너는 생성, 의존관계 주입, 초기화까지만 관리하고 이후에는 관리하지 않는다.

따라서 종료 메서드도 자동으로 호출되지 않는다.

## request 스코프

웹 요청이 들어오고 나갈 때까지 유지되는 스코프이다.

각 HTTP 요청마다 별도의 빈 인스턴스가 생성된다.

---

## 24. 프로토타입 스코프 주의점

프로토타입 빈은 요청할 때마다 새로 생성된다.

하지만 싱글톤 빈이 프로토타입 빈을 주입받으면 문제가 생길 수 있다.

싱글톤 빈은 한 번만 생성되므로, 생성 시점에 주입받은 프로토타입 빈을 계속 재사용하게 된다.

즉, 프로토타입 빈을 매번 새로 사용하고 싶어도 실제로는 같은 객체를 계속 사용할 수 있다.

이를 해결하려면 필요한 시점마다 프로토타입 빈을 조회해야 한다.

대표적인 방법은 다음과 같다.

- `ObjectProvider`
- `ObjectFactory`
- JSR-330 `Provider`

---

## 25. 웹 스코프와 프록시

request 스코프는 HTTP 요청이 있어야 생성된다.

그런데 싱글톤 빈이 request 스코프 빈을 바로 주입받으려고 하면 애플리케이션 시작 시점에는 request가 없기 때문에 문제가 생길 수 있다.

이때 Provider나 프록시를 사용할 수 있다.

프록시를 사용하면 실제 request 스코프 빈 대신 가짜 프록시 객체를 싱글톤 빈에 주입해 둔다.

실제 요청이 들어왔을 때 프록시가 진짜 request 스코프 빈을 찾아서 위임한다.

---

## 26. 오늘 정리

스프링 핵심 원리에서 가장 중요한 흐름은 다음과 같다.

- 객체지향 설계에서는 역할과 구현을 분리해야 한다.
- 다형성만으로는 OCP와 DIP를 완전히 지키기 어렵다.
- 객체 생성과 의존관계 연결은 외부 설정으로 분리해야 한다.
- AppConfig는 순수 Java 기반 DI 컨테이너 역할을 한다.
- 스프링 컨테이너는 AppConfig 역할을 더 강력하게 지원한다.
- 스프링 빈은 컨테이너가 관리하는 객체이다.
- 기본 빈 스코프는 싱글톤이다.
- 싱글톤 빈은 무상태로 설계해야 한다.
- 컴포넌트 스캔은 빈 등록을 자동화한다.
- 의존관계 자동 주입은 생성자 주입을 기본으로 생각하는 것이 좋다.
- 빈 생명주기는 초기화와 종료 작업을 안전하게 처리하기 위해 필요하다.
- 빈 스코프는 빈이 유지되는 범위를 결정한다.

스프링을 이해하려면 단순히 애노테이션을 외우는 것보다, 왜 이런 기능이 필요한지 객체지향 설계 문제에서 출발해 이해하는 것이 중요하다고 느꼈다.
