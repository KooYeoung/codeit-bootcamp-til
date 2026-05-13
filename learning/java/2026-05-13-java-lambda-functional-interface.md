# Java 람다와 함수형 인터페이스 정리

## 학습 날짜

2026-05-13

## 학습 주제

- 람다가 필요한 이유
- 람다식
- 함수형 인터페이스
- 람다와 시그니처
- 람다 생략 규칙
- 람다 전달과 반환
- 고차 함수
- 람다 활용
- 람다와 익명 클래스 차이
- 메서드 참조
- 함수형 프로그래밍 기초

---

## 1. 람다가 필요한 이유

람다는 Java에서 간단한 동작을 값처럼 전달하기 위해 사용한다.

람다가 없을 때는 특정 동작을 전달하기 위해 익명 클래스를 자주 사용했다.

예를 들어 실행할 동작 하나를 전달하고 싶어도 익명 클래스를 사용하면 코드가 길어진다.

```java
Procedure procedure = new Procedure() {
    @Override
    public void run() {
        System.out.println("hello");
    }
};
```

람다를 사용하면 같은 동작을 더 간결하게 표현할 수 있다.

```java
Procedure procedure = () -> System.out.println("hello");
```

핵심은 “동작 자체”를 간단한 문법으로 표현하고 전달할 수 있다는 점이다.

람다를 사용하면 반복되는 익명 클래스 문법을 줄이고, 코드에서 중요한 로직만 더 잘 보이게 만들 수 있다.

---

## 2. 람다란

람다는 이름 없는 함수처럼 표현할 수 있는 문법이다.

기본 형태는 다음과 같다.

```java
(매개변수) -> { 실행문 }
```

예를 들어 숫자 하나를 받아 1을 더하는 람다는 다음과 같이 표현할 수 있다.

```java
(int x) -> { return x + 1; }
```

더 간단하게 생략하면 다음처럼 쓸 수도 있다.

```java
x -> x + 1
```

람다는 “개념”이고, 람다식은 Java에서 람다를 표현하는 구체적인 문법이라고 이해할 수 있다.

실무에서는 두 용어를 엄격히 구분하기보다 보통 람다라고 부르는 경우가 많다.

---

## 3. Java에서 람다는 함수형 인터페이스를 통해 사용한다

Java는 독립적인 함수를 직접 변수에 담는 언어가 아니다.

Java에서 람다는 함수형 인터페이스에 대입해서 사용한다.

함수형 인터페이스는 추상 메서드가 하나만 있는 인터페이스이다.

```java
@FunctionalInterface
public interface MyFunction {
    int apply(int a, int b);
}
```

이 인터페이스에는 추상 메서드가 `apply` 하나만 있다.

따라서 다음처럼 람다를 대입할 수 있다.

```java
MyFunction add = (a, b) -> a + b;
```

람다식 `(a, b) -> a + b`는 `apply(int a, int b)` 메서드를 구현한 것처럼 동작한다.

즉, 람다는 함수형 인터페이스의 추상 메서드 구현을 간결하게 표현한 것이다.

---

## 4. SAM과 @FunctionalInterface

함수형 인터페이스는 단일 추상 메서드를 가져야 한다.

이를 SAM이라고 한다.

SAM은 Single Abstract Method의 약자이다.

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

`@FunctionalInterface`를 붙이면 컴파일러가 이 인터페이스가 함수형 인터페이스 조건을 만족하는지 검사해준다.

만약 추상 메서드를 두 개 이상 만들면 컴파일 오류가 발생한다.

```java
@FunctionalInterface
interface WrongInterface {
    void run();
    void stop(); // 컴파일 오류
}
```

`@FunctionalInterface`는 필수는 아니지만, 실수를 막기 위해 붙이는 것이 좋다.

---

## 5. 람다와 시그니처

람다식은 대입되는 함수형 인터페이스의 추상 메서드 시그니처와 맞아야 한다.

시그니처를 볼 때 중요한 기준은 다음과 같다.

- 매개변수 개수
- 매개변수 타입
- 반환 타입

예를 들어 다음 인터페이스가 있다고 하자.

```java
@FunctionalInterface
interface StringFunction {
    String apply(String value);
}
```

이 인터페이스에는 문자열 하나를 받아 문자열을 반환하는 메서드가 있다.

따라서 다음 람다식은 가능하다.

```java
StringFunction upper = value -> value.toUpperCase();
```

하지만 매개변수 개수나 반환 타입이 맞지 않으면 사용할 수 없다.

람다는 스스로 독립적인 타입을 가지기보다, 어떤 함수형 인터페이스에 대입되느냐에 따라 타입이 결정된다.

이를 타겟 타입이라고 이해할 수 있다.

---

## 6. 람다 생략 규칙

람다는 문맥상 타입을 추론할 수 있으면 여러 부분을 생략할 수 있다.

기본 형태는 다음과 같다.

```java
(int x) -> { return x + 1; }
```

타입을 추론할 수 있으면 타입을 생략할 수 있다.

```java
(x) -> { return x + 1; }
```

매개변수가 하나라면 괄호도 생략할 수 있다.

```java
x -> { return x + 1; }
```

본문이 한 줄이고 바로 값을 반환한다면 중괄호와 `return`도 생략할 수 있다.

```java
x -> x + 1
```

다만 가독성을 해칠 정도로 과하게 생략하는 것은 좋지 않다.

람다는 짧고 명확할 때 가장 읽기 좋다.

---

## 7. 기본 함수형 인터페이스

Java는 자주 사용하는 함수형 인터페이스를 `java.util.function` 패키지에 제공한다.

직접 함수형 인터페이스를 매번 만들기보다, 기본 제공 인터페이스를 활용하면 좋다.

### Predicate<T>

값을 받아 조건을 검사하고 `boolean`을 반환한다.

```java
Predicate<Integer> isEven = n -> n % 2 == 0;
```

주로 필터 조건에 사용한다.

### Function<T, R>

값을 받아 다른 값으로 변환한다.

```java
Function<String, Integer> lengthFunction = s -> s.length();
```

입력 타입과 반환 타입이 다를 수 있다.

### Consumer<T>

값을 받아 소비만 하고 반환하지 않는다.

```java
Consumer<String> printer = s -> System.out.println(s);
```

출력, 저장, 로깅처럼 결과를 반환하지 않는 작업에 사용할 수 있다.

### Supplier<T>

입력 없이 값을 공급한다.

```java
Supplier<Double> randomSupplier = () -> Math.random();
```

필요할 때 값을 만들어내는 용도로 사용한다.

---

## 8. 람다를 변수에 담기

람다는 함수형 인터페이스 타입의 변수에 담을 수 있다.

```java
MyFunction add = (a, b) -> a + b;
```

이때 변수에는 람다 인스턴스의 참조가 들어간다고 이해할 수 있다.

이후 변수명을 통해 람다를 실행할 수 있다.

```java
int result = add.apply(1, 2);
```

즉, 람다를 변수에 담는다는 것은 실행할 동작을 변수로 다룰 수 있다는 의미이다.

---

## 9. 람다를 메서드 인자로 전달하기

람다는 메서드의 인자로 전달할 수 있다.

```java
static int calculate(MyFunction function) {
    return function.apply(10, 20);
}
```

호출하는 쪽에서는 람다식을 직접 전달할 수 있다.

```java
int result = calculate((a, b) -> a + b);
```

이렇게 하면 메서드는 “어떤 계산을 할지”를 외부에서 전달받을 수 있다.

이 방식은 조건, 변환, 계산 로직을 유연하게 바꿀 때 유용하다.

---

## 10. 람다를 반환하기

메서드가 람다를 반환할 수도 있다.

```java
static MyFunction getOperation(String operator) {
    switch (operator) {
        case "add":
            return (a, b) -> a + b;
        case "sub":
            return (a, b) -> a - b;
        default:
            return (a, b) -> 0;
    }
}
```

반환 타입은 함수형 인터페이스 타입이다.

```java
MyFunction add = getOperation("add");
int result = add.apply(1, 2);
```

이처럼 람다는 변수에 담고, 인자로 전달하고, 반환할 수 있다.

---

## 11. 고차 함수

고차 함수는 함수를 값처럼 다루는 함수를 의미한다.

다음 중 하나를 만족하면 고차 함수라고 볼 수 있다.

- 함수를 인자로 받는 함수
- 함수를 반환하는 함수

Java에서는 함수를 직접 전달한다기보다, 함수형 인터페이스를 구현한 람다 인스턴스를 전달한다고 이해하면 된다.

예를 들어 다음 메서드는 람다를 인자로 받는다.

```java
static int calculate(MyFunction function) {
    return function.apply(1, 2);
}
```

다음 메서드는 람다를 반환한다.

```java
static MyFunction getAddFunction() {
    return (a, b) -> a + b;
}
```

고차 함수를 활용하면 동작을 외부에서 주입하거나 조합할 수 있어 코드가 유연해진다.

---

## 12. 람다 활용: filter

람다는 조건을 외부에서 전달할 때 유용하다.

예를 들어 숫자 목록에서 짝수만 고르는 기능과 홀수만 고르는 기능은 조건만 다르고 나머지 흐름은 같다.

```java
static List<Integer> filter(List<Integer> numbers, Predicate<Integer> predicate) {
    List<Integer> result = new ArrayList<>();

    for (Integer number : numbers) {
        if (predicate.test(number)) {
            result.add(number);
        }
    }

    return result;
}
```

사용하는 쪽에서는 조건만 람다로 전달한다.

```java
List<Integer> evenNumbers = filter(numbers, n -> n % 2 == 0);
List<Integer> oddNumbers = filter(numbers, n -> n % 2 == 1);
```

이렇게 하면 필터링 흐름은 하나로 유지하고, 조건만 바꿀 수 있다.

---

## 13. 람다 활용: map

`map`은 값을 다른 값으로 변환하는 작업이다.

예를 들어 문자열 목록을 대문자로 바꾸거나, 학생 목록에서 이름만 뽑을 때 사용할 수 있다.

```java
static <T, R> List<R> map(List<T> list, Function<T, R> mapper) {
    List<R> result = new ArrayList<>();

    for (T value : list) {
        result.add(mapper.apply(value));
    }

    return result;
}
```

사용 예시는 다음과 같다.

```java
List<String> upperNames = map(names, name -> name.toUpperCase());
```

`filter`는 조건에 맞는 데이터만 남기는 것이고, `map`은 데이터를 다른 형태로 바꾸는 것이다.

---

## 14. 람다와 익명 클래스 차이

람다와 익명 클래스는 모두 간단한 구현을 전달할 때 사용할 수 있다.

하지만 차이가 있다.

### 익명 클래스

- 클래스를 선언하면서 바로 인스턴스를 생성한다.
- 여러 메서드를 오버라이드할 수 있다.
- 필드나 상태를 가질 수 있다.
- `this`는 익명 클래스 자기 자신을 의미한다.
- 함수형 인터페이스가 아니어도 사용할 수 있다.

### 람다

- 함수형 인터페이스에만 사용할 수 있다.
- 문법이 간결하다.
- 상태를 직접 가지기보다는 외부 값을 캡처해서 사용한다.
- `this`는 람다가 선언된 외부 객체를 의미한다.
- 단순한 동작 전달에 적합하다.

정리하면, 단일 메서드의 간단한 동작을 전달할 때는 람다가 적합하다.

반대로 여러 메서드 구현이나 상태 관리가 필요하면 익명 클래스가 더 적합할 수 있다.

---

## 15. 메서드 참조

메서드 참조는 기존 메서드를 람다식 대신 더 간결하게 표현하는 문법이다.

예를 들어 다음 람다는 단순히 `add` 메서드를 호출한다.

```java
BinaryOperator<Integer> add = (x, y) -> add(x, y);
```

이 경우 메서드 참조로 바꿀 수 있다.

```java
BinaryOperator<Integer> add = MethodRefMain::add;
```

메서드 참조는 새로운 로직을 만드는 것이 아니라, 이미 있는 메서드를 함수형 인터페이스의 시그니처에 맞게 연결하는 것이다.

대표적인 형태는 다음과 같다.

- 정적 메서드 참조: `ClassName::staticMethod`
- 특정 객체의 인스턴스 메서드 참조: `object::instanceMethod`
- 임의 객체의 인스턴스 메서드 참조: `ClassName::instanceMethod`
- 생성자 참조: `ClassName::new`

예를 들어 다음 코드는 문자열을 대문자로 변환한다.

```java
list.stream()
        .map(String::toUpperCase)
        .toList();
```

다음 코드는 값을 출력한다.

```java
list.forEach(System.out::println);
```

메서드 참조는 람다식이 단순히 기존 메서드를 호출하는 경우에 사용하면 좋다.

---

## 16. 함수형 프로그래밍 기초

함수형 프로그래밍은 부수 효과를 줄이고, 함수를 조합해서 프로그램을 구성하려는 방식이다.

중요한 개념은 다음과 같다.

- 순수 함수
- 불변성
- 부수 효과 최소화
- 선언형 프로그래밍
- 고차 함수

순수 함수는 같은 입력에 대해 항상 같은 출력을 반환하고, 외부 상태를 변경하지 않는 함수이다.

불변성은 값을 변경하기보다 새로운 값을 만들어 사용하는 방향이다.

부수 효과는 함수 실행 중 외부 상태가 바뀌는 것을 의미한다.

Java는 기본적으로 객체지향 언어지만, 람다와 Stream API를 통해 함수형 스타일도 사용할 수 있다.

---

## 17. 오늘 정리

람다는 단순히 코드를 짧게 쓰는 문법이 아니라, 동작을 값처럼 전달하기 위한 중요한 기능이다.

Java에서는 람다를 함수형 인터페이스를 통해 사용한다.

람다를 이해하기 위해서는 다음 흐름이 중요하다.

- 함수형 인터페이스는 추상 메서드가 하나만 있어야 한다.
- 람다식은 해당 추상 메서드의 시그니처와 일치해야 한다.
- 람다는 변수에 담거나 메서드에 전달하거나 반환할 수 있다.
- `Predicate`, `Function`, `Consumer`, `Supplier`를 알면 많은 람다 활용을 이해할 수 있다.
- 메서드 참조는 단순한 람다식을 더 간결하게 표현하는 방법이다.
- 람다는 함수형 프로그래밍과 Stream API를 이해하기 위한 기반이다.

앞으로는 단순 반복문을 람다로 바꾸는 것에서 끝내지 않고, 어떤 동작을 분리해서 전달할 수 있는지 생각하며 사용해야겠다.
