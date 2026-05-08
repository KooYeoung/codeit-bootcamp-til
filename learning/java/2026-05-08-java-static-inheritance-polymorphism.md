# Java static, 상속, 다형성, 추상 클래스와 인터페이스

## 학습 날짜

2026-05-08

## 학습 주제

- `static`
- `final`
- 상속
- 오버라이딩
- `super`
- 다형성
- 타입 캐스팅과 `instanceof`
- 추상 클래스와 추상 메서드
- 인터페이스
- 추상 클래스와 인터페이스의 차이

---

## 1. static

`static`은 객체가 아니라 클래스에 소속되는 멤버를 만들 때 사용한다.

일반 필드는 객체를 만들 때마다 각 객체 안에 따로 생긴다. 반면 `static` 필드는 객체마다 복사되는 값이 아니라 클래스 기준으로 하나만 존재하고, 모든 객체가 그 값을 공유한다.

```java
class Counter {
    static int count;

    Counter() {
        count++;
    }
}
```

위 코드에서 `count`는 객체마다 따로 존재하는 값이 아니라 `Counter` 클래스가 공유하는 값이다. 그래서 객체가 여러 개 만들어져도 하나의 `count` 값이 증가한다고 이해했다.

`static` 멤버는 클래스 이름으로 바로 접근할 수 있다.

```java
Counter.count = 10;
```

객체를 만들지 않아도 사용할 수 있기 때문에 참조 변수가 꼭 필요하지 않다. 이 점 때문에 `static` 메서드 안에서는 `this`를 사용할 수 없다. `this`는 현재 객체 자신을 가리키는 키워드인데, `static` 메서드는 객체 생성 없이 호출될 수 있기 때문이다.

```java
class Example {
    int number = 10;
    static int shared = 20;

    static void printShared() {
        System.out.println(shared);
    }
}
```

`static` 메서드에서는 같은 `static` 멤버를 바로 사용할 수 있다. 하지만 인스턴스 필드나 인스턴스 메서드를 사용하려면 객체를 먼저 만들어야 한다.

```java
class Example {
    int number = 10;

    static void printNumber() {
        Example example = new Example();
        System.out.println(example.number);
    }
}
```

`static`은 모든 객체가 공유해도 문제가 없는 값이나 기능에 붙이는 것이 적절하다. 객체마다 달라져야 하는 상태에 `static`을 붙이면 의도하지 않은 값 공유가 발생할 수 있다.

---

## 2. Singleton 패턴

싱글톤 패턴은 클래스의 인스턴스가 하나만 만들어지도록 제한하는 디자인 패턴이다.

여러 곳에서 같은 객체를 공유해야 하거나, 객체를 계속 새로 만드는 것이 불필요할 때 사용할 수 있다.

```java
class AppConfig {
    private static final AppConfig instance = new AppConfig();

    private AppConfig() {
    }

    static AppConfig getInstance() {
        return instance;
    }
}
```

생성자를 `private`으로 막으면 외부에서 `new AppConfig()`를 호출할 수 없다. 대신 클래스 내부에 미리 만들어 둔 객체를 `getInstance()`로 제공한다.

이 구조는 객체 생성을 제한할 수 있다는 장점이 있지만, 모든 곳에서 같은 객체를 공유하므로 상태를 조심해서 다뤄야 한다고 느꼈다.

---

## 3. final

`final`은 더 이상 바뀌지 않도록 제한할 때 사용하는 키워드이다.

변수에 붙이면 값을 한 번만 대입할 수 있다.

```java
final int maxCount = 10;
```

상수는 변하지 않는 값이면서 여러 객체가 공유해도 되는 값이므로 보통 `static final`을 함께 사용한다.

```java
class MathValue {
    static final double PI = 3.14;
}
```

`static`은 공유의 의미가 있고, `final`은 변경 불가의 의미가 있다. 따라서 `static final`은 클래스 전체에서 공유하면서도 바뀌면 안 되는 값을 표현할 때 적합하다고 이해했다.

---

## 4. 상속

상속은 부모 클래스가 가진 필드와 메서드를 자식 클래스가 물려받는 것이다.

비슷한 속성과 기능을 가진 클래스가 여러 개 있을 때 공통 부분을 부모 클래스로 분리하면 코드 중복을 줄일 수 있다.

```java
class Animal {
    void eat() {
        System.out.println("먹는다");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("짖는다");
    }
}
```

`Dog`는 `Animal`을 상속받았기 때문에 `eat()`을 사용할 수 있다. 자식 클래스는 부모 클래스의 기능을 물려받고, 필요한 기능을 추가할 수 있다.

Java에서 클래스 상속은 단일 상속만 가능하다. 즉, 하나의 클래스는 하나의 부모 클래스만 직접 상속할 수 있다.

상속에서 주의할 점은 부모의 생성자는 상속되지 않는다는 것이다. 자식 객체가 만들어질 때 부모 객체 부분도 함께 만들어지지만, 부모 생성자를 그대로 물려받아 사용하는 것은 아니다.

또한 `private` 멤버는 자식 클래스에서 직접 접근할 수 없다. 부모 내부에 숨겨진 값은 부모가 제공하는 메서드를 통해 접근해야 한다.

---

## 5. 오버라이딩

오버라이딩은 부모에게 물려받은 메서드를 자식 클래스에서 다시 정의하는 것이다.

부모의 기본 동작이 자식 클래스에 맞지 않거나 더 구체적인 동작이 필요할 때 사용한다.

```java
class Animal {
    void sound() {
        System.out.println("소리를 낸다");
    }
}

class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("야옹");
    }
}
```

`Cat` 객체에서 `sound()`를 호출하면 부모의 메서드가 아니라 자식 클래스에서 다시 정의한 메서드가 실행된다.

오버라이딩을 할 때는 부모 메서드와 이름, 매개변수, 반환 타입이 맞아야 한다. 접근 제한자는 부모보다 좁게 만들 수 없다.

`@Override`는 필수 문법은 아니지만, 오버라이딩이 제대로 되었는지 컴파일러가 확인해 주므로 사용하는 것이 좋다고 느꼈다.

---

## 6. super

`super`는 부모 객체를 가리킬 때 사용하는 키워드이다.

부모의 필드나 메서드에 접근할 때는 `super.`을 사용할 수 있고, 부모 생성자를 호출할 때는 `super()`를 사용할 수 있다.

```java
class Parent {
    Parent(String name) {
        System.out.println(name);
    }
}

class Child extends Parent {
    Child() {
        super("parent");
    }
}
```

자식 생성자에서 부모 생성자를 호출해야 할 때 `super()`를 사용한다. 부모 생성자 호출은 자식 생성자의 첫 줄에 와야 한다.

부모의 메서드를 완전히 대체하지 않고 일부 동작만 덧붙이고 싶을 때도 `super`를 사용할 수 있다.

```java
class ParentPrinter {
    void print() {
        System.out.println("parent");
    }
}

class ChildPrinter extends ParentPrinter {
    @Override
    void print() {
        super.print();
        System.out.println("child");
    }
}
```

---

## 7. 다형성

다형성은 하나의 객체가 여러 타입으로 다뤄질 수 있는 성질이다.

Java에서는 자식 객체를 부모 타입의 참조 변수에 담을 수 있다.

```java
Animal animal = new Cat();
animal.sound();
```

참조 변수의 타입은 `Animal`이지만 실제 객체는 `Cat`이다. 이때 오버라이딩된 메서드를 호출하면 실제 객체인 `Cat`의 메서드가 실행된다.

다형성을 사용하면 여러 자식 클래스를 부모 타입 하나로 묶어 다룰 수 있다.

```java
Animal[] animals = { new Cat(), new Dog() };

for (Animal animal : animals) {
    animal.sound();
}
```

이렇게 작성하면 객체의 구체적인 클래스가 달라도 공통 부모 타입을 기준으로 코드를 단순하게 만들 수 있다.

다형성은 상속이나 인터페이스를 전제로 한다. 공통 타입이 있어야 여러 객체를 하나의 기준으로 다룰 수 있기 때문이다.

---

## 8. 타입 캐스팅과 instanceof

부모 타입의 참조 변수로는 부모가 알고 있는 멤버만 사용할 수 있다.

실제 객체가 자식 타입이더라도 참조 변수의 타입이 부모라면 자식 고유 메서드는 바로 호출할 수 없다.

```java
Animal animal = new Cat();
```

이 상태에서 `Cat`에만 있는 기능을 사용하려면 자식 타입으로 형변환해야 한다.

```java
if (animal instanceof Cat) {
    Cat cat = (Cat) animal;
    cat.scratch();
}
```

`instanceof`는 왼쪽 객체가 오른쪽 타입으로 다뤄질 수 있는지 확인할 때 사용한다. 형변환 전에 확인하면 잘못된 타입 캐스팅으로 인한 오류를 줄일 수 있다.

---

## 9. 추상 클래스와 추상 메서드

추상 메서드는 선언만 있고 실행 블록이 없는 메서드이다.

```java
abstract void move();
```

추상 메서드는 자식 클래스가 반드시 구현해야 하는 메서드의 틀이다.

추상 메서드를 하나라도 가진 클래스는 추상 클래스가 되어야 한다.

```java
abstract class Vehicle {
    abstract void move();
}

class Car extends Vehicle {
    @Override
    void move() {
        System.out.println("도로를 달린다");
    }
}
```

추상 클래스는 직접 객체를 만들 수 없다. 아직 완성되지 않은 부분이 있기 때문이다. 대신 추상 클래스를 상속받은 자식 클래스가 추상 메서드를 구현한 뒤 객체로 만들어진다.

추상 클래스는 공통 필드나 공통 메서드를 가질 수 있다. 그래서 여러 자식 클래스가 공유해야 하는 코드가 있고, 동시에 반드시 구현해야 하는 메서드가 있을 때 적합하다고 이해했다.

---

## 10. 인터페이스

인터페이스는 어떤 클래스가 반드시 구현해야 할 기능의 규칙을 정하는 역할을 한다.

클래스가 인터페이스를 구현하면, 인터페이스에 선언된 메서드를 반드시 구현해야 한다.

```java
interface Flyable {
    void fly();
}

class Airplane implements Flyable {
    @Override
    public void fly() {
        System.out.println("비행기가 난다");
    }
}
```

상속은 `extends`를 사용하지만, 인터페이스 구현은 `implements`를 사용한다.

인터페이스는 객체를 직접 생성하기 위한 개념이 아니라, 객체들이 같은 방식으로 사용될 수 있도록 약속을 만드는 개념이다.

하나의 클래스는 여러 인터페이스를 구현할 수 있다.

```java
interface Printable {
    void print();
}

interface Scannable {
    void scan();
}

class MultiPrinter implements Printable, Scannable {
    public void print() {
        System.out.println("출력한다");
    }

    public void scan() {
        System.out.println("스캔한다");
    }
}
```

인터페이스를 사용하면 상속 관계가 깊지 않아도 같은 기능을 가진 객체들을 하나의 타입으로 다룰 수 있다. 이 점에서 다형성을 더 유연하게 사용할 수 있다고 이해했다.

---

## 11. 추상 클래스와 인터페이스의 차이

추상 클래스와 인터페이스는 모두 완성되지 않은 기능을 자식 클래스나 구현 클래스가 완성하게 만들 수 있다.

하지만 목적에는 차이가 있다.

| 구분 | 추상 클래스 | 인터페이스 |
| --- | --- | --- |
| 관계 | 상속 관계 | 구현 약속 |
| 키워드 | `extends` | `implements` |
| 객체 생성 | 직접 생성 불가 | 직접 생성 불가 |
| 다중 적용 | 클래스는 단일 상속 | 여러 인터페이스 구현 가능 |
| 목적 | 공통 코드 재사용과 확장 | 동일한 사용 방식 강제 |

추상 클래스는 부모 클래스의 공통 기능을 물려주면서 일부 기능의 구현을 자식에게 맡기는 구조이다.

인터페이스는 클래스들이 같은 이름의 기능을 제공하도록 강제하고, 사용자 입장에서 같은 방식으로 사용할 수 있게 만드는 규칙에 가깝다.

공통 필드나 구현된 메서드를 함께 물려줘야 한다면 추상 클래스를 고려할 수 있다. 서로 다른 계층의 클래스라도 같은 기능을 제공해야 한다면 인터페이스가 더 적합하다고 느꼈다.

---

## 12. 오늘 정리

오늘은 Java 객체지향의 핵심 개념을 이어서 학습했다.

`static`은 객체가 아니라 클래스 기준으로 공유되는 멤버이고, `final`은 변경을 막는 키워드이다.

상속은 공통 기능을 부모 클래스로 묶어 재사용하는 방식이며, 오버라이딩을 통해 자식 클래스에 맞는 동작으로 바꿀 수 있다.

다형성은 부모 타입이나 인터페이스 타입으로 여러 객체를 다룰 수 있게 해 준다. 이때 자식 고유 기능이 필요하면 타입 캐스팅과 `instanceof`를 함께 고려해야 한다.

추상 클래스는 공통 코드와 미완성 메서드를 함께 다룰 때 유용하고, 인터페이스는 여러 클래스에 동일한 사용 규칙을 강제할 때 유용하다고 정리했다.
