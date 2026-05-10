# Java 디자인 패턴 기초 정리

## 학습 날짜

2026-05-10

## 학습 주제

- 디자인 패턴의 기본 개념
- 객체지향 관점에서 자주 등장하는 설계 문제
- 각 패턴이 필요한 상황과 주의할 점

---

## 파사드 패턴

### 한 줄 설명

- 복잡한 기능들을 하나의 쉬운 창구로 감싸서 사용할 수 있게 하는 패턴이다.

### 왜 사용하는가

- 내부 시스템이 여러 클래스와 메서드로 복잡하게 나뉘어 있으면 사용하는 쪽 코드도 복잡해진다.
- 파사드를 사용하면 클라이언트는 내부 과정을 자세히 몰라도 필요한 기능을 간단한 메서드 하나로 사용할 수 있다.
- 복잡한 코드가 한곳에 모이기 때문에 사용하는 쪽의 코드가 읽기 쉬워진다.

### 쉬운 예시

- 영화를 보기 위해 TV 전원, 스피커, 셋톱박스, 조명을 각각 조작해야 한다고 생각할 수 있다.
- `HomeTheaterFacade.watchMovie()` 같은 메서드를 만들면 사용자는 버튼 하나처럼 영화를 볼 준비를 할 수 있다.

### Java 관점에서 생각하기

- 여러 객체를 조합해서 사용하는 별도의 클래스를 만든다.
- 클라이언트는 세부 클래스에 직접 의존하지 않고 파사드 클래스에 의존한다.
- 내부 구현이 바뀌어도 파사드의 사용법이 유지되면 클라이언트 코드는 덜 흔들린다.

### Java 코드 예시

```java
class Tv {
    void on() {
        System.out.println("TV 켜기");
    }
}

class Speaker {
    void on() {
        System.out.println("스피커 켜기");
    }
}

class HomeTheaterFacade {
    private final Tv tv = new Tv();
    private final Speaker speaker = new Speaker();

    void watchMovie() {
        tv.on();
        speaker.on();
        System.out.println("영화 보기 준비 완료");
    }
}
```

### 주의할 점

- 여러 기능을 묶어서 단순하게 제공해야 할 때 좋다.
- 파사드가 너무 많은 일을 직접 처리하면 거대한 관리 클래스가 될 수 있다.
- 단순한 기능까지 무리하게 감싸면 오히려 코드가 한 단계 더 복잡해질 수 있다.

---

## 전략 패턴

### 한 줄 설명

- 같은 일을 하는 여러 방법을 따로 분리해 두고, 상황에 따라 갈아 끼워 사용하는 패턴이다.

### 왜 사용하는가

- 조건문으로 처리 방식이 계속 늘어나면 코드가 길어지고 수정하기 어려워진다.
- 전략 패턴을 사용하면 각각의 처리 방식을 독립된 클래스로 분리할 수 있다.
- 새로운 방식이 추가되어도 기존 코드를 크게 바꾸지 않고 전략 클래스만 추가할 수 있다.

### 쉬운 예시

- 결제 기능에서 카드 결제, 계좌 이체, 포인트 결제가 모두 같은 `pay()` 역할을 한다고 볼 수 있다.
- 결제 방식만 전략으로 분리하면 주문 코드는 어떤 결제 방식인지 자세히 몰라도 결제를 요청할 수 있다.

### Java 관점에서 생각하기

- 공통 동작을 인터페이스로 만들고, 각 전략 클래스가 그 인터페이스를 구현한다.
- 사용하는 객체는 구체 클래스가 아니라 인터페이스 타입에 의존한다.
- 런타임에 어떤 전략 객체를 넣느냐에 따라 실행 결과가 달라진다.

### Java 코드 예시

```java
interface PaymentStrategy {
    void pay(int amount);
}

class CardPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println(amount + "원을 카드로 결제");
    }
}

class Order {
    private PaymentStrategy paymentStrategy;

    Order(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    void pay(int amount) {
        paymentStrategy.pay(amount);
    }
}
```

### 주의할 점

- 같은 목적의 알고리즘이나 처리 방식이 여러 개 있을 때 좋다.
- 전략이 한두 개뿐이고 변경 가능성도 낮다면 단순한 메서드나 조건문이 더 나을 수 있다.
- 전략 객체가 너무 많아지면 클래스 수가 늘어 관리 부담이 생길 수 있다.

---

## 템플릿 메서드 패턴

### 한 줄 설명

- 전체 작업 순서는 부모 클래스에서 정하고, 세부 단계는 자식 클래스가 다르게 구현하는 패턴이다.

### 왜 사용하는가

- 어떤 작업은 반드시 정해진 순서대로 실행되어야 한다.
- 하지만 각 단계의 구체적인 내용은 상황마다 달라질 수 있다.
- 템플릿 메서드를 사용하면 전체 흐름은 유지하면서 일부 단계만 다르게 만들 수 있다.

### 쉬운 예시

- 음료 만들기는 `물 끓이기 -> 재료 넣기 -> 컵에 따르기` 순서가 비슷하다.
- 커피와 차는 재료를 넣는 방법이 다르지만 전체 흐름은 유지할 수 있다.

### Java 관점에서 생각하기

- 부모 클래스에 전체 순서를 담당하는 메서드를 둔다.
- 바뀌는 단계는 `abstract` 메서드나 오버라이딩 가능한 메서드로 분리한다.
- 자식 클래스는 필요한 단계만 구현하고, 전체 실행 순서는 부모 클래스가 통제한다.

### Java 코드 예시

```java
abstract class Beverage {
    final void make() {
        boilWater();
        addIngredient();
        pour();
    }

    void boilWater() {
        System.out.println("물을 끓인다");
    }

    abstract void addIngredient();

    void pour() {
        System.out.println("컵에 따른다");
    }
}

class Coffee extends Beverage {
    void addIngredient() {
        System.out.println("커피를 넣는다");
    }
}
```

### 주의할 점

- 실행 순서가 중요하고 여러 클래스가 비슷한 흐름을 공유할 때 좋다.
- 상속을 기반으로 하기 때문에 부모와 자식 클래스의 관계가 강해진다.
- 흐름이 자주 바뀌는 경우에는 상속보다 조합을 사용하는 전략 패턴이 더 적절할 수 있다.

---

## 싱글턴 패턴

### 한 줄 설명

- 어떤 클래스의 객체가 프로그램 안에서 하나만 만들어지도록 제한하는 패턴이다.

### 왜 사용하는가

- 설정 정보, 공통 관리 객체처럼 하나만 있어야 자연스러운 객체가 있다.
- 여러 곳에서 같은 객체를 공유하면 상태나 설정을 일관되게 관리할 수 있다.
- 불필요하게 같은 객체를 계속 만드는 일을 줄일 수 있다.

### 쉬운 예시

- 애플리케이션 설정을 관리하는 `AppConfig` 객체는 여러 개일 필요가 없을 수 있다.
- 어디서든 같은 설정 객체를 가져와 사용하면 설정 값이 흩어지지 않는다.

### Java 관점에서 생각하기

- 생성자를 `private`으로 막고, 클래스 내부에서 하나의 인스턴스를 관리한다.
- 외부에서는 보통 `getInstance()` 같은 정적 메서드로 객체를 가져온다.
- 객체 생성 책임을 클래스 자신이 가진다는 점이 특징이다.

### Java 코드 예시

```java
class AppConfig {
    private static final AppConfig INSTANCE = new AppConfig();

    private AppConfig() {
    }

    static AppConfig getInstance() {
        return INSTANCE;
    }
}
```

### 주의할 점

- 정말 하나만 있어야 하는 객체에만 사용하는 것이 좋다.
- 전역 변수처럼 남용하면 테스트가 어려워지고 객체 사이의 숨은 의존성이 늘어난다.
- 멀티스레드 환경에서는 동시에 여러 객체가 만들어지지 않도록 주의해야 한다.

---

## 상태 패턴

### 한 줄 설명

- 객체의 상태를 별도 클래스로 나누고, 현재 상태에 따라 같은 요청도 다르게 동작하게 하는 패턴이다.

### 왜 사용하는가

- 상태에 따라 조건문이 계속 늘어나면 코드가 복잡해진다.
- 상태 패턴을 사용하면 상태별 행동을 각각의 클래스로 분리할 수 있다.
- 새로운 상태가 추가될 때 기존 조건문 전체를 수정하는 부담을 줄일 수 있다.

### 쉬운 예시

- 문은 `열림`, `닫힘`, `잠김` 상태를 가질 수 있다.
- 같은 `open()` 요청이라도 닫힌 문은 열리고, 잠긴 문은 열리지 않는 식으로 결과가 달라진다.

### Java 관점에서 생각하기

- 상태를 공통 인터페이스로 정의하고, 각 상태 클래스를 구현체로 만든다.
- 실제 객체는 현재 상태 객체를 필드로 가지고 있다.
- 요청이 들어오면 현재 상태 객체에게 동작을 위임한다.

### Java 코드 예시

```java
interface DoorState {
    void open(Door door);
}

class ClosedState implements DoorState {
    public void open(Door door) {
        System.out.println("문을 연다");
        door.setState(new OpenState());
    }
}

class OpenState implements DoorState {
    public void open(Door door) {
        System.out.println("이미 열려 있다");
    }
}

class Door {
    private DoorState state = new ClosedState();

    void setState(DoorState state) {
        this.state = state;
    }

    void open() {
        state.open(this);
    }
}
```

### 주의할 점

- 상태가 여러 개이고 상태마다 행동이 뚜렷하게 다를 때 좋다.
- 상태가 거의 없거나 단순하면 `if`문이 더 읽기 쉬울 수 있다.
- 상태 전환 규칙이 흩어지지 않도록 어느 클래스가 상태를 바꾸는지 명확히 해야 한다.

---

## 어댑터 패턴

### 한 줄 설명

- 서로 맞지 않는 규격을 중간 어댑터로 연결해서 같은 방식으로 사용할 수 있게 하는 패턴이다.

### 왜 사용하는가

- 이미 만들어진 클래스의 메서드 이름이나 사용 방식이 현재 코드와 맞지 않을 수 있다.
- 기존 코드를 직접 수정하지 않고도 필요한 인터페이스에 맞춰 사용할 수 있다.
- 외부 라이브러리나 레거시 코드를 현재 프로젝트에 연결할 때 유용하다.

### 쉬운 예시

- 한국 콘센트와 해외 플러그가 맞지 않을 때 변환 어댑터를 끼워 사용한다.
- 코드에서도 기존 클래스와 새 인터페이스 사이에 변환 역할을 하는 클래스를 둘 수 있다.

### Java 관점에서 생각하기

- 클라이언트가 기대하는 인터페이스를 어댑터가 구현한다.
- 어댑터 내부에서는 실제 기능을 가진 기존 객체를 호출한다.
- 클라이언트는 기존 객체의 구체적인 사용법을 몰라도 된다.

### Java 코드 예시

```java
interface Charger {
    void charge();
}

class OldCharger {
    void connectOldPort() {
        System.out.println("구형 포트로 충전");
    }
}

class ChargerAdapter implements Charger {
    private final OldCharger oldCharger;

    ChargerAdapter(OldCharger oldCharger) {
        this.oldCharger = oldCharger;
    }

    public void charge() {
        oldCharger.connectOldPort();
    }
}
```

### 주의할 점

- 수정하기 어려운 기존 코드나 외부 API를 맞춰 써야 할 때 좋다.
- 처음부터 직접 설계할 수 있는 코드라면 인터페이스를 맞게 설계하는 것이 더 단순할 수 있다.
- 어댑터가 너무 많아지면 실제 호출 흐름을 추적하기 어려워질 수 있다.

---

## 브릿지 패턴

### 한 줄 설명

- 기능의 큰 개념과 실제 구현을 분리해서 각각 독립적으로 바꿀 수 있게 하는 패턴이다.

### 왜 사용하는가

- 기능 종류와 구현 방식이 함께 늘어나면 클래스 조합이 폭발적으로 많아질 수 있다.
- 브릿지 패턴은 추상화와 구현을 분리해서 각각 따로 확장할 수 있게 한다.
- 상속만으로 모든 조합을 만들 때 생기는 복잡함을 줄일 수 있다.

### 쉬운 예시

- 리모컨은 TV, 라디오, 스피커 같은 여러 기기를 조작할 수 있다.
- 리모컨 종류와 실제 기기 종류를 분리하면 새 리모컨이나 새 기기를 따로 추가할 수 있다.

### Java 관점에서 생각하기

- 추상화 역할의 클래스가 구현 역할의 인터페이스를 필드로 가진다.
- 기능 호출은 추상화 객체에서 시작하지만 실제 동작은 구현 객체에 위임된다.
- 상속보다 조합을 사용해 변화하는 축을 분리한다.

### Java 코드 예시

```java
interface Device {
    void turnOn();
}

class TvDevice implements Device {
    public void turnOn() {
        System.out.println("TV 켜기");
    }
}

class RemoteControl {
    private final Device device;

    RemoteControl(Device device) {
        this.device = device;
    }

    void powerOn() {
        device.turnOn();
    }
}
```

### 주의할 점

- 두 방향으로 독립적인 확장이 예상될 때 좋다.
- 단순한 구조에서는 추상화와 구현을 분리하는 것 자체가 과한 설계가 될 수 있다.
- 어떤 부분이 추상화이고 어떤 부분이 구현인지 먼저 구분해야 한다.

---

## 팩토리 메서드 패턴

### 한 줄 설명

- 객체 생성을 직접 하지 않고, 객체를 만들어주는 메서드나 클래스를 통해 생성하는 패턴이다.

### 왜 사용하는가

- 클라이언트가 `new`로 구체 클래스를 직접 만들면 특정 클래스에 강하게 묶인다.
- 팩토리 메서드를 사용하면 어떤 객체를 만들지 결정하는 책임을 분리할 수 있다.
- 새로운 제품 클래스가 생겼을 때 생성 로직을 한곳에서 관리하기 쉽다.

### 쉬운 예시

- 문서 프로그램에서 `createDocument()`를 호출하면 PDF 문서나 워드 문서를 만들 수 있다.
- 사용하는 쪽은 구체적인 문서 클래스보다 문서라는 공통 역할에 집중한다.

### Java 관점에서 생각하기

- 반환 타입은 인터페이스나 부모 클래스 타입으로 두는 경우가 많다.
- 팩토리 메서드는 구체 객체 생성 코드를 숨기고 필요한 객체를 반환한다.
- 객체 생성 책임과 객체 사용 책임을 분리한다.

### Java 코드 예시

```java
interface Document {
    void open();
}

class PdfDocument implements Document {
    public void open() {
        System.out.println("PDF 문서 열기");
    }
}

class DocumentFactory {
    Document createDocument(String type) {
        if ("pdf".equals(type)) {
            return new PdfDocument();
        }
        throw new IllegalArgumentException("지원하지 않는 문서 형식");
    }
}
```

### 주의할 점

- 생성해야 할 객체 종류가 여러 개이거나 생성 과정이 자주 바뀔 때 좋다.
- 단순히 `new` 한 줄이면 충분한 상황에서는 불필요한 클래스가 늘어날 수 있다.
- 팩토리가 너무 많은 조건문을 가지면 다시 복잡한 생성 관리자가 될 수 있다.

---

## 프록시 패턴

### 한 줄 설명

- 실제 객체 대신 대리 객체를 앞에 세워 접근을 제어하거나 부가 기능을 처리하는 패턴이다.

### 왜 사용하는가

- 실제 객체를 바로 호출하기 전에 권한 확인, 지연 로딩, 캐싱, 로깅 같은 처리가 필요할 수 있다.
- 프록시는 실제 객체와 같은 인터페이스를 제공하면서 중간에서 요청을 관리한다.
- 클라이언트는 실제 객체인지 프록시인지 크게 신경 쓰지 않고 사용할 수 있다.

### 쉬운 예시

- 은행 계좌에 직접 접근하지 않고 은행 직원을 통해 출금 요청을 한다고 볼 수 있다.
- 직원은 신분 확인을 한 뒤 실제 출금 처리를 진행한다.

### Java 관점에서 생각하기

- 프록시와 실제 객체가 같은 인터페이스를 구현한다.
- 프록시는 내부에 실제 객체를 가지고 있다가 필요한 시점에 호출한다.
- Spring AOP에서 메서드 호출 전후로 기능을 붙이는 구조도 프록시 개념과 관련이 있다.

### Java 코드 예시

```java
interface Image {
    void display();
}

class RealImage implements Image {
    public void display() {
        System.out.println("이미지 표시");
    }
}

class ImageProxy implements Image {
    private RealImage realImage;

    public void display() {
        if (realImage == null) {
            realImage = new RealImage();
        }
        realImage.display();
    }
}
```

### 주의할 점

- 접근 제어, 로깅, 캐싱, 지연 생성이 필요할 때 좋다.
- 단순한 호출에 프록시를 추가하면 흐름이 한 단계 늘어나 디버깅이 어려워질 수 있다.
- 프록시가 실제 객체의 역할까지 대신하려고 하면 책임이 섞일 수 있다.

---

## 옵저버 패턴

### 한 줄 설명

- 한 객체의 변화가 생기면 그 객체를 지켜보던 여러 객체에게 알려주는 패턴이다.

### 왜 사용하는가

- 한 객체의 변경에 여러 객체가 반응해야 할 때 직접 호출 관계를 모두 만들면 결합도가 높아진다.
- 옵저버 패턴을 사용하면 대상 객체는 관찰자 목록만 관리하고, 변경이 생기면 알림을 보낸다.
- 관찰자를 추가하거나 제거하기 쉬워진다.

### 쉬운 예시

- 유튜브 채널을 구독하면 새 영상이 올라왔을 때 구독자들에게 알림이 간다.
- 채널은 구독자들이 각각 무엇을 하는지 자세히 몰라도 알림만 보낼 수 있다.

### Java 관점에서 생각하기

- 대상 객체는 옵저버 인터페이스 목록을 가진다.
- 변경이 발생하면 각 옵저버의 알림 메서드를 호출한다.
- 대상과 옵저버는 구체 클래스보다 인터페이스를 통해 연결된다.

### Java 코드 예시

```java
import java.util.ArrayList;
import java.util.List;

interface Observer {
    void update(String message);
}

class Channel {
    private final List<Observer> observers = new ArrayList<>();

    void subscribe(Observer observer) {
        observers.add(observer);
    }

    void upload(String title) {
        for (Observer observer : observers) {
            observer.update(title);
        }
    }
}
```

### 주의할 점

- 한 변경에 여러 객체가 반응해야 할 때 좋다.
- 알림 순서가 중요하거나 실패 처리가 복잡한 경우에는 설계를 더 신중히 해야 한다.
- 옵저버가 너무 많아지면 어떤 객체가 왜 실행되는지 추적하기 어려워질 수 있다.

---

## 플라이웨이트 패턴

### 한 줄 설명

- 반복해서 많이 필요한 객체를 매번 새로 만들지 않고 공유해서 메모리를 아끼는 패턴이다.

### 왜 사용하는가

- 같은 값이나 같은 속성을 가진 객체가 대량으로 만들어지면 메모리 낭비가 커질 수 있다.
- 공통으로 공유 가능한 부분을 분리하면 객체 생성 비용을 줄일 수 있다.
- 특히 작은 객체가 아주 많이 필요한 상황에서 효과가 있다.

### 쉬운 예시

- 게임에서 같은 나무 이미지를 수천 개 배치할 때 이미지 데이터를 매번 새로 만들 필요는 없다.
- 나무 종류 정보는 공유하고, 위치만 각 객체가 따로 가지면 된다.

### Java 관점에서 생각하기

- 공유 가능한 내부 상태와 객체마다 달라지는 외부 상태를 구분한다.
- 팩토리나 캐시를 사용해 이미 만들어진 객체가 있으면 재사용한다.
- 불변 객체로 만들면 공유할 때 더 안전하다.

### Java 코드 예시

```java
import java.util.HashMap;
import java.util.Map;

class TreeType {
    private final String name;

    TreeType(String name) {
        this.name = name;
    }
}

class TreeTypeFactory {
    private final Map<String, TreeType> cache = new HashMap<>();

    TreeType getTreeType(String name) {
        return cache.computeIfAbsent(name, TreeType::new);
    }
}
```

### 주의할 점

- 같은 객체가 대량으로 반복될 때 좋다.
- 객체 수가 적거나 메모리 문제가 없다면 굳이 적용할 필요가 없다.
- 공유 상태를 잘못 수정하면 여러 곳에 영향을 줄 수 있으므로 불변성을 고려해야 한다.

---

## 추상 팩토리 패턴

### 한 줄 설명

- 서로 관련 있는 객체들을 하나의 제품군으로 묶어 생성하는 패턴이다.

### 왜 사용하는가

- 팩토리 메서드가 보통 하나의 제품 생성을 다룬다면, 추상 팩토리는 관련된 여러 제품 생성을 함께 다룬다.
- 같은 계열의 객체들이 섞이지 않게 만들 수 있다.
- 제품군 전체를 바꿔야 할 때 팩토리만 교체하면 된다.

### 쉬운 예시

- 다크 모드 UI에서는 다크 버튼, 다크 체크박스, 다크 입력창이 필요하다.
- 라이트 모드 UI에서는 라이트 버튼, 라이트 체크박스, 라이트 입력창이 필요하다.
- 모드별 팩토리를 두면 서로 맞는 UI 제품군을 만들 수 있다.

### Java 관점에서 생각하기

- 추상 팩토리 인터페이스에 관련 객체 생성 메서드들을 정의한다.
- 구체 팩토리는 같은 계열의 객체들을 생성한다.
- 클라이언트는 구체 제품 클래스보다 추상 타입에 의존한다.

### Java 코드 예시

```java
interface Button {
    void render();
}

interface Checkbox {
    void render();
}

interface UiFactory {
    Button createButton();
    Checkbox createCheckbox();
}

class DarkUiFactory implements UiFactory {
    public Button createButton() {
        return () -> System.out.println("다크 버튼");
    }

    public Checkbox createCheckbox() {
        return () -> System.out.println("다크 체크박스");
    }
}
```

### 주의할 점

- 관련 객체들을 일관된 조합으로 만들어야 할 때 좋다.
- 제품 종류가 자주 추가되면 모든 팩토리 인터페이스와 구현체를 수정해야 할 수 있다.
- 제품군 개념이 뚜렷하지 않다면 팩토리 메서드나 단순 생성이 더 나을 수 있다.

---

## 중재자 패턴

### 한 줄 설명

- 여러 객체가 서로 직접 얽히지 않고, 중간의 중재자를 통해 소통하게 하는 패턴이다.

### 왜 사용하는가

- 객체들이 서로를 직접 많이 알고 있으면 하나를 바꿀 때 다른 객체도 영향을 받기 쉽다.
- 중재자를 두면 객체들은 중재자만 알고, 중재자가 전체 흐름을 조정한다.
- 객체 사이의 결합도를 낮출 수 있다.

### 쉬운 예시

- 채팅방에서는 사용자가 다른 모든 사용자에게 직접 메시지를 보내지 않는다.
- 사용자는 채팅방에 메시지를 보내고, 채팅방이 다른 사용자에게 전달한다.

### Java 관점에서 생각하기

- 여러 객체가 공통 중재자 인터페이스를 통해 요청을 보낸다.
- 중재자 클래스는 어떤 객체에게 어떤 메시지를 전달할지 결정한다.
- 객체들은 서로를 직접 참조하지 않아도 된다.

### Java 코드 예시

```java
import java.util.ArrayList;
import java.util.List;

class ChatRoom {
    private final List<User> users = new ArrayList<>();

    void join(User user) {
        users.add(user);
    }

    void send(String message, User sender) {
        for (User user : users) {
            if (user != sender) {
                user.receive(message);
            }
        }
    }
}

class User {
    private final ChatRoom chatRoom;

    User(ChatRoom chatRoom) {
        this.chatRoom = chatRoom;
    }

    void receive(String message) {
        System.out.println(message);
    }
}
```

### 주의할 점

- 객체 사이의 관계가 복잡하게 얽혀 있을 때 좋다.
- 중재자가 모든 로직을 떠안으면 너무 큰 클래스가 될 수 있다.
- 단순한 객체 관계에는 중재자를 추가하는 것이 오히려 복잡할 수 있다.

---

## 방문자 패턴

### 한 줄 설명

- 기존 객체 구조는 그대로 두고, 그 구조에 대해 수행할 작업을 별도 방문자 클래스로 추가하는 패턴이다.

### 왜 사용하는가

- 객체 구조는 자주 바뀌지 않지만, 그 객체들에 적용할 기능이 자주 추가될 수 있다.
- 방문자 패턴을 사용하면 기존 클래스 수정 없이 새로운 연산을 추가할 수 있다.
- 데이터 구조와 처리 로직을 분리할 수 있다.

### 쉬운 예시

- 쇼핑몰 장바구니에 책, 전자제품, 식품이 들어 있다.
- 할인 계산, 배송비 계산, 출력 기능을 각각 방문자로 만들면 상품 클래스 자체를 덜 수정할 수 있다.

### Java 관점에서 생각하기

- 방문자는 각 객체 타입을 처리하는 메서드를 가진다.
- 객체는 `accept(visitor)` 같은 메서드로 방문자를 받아들인다.
- 객체 구조와 연산 클래스를 분리한다.

### Java 코드 예시

```java
interface Item {
    void accept(ItemVisitor visitor);
}

class Book implements Item {
    public void accept(ItemVisitor visitor) {
        visitor.visit(this);
    }
}

interface ItemVisitor {
    void visit(Book book);
}

class PriceVisitor implements ItemVisitor {
    public void visit(Book book) {
        System.out.println("책 가격 계산");
    }
}
```

### 주의할 점

- 객체 종류는 안정적이고 기능이 자주 늘어날 때 좋다.
- 새로운 객체 타입이 자주 추가되면 모든 방문자에 메서드를 추가해야 해서 불편하다.
- 처음 공부할 때는 흐름이 낯설 수 있어 작은 예제로 이해하는 것이 좋다.

---

## 빌더 패턴

### 한 줄 설명

- 복잡한 객체를 여러 단계로 나누어 읽기 좋게 만드는 패턴이다.

### 왜 사용하는가

- 생성자 매개변수가 많아지면 어떤 값이 무엇을 의미하는지 알기 어렵다.
- 선택 값이 많은 객체는 생성자 오버로딩이 많아질 수 있다.
- 빌더를 사용하면 필요한 값만 단계적으로 설정하고 마지막에 객체를 만들 수 있다.

### 쉬운 예시

- 회원 가입 정보에서 이름, 이메일은 필수이고 주소, 프로필 이미지, 마케팅 동의는 선택일 수 있다.
- 빌더를 사용하면 선택 값을 필요한 만큼만 붙여 객체를 만들 수 있다.

### Java 관점에서 생각하기

- 보통 정적 내부 클래스인 `Builder`를 둔다.
- 빌더가 필드를 임시로 모은 뒤 `build()`에서 실제 객체를 생성한다.
- 생성자 호출보다 어떤 값을 설정하는지 코드에서 잘 드러난다.

### Java 코드 예시

```java
class UserProfile {
    private final String name;
    private final String email;

    private UserProfile(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
    }

    static class Builder {
        private String name;
        private String email;

        Builder name(String name) {
            this.name = name;
            return this;
        }

        Builder email(String email) {
            this.email = email;
            return this;
        }

        UserProfile build() {
            return new UserProfile(this);
        }
    }
}
```

### 주의할 점

- 필드가 많거나 선택 매개변수가 많을 때 좋다.
- 필드가 적은 단순 객체에는 빌더가 오히려 과할 수 있다.
- 객체 생성 규칙이 있다면 `build()`에서 검증하는 습관이 필요하다.

---

## 데코레이터 패턴

### 한 줄 설명

- 기존 객체를 감싸서 기능을 하나씩 덧붙이는 패턴이다.

### 왜 사용하는가

- 기능 조합이 많을 때 상속으로 모든 조합을 만들면 클래스 수가 너무 많아진다.
- 데코레이터는 필요한 기능을 런타임에 감싸서 추가할 수 있다.
- 원래 객체의 코드를 바꾸지 않고 부가 기능을 붙일 수 있다.

### 쉬운 예시

- 기본 커피에 샷 추가, 우유 추가, 시럽 추가를 선택적으로 붙일 수 있다.
- 각각의 추가 옵션을 데코레이터로 만들면 조합을 유연하게 만들 수 있다.

### Java 관점에서 생각하기

- 원본 객체와 데코레이터가 같은 인터페이스를 구현한다.
- 데코레이터는 내부에 같은 인터페이스 타입의 객체를 가지고 있다.
- 호출을 내부 객체에 위임하면서 앞뒤로 기능을 추가한다.

### Java 코드 예시

```java
interface Coffee {
    int cost();
}

class Americano implements Coffee {
    public int cost() {
        return 3000;
    }
}

class ShotDecorator implements Coffee {
    private final Coffee coffee;

    ShotDecorator(Coffee coffee) {
        this.coffee = coffee;
    }

    public int cost() {
        return coffee.cost() + 500;
    }
}
```

### 주의할 점

- 기능을 여러 방식으로 조합해야 할 때 좋다.
- 감싸는 단계가 너무 많아지면 실제 실행 흐름을 읽기 어려워진다.
- 단순히 기능 하나를 추가하는 정도라면 일반 메서드 수정이 더 나을 수 있다.

---

## 커맨드 패턴

### 한 줄 설명

- 실행할 명령을 객체로 만들어 저장하거나 나중에 실행할 수 있게 하는 패턴이다.

### 왜 사용하는가

- 요청을 보낸 쪽과 실제 실행하는 쪽을 분리할 수 있다.
- 명령 객체를 컬렉션에 저장하면 실행 기록, 실행 취소, 재실행, 로깅 등에 활용할 수 있다.
- 버튼, 메뉴, 단축키처럼 같은 명령을 여러 곳에서 실행해야 할 때 유용하다.

### 쉬운 예시

- 문서 편집기에서 `복사`, `붙여넣기`, `실행 취소`를 각각 명령 객체로 만들 수 있다.
- 실행한 명령을 스택에 저장하면 이전 작업을 되돌릴 수 있다.

### Java 관점에서 생각하기

- `Command` 인터페이스에 `execute()` 같은 메서드를 둔다.
- 각 명령 클래스는 실제 작업을 수행할 객체를 내부에 가진다.
- 호출자는 구체 명령의 내용을 몰라도 `execute()`만 호출하면 된다.

### Java 코드 예시

```java
interface Command {
    void execute();
}

class Light {
    void on() {
        System.out.println("불 켜기");
    }
}

class LightOnCommand implements Command {
    private final Light light;

    LightOnCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.on();
    }
}
```

### 주의할 점

- 실행 기록이나 취소 기능이 필요할 때 좋다.
- 모든 작은 동작을 명령 객체로 만들면 클래스가 너무 많아질 수 있다.
- 실행 취소까지 필요하다면 이전 상태를 어떻게 보관할지도 함께 설계해야 한다.

---

## 메멘토 패턴

### 한 줄 설명

- 객체의 상태를 스냅샷처럼 저장해 두었다가 필요할 때 되돌리는 패턴이다.

### 왜 사용하는가

- 객체 내부 상태를 직접 외부에 공개하지 않고도 저장과 복원이 필요할 수 있다.
- 메멘토를 사용하면 캡슐화를 지키면서 이전 상태로 돌아갈 수 있다.
- 실행 취소, 임시 저장, 게임 저장 같은 기능에 사용할 수 있다.

### 쉬운 예시

- 글 작성 중 현재 문서 상태를 저장해 두었다가 실수하면 이전 상태로 되돌릴 수 있다.
- 저장된 상태가 메멘토 역할을 한다.

### Java 관점에서 생각하기

- 원본 객체는 자신의 상태를 담은 메멘토 객체를 만든다.
- 관리 객체는 메멘토를 보관하지만 내부 내용을 함부로 수정하지 않는다.
- 원본 객체만 메멘토를 이용해 상태를 복원한다.

### Java 코드 예시

```java
class EditorMemento {
    private final String text;

    EditorMemento(String text) {
        this.text = text;
    }

    String getText() {
        return text;
    }
}

class Editor {
    private String text;

    EditorMemento save() {
        return new EditorMemento(text);
    }

    void restore(EditorMemento memento) {
        this.text = memento.getText();
    }
}
```

### 주의할 점

- 이전 상태로 되돌리는 기능이 필요할 때 좋다.
- 상태가 크거나 저장 횟수가 많으면 메모리를 많이 사용할 수 있다.
- 어떤 상태까지 저장해야 하는지 기준을 분명히 해야 한다.

---

## 프로토타입 패턴

### 한 줄 설명

- 새 객체를 처음부터 만들지 않고, 이미 만들어진 객체를 복제해서 사용하는 패턴이다.

### 왜 사용하는가

- 객체 생성 비용이 크거나 초기 설정이 복잡할 때 매번 새로 만들면 비효율적이다.
- 프로토타입을 복제하면 비슷한 객체를 빠르게 만들 수 있다.
- 런타임에 만들어진 객체를 기준으로 새 객체를 만들 수 있다.

### 쉬운 예시

- 게임 캐릭터 기본 설정을 가진 객체를 하나 만들어 두고, 이를 복사해 여러 캐릭터를 만들 수 있다.
- 복사한 뒤 이름이나 위치만 다르게 바꾸면 된다.

### Java 관점에서 생각하기

- 객체가 자기 자신을 복사하는 메서드를 제공한다.
- 단순한 얕은 복사와 내부 객체까지 복사하는 깊은 복사를 구분해야 한다.
- Java의 `Cloneable`을 사용할 수도 있지만, 직접 복사 생성자나 복사 메서드를 만드는 방식도 고려할 수 있다.

### Java 코드 예시

```java
class Monster {
    private final String type;
    private int hp;

    Monster(String type, int hp) {
        this.type = type;
        this.hp = hp;
    }

    Monster copy() {
        return new Monster(type, hp);
    }
}
```

### 주의할 점

- 생성 비용이 큰 객체를 비슷하게 여러 개 만들어야 할 때 좋다.
- 내부에 참조 타입 필드가 있으면 복사 방식에 따라 예상치 못한 공유가 생길 수 있다.
- 단순한 객체라면 일반 생성자가 더 명확할 수 있다.

---

## 책임 연쇄 패턴

### 한 줄 설명

- 요청을 처리할 수 있는 객체들이 줄줄이 이어져 있고, 요청이 체인을 따라 전달되는 패턴이다.

### 왜 사용하는가

- 요청을 누가 처리할지 미리 하나로 정하기 어려울 수 있다.
- 여러 처리 객체가 순서대로 요청을 확인하고, 자신이 처리할 수 있으면 처리한다.
- 요청을 보내는 쪽은 어떤 객체가 최종 처리하는지 몰라도 된다.

### 쉬운 예시

- 고객 문의가 1차 상담원, 팀장, 관리자 순서로 올라간다고 볼 수 있다.
- 앞 단계에서 해결하지 못하면 다음 단계로 넘긴다.

### Java 관점에서 생각하기

- 각 처리 객체는 다음 처리 객체를 참조한다.
- 공통 인터페이스나 추상 클래스로 `handle()` 같은 메서드를 정의한다.
- 처리하지 못한 요청은 다음 객체에게 위임한다.

### Java 코드 예시

```java
abstract class SupportHandler {
    private SupportHandler next;

    SupportHandler setNext(SupportHandler next) {
        this.next = next;
        return next;
    }

    void handle(String request) {
        if (canHandle(request)) {
            System.out.println("요청 처리");
        } else if (next != null) {
            next.handle(request);
        }
    }

    abstract boolean canHandle(String request);
}
```

### 주의할 점

- 여러 단계 중 하나가 요청을 처리해야 할 때 좋다.
- 요청이 끝까지 처리되지 않을 가능성을 고려해야 한다.
- 체인이 길어지면 성능과 디버깅 흐름을 확인해야 한다.

---

## 복합체 패턴

### 한 줄 설명

- 개별 객체와 객체 묶음을 같은 방식으로 다루기 위한 패턴이다.

### 왜 사용하는가

- 트리 구조처럼 전체와 부분이 반복되는 구조가 있다.
- 복합체 패턴을 사용하면 단일 객체와 그룹 객체를 같은 인터페이스로 다룰 수 있다.
- 클라이언트는 대상이 하나인지 여러 개의 묶음인지 크게 신경 쓰지 않아도 된다.

### 쉬운 예시

- 폴더 안에는 파일도 있고 다른 폴더도 있을 수 있다.
- 파일과 폴더 모두 `getSize()`를 제공하면 전체 크기를 같은 방식으로 계산할 수 있다.

### Java 관점에서 생각하기

- 공통 인터페이스를 만들고, 잎 객체와 복합 객체가 이를 구현한다.
- 복합 객체는 내부에 같은 인터페이스 타입의 자식 목록을 가진다.
- 재귀 구조를 사용해 전체와 부분을 같은 방식으로 처리한다.

### Java 코드 예시

```java
import java.util.ArrayList;
import java.util.List;

interface FileNode {
    int size();
}

class FileItem implements FileNode {
    private final int size;

    FileItem(int size) {
        this.size = size;
    }

    public int size() {
        return size;
    }
}

class Folder implements FileNode {
    private final List<FileNode> children = new ArrayList<>();

    void add(FileNode node) {
        children.add(node);
    }

    public int size() {
        return children.stream().mapToInt(FileNode::size).sum();
    }
}
```

### 주의할 점

- 트리 구조나 계층 구조를 표현할 때 좋다.
- 모든 객체에 같은 메서드를 두는 것이 어색한 경우에는 인터페이스 설계를 조심해야 한다.
- 구조가 단순한 리스트 정도라면 복합체 패턴까지는 필요 없을 수 있다.

---

## 인터프리터 패턴

### 한 줄 설명

- 특정 규칙이나 문법으로 작성된 문장을 해석해서 실행하는 패턴이다.

### 왜 사용하는가

- 간단한 언어, 조건식, 쿼리처럼 정해진 문법을 코드로 해석해야 할 수 있다.
- 문법 요소를 객체로 나누면 각 요소가 자신의 해석 방법을 가질 수 있다.
- 규칙을 조합해 다양한 표현을 처리할 수 있다.

### 쉬운 예시

- 검색 조건에서 `age > 20 AND city = Seoul` 같은 문장을 해석한다고 볼 수 있다.
- 문자열을 키워드, 연산자, 값으로 나누고 각각의 의미대로 실행한다.

### Java 관점에서 생각하기

- 문법 요소를 나타내는 표현식 클래스를 만든다.
- 각 표현식은 `interpret()` 같은 메서드로 자신을 해석한다.
- 파싱 과정과 실행 과정을 구분해서 생각해야 한다.

### Java 코드 예시

```java
interface Expression {
    boolean interpret(String text);
}

class ContainsExpression implements Expression {
    private final String keyword;

    ContainsExpression(String keyword) {
        this.keyword = keyword;
    }

    public boolean interpret(String text) {
        return text.contains(keyword);
    }
}
```

### 주의할 점

- 문법이 작고 단순할 때 좋다.
- 문법이 복잡해지면 직접 구현하기보다 검증된 파서나 라이브러리를 사용하는 것이 낫다.
- 문자열 처리만으로 억지 구현하면 유지보수가 어려워질 수 있다.

---

## 발행-구독 패턴

### 한 줄 설명

- 메시지를 보내는 쪽과 받는 쪽이 직접 연결되지 않고, 중간 브로커를 통해 메시지를 주고받는 패턴이다.

### 왜 사용하는가

- 발행자와 구독자가 서로를 직접 알면 관계가 강하게 묶인다.
- 브로커를 두면 발행자는 메시지만 보내고, 구독자는 관심 있는 메시지만 받는다.
- 시스템 간 통신이나 이벤트 기반 구조에서 결합도를 낮출 수 있다.

### 쉬운 예시

- 뉴스 앱에서 사용자는 관심 있는 주제를 구독한다.
- 기자가 뉴스를 발행하면 중간 플랫폼이 해당 주제를 구독한 사용자에게 전달한다.

### Java 관점에서 생각하기

- 발행자는 브로커나 이벤트 버스에 메시지를 보낸다.
- 구독자는 특정 주제나 이벤트 타입을 구독한다.
- 발행자와 구독자는 서로의 구체 클래스를 몰라도 된다.

### Java 코드 예시

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

interface Subscriber {
    void receive(String message);
}

class EventBroker {
    private final Map<String, List<Subscriber>> subscribers = new HashMap<>();

    void subscribe(String topic, Subscriber subscriber) {
        subscribers.computeIfAbsent(topic, key -> new ArrayList<>()).add(subscriber);
    }

    void publish(String topic, String message) {
        for (Subscriber subscriber : subscribers.getOrDefault(topic, List.of())) {
            subscriber.receive(message);
        }
    }
}
```

### 주의할 점

- 여러 시스템이나 여러 객체가 느슨하게 이벤트를 주고받아야 할 때 좋다.
- 메시지 흐름이 눈에 잘 보이지 않아 디버깅이 어려울 수 있다.
- 메시지 순서, 실패 처리, 중복 처리 같은 규칙을 정해야 한다.

---

## 반복자 패턴

### 한 줄 설명

- 컬렉션 내부 구조를 몰라도 요소를 하나씩 꺼내 볼 수 있게 하는 패턴이다.

### 왜 사용하는가

- 리스트, 배열, 트리처럼 내부 구조가 달라도 사용하는 쪽은 순회 방법을 통일하고 싶을 수 있다.
- 반복자를 사용하면 컬렉션의 내부 구현을 숨기면서 요소를 차례대로 접근할 수 있다.
- 순회 책임을 컬렉션 밖의 반복자 객체로 분리할 수 있다.

### 쉬운 예시

- 책장 안의 책을 왼쪽부터 하나씩 꺼내 보는 것처럼, 컬렉션의 요소를 순서대로 확인한다.
- 사용하는 사람은 책장이 내부적으로 어떻게 정리되어 있는지 자세히 몰라도 된다.

### Java 관점에서 생각하기

- Java의 `Iterator`가 대표적인 예시다.
- `hasNext()`로 다음 요소가 있는지 확인하고, `next()`로 요소를 가져온다.
- `for-each` 문도 내부적으로 반복자 개념과 연결된다.

### Java 코드 예시

```java
import java.util.Iterator;
import java.util.List;

class Bookshelf {
    private final List<String> books = List.of("Java", "Spring", "Database");

    Iterator<String> iterator() {
        return books.iterator();
    }
}
```

### 주의할 점

- 컬렉션 내부 구조를 숨기고 순회 방법을 통일하고 싶을 때 좋다.
- 단순 배열을 한 번 순회하는 정도라면 일반 반복문으로 충분하다.
- 반복 중 컬렉션을 수정하면 예외나 예상하지 못한 동작이 생길 수 있다.

---

## 명세 패턴

### 한 줄 설명

- 객체가 특정 조건을 만족하는지 판단하는 규칙을 객체로 분리하는 패턴이다.

### 왜 사용하는가

- 조건문이 여러 곳에 흩어지면 같은 규칙을 반복해서 작성하게 된다.
- 명세 패턴을 사용하면 조건을 재사용 가능한 객체로 만들 수 있다.
- 여러 조건을 조합해서 더 복잡한 조건을 만들 수 있다.

### 쉬운 예시

- 상품이 `재고가 있는가`, `가격이 1만 원 이상인가`, `할인 가능한가` 같은 조건을 만족하는지 확인할 수 있다.
- 각각의 조건을 명세로 만들면 필요한 곳에서 조합해 사용할 수 있다.

### Java 관점에서 생각하기

- 보통 `isSatisfiedBy()` 같은 메서드를 가진 인터페이스를 만든다.
- 구체 명세 클래스는 하나의 조건 판단을 담당한다.
- `and`, `or`, `not` 같은 조합을 만들면 조건을 더 유연하게 표현할 수 있다.

### Java 코드 예시

```java
interface Specification<T> {
    boolean isSatisfiedBy(T target);
}

class Product {
    int price;
    int stock;
}

class InStockSpecification implements Specification<Product> {
    public boolean isSatisfiedBy(Product product) {
        return product.stock > 0;
    }
}
```

### 주의할 점

- 비즈니스 규칙이나 필터 조건이 자주 재사용될 때 좋다.
- 한 번만 쓰는 단순 조건까지 명세로 만들면 코드가 불필요하게 많아질 수 있다.
- 조건 이름을 명확하게 지어야 읽는 사람이 규칙을 이해하기 쉽다.

---

## 오늘 정리

- 디자인 패턴은 정답처럼 외우는 것이 아니라, 반복해서 등장하는 설계 문제를 해결하기 위한 방법이라고 이해했다.
- 처음부터 모든 패턴을 코드에 적용하려고 하기보다, 문제가 생겼을 때 어떤 패턴이 도움이 될 수 있는지 떠올리는 것이 중요하다고 느꼈다.
- 현재 `sprint-mission` 코드에서는 공통 설정이나 공유 객체가 있다면 싱글턴 패턴을 조심스럽게 검토할 수 있다.
- 객체 생성 로직이 여러 곳에 흩어져 있다면 팩토리 메서드 패턴으로 생성 책임을 분리해볼 수 있다.
- 정렬, 검색, 검증처럼 처리 방식이 여러 개로 바뀔 수 있는 기능이 있다면 전략 패턴으로 분리해볼 수 있다.
- 요청 처리 흐름이 단계별로 이어진다면 책임 연쇄 패턴도 적용 가능성을 생각해볼 수 있다.
