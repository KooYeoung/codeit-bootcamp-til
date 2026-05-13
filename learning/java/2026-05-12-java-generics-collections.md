# Java 제네릭과 컬렉션 프레임워크 정리

## 학습 날짜

2026-05-12

## 학습 주제

- 제네릭
- 타입 매개변수와 타입 인자
- 타입 매개변수 제한
- 제네릭 메서드
- 와일드카드
- 타입 이레이저
- ArrayList
- LinkedList
- List
- Hash
- HashSet
- Set
- Map
- Stack
- Queue
- Iterable, Iterator
- Comparable, Comparator

---

## 1. 제네릭이 필요한 이유

제네릭은 클래스나 메서드를 작성할 때 특정 타입에 고정하지 않고, 사용하는 시점에 타입을 정할 수 있게 해주는 기능이다.

예를 들어 값을 담는 박스를 만든다고 할 때, 문자열 박스와 숫자 박스를 각각 따로 만들면 코드 중복이 생긴다.

```java
public class StringBox {
    private String value;
}

public class IntegerBox {
    private Integer value;
}
```

이런 중복을 줄이기 위해 Object를 사용할 수도 있다.

```java
public class ObjectBox {
    private Object value;
}
```

하지만 Object를 사용하면 값을 꺼낼 때 다운캐스팅이 필요하고, 잘못된 타입이 들어가도 컴파일 시점에 막기 어렵다.

제네릭은 이 문제를 해결한다.

```java
public class GenericBox<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```

사용할 때 타입을 정한다.

```java
GenericBox<String> stringBox = new GenericBox<>();
GenericBox<Integer> integerBox = new GenericBox<>();
```

이렇게 하면 코드 재사용성과 타입 안전성을 함께 얻을 수 있다.

---

## 2. 타입 매개변수와 타입 인자

제네릭에서 자주 나오는 용어는 다음과 같다.

### 제네릭 타입

타입 매개변수를 사용하는 클래스나 인터페이스를 말한다.

```java
class GenericBox<T> {
}
```

여기서 `GenericBox<T>`는 제네릭 타입이다.

### 타입 매개변수

제네릭 타입이나 메서드에서 사용하는 타입 변수이다.

```java
class GenericBox<T> {
}
```

여기서 `T`가 타입 매개변수이다.

### 타입 인자

제네릭을 실제로 사용할 때 전달하는 타입이다.

```java
GenericBox<Integer> box = new GenericBox<>();
```

여기서 `Integer`가 타입 인자이다.

### 명명 관례

제네릭 타입 매개변수는 보통 대문자 한 글자를 사용한다.

- `T`: Type
- `E`: Element
- `K`: Key
- `V`: Value
- `N`: Number

---

## 3. 제네릭과 기본형

제네릭 타입 인자로 기본형은 사용할 수 없다.

```java
GenericBox<int> box; // 불가능
```

대신 래퍼 클래스를 사용해야 한다.

```java
GenericBox<Integer> box; // 가능
```

즉, `int`, `double`, `boolean` 같은 기본형 대신 `Integer`, `Double`, `Boolean`을 사용한다.

---

## 4. 제네릭 타입 제한

제네릭은 기본적으로 어떤 타입이든 받을 수 있다.

하지만 어떤 경우에는 특정 부모 타입과 그 자식 타입만 받도록 제한해야 한다.

예를 들어 동물 병원 예제를 생각해볼 수 있다.

개 병원은 개를 받고, 고양이 병원은 고양이를 받는다.

처음에는 `DogHospital`, `CatHospital`을 각각 만들 수 있지만, 코드 중복이 생긴다.

다형성을 사용해 `Animal` 타입으로 처리할 수도 있지만, 이 경우 타입 안전성이 떨어질 수 있다.

제네릭을 사용하되 타입을 제한하면 이 문제를 해결할 수 있다.

```java
public class AnimalHospital<T extends Animal> {
    private T animal;

    public void set(T animal) {
        this.animal = animal;
    }

    public void checkup() {
        animal.sound();
    }
}
```

`T extends Animal`은 `T`에 `Animal` 또는 `Animal`의 자식 타입만 올 수 있다는 뜻이다.

이렇게 하면 다음과 같은 장점이 있다.

- 동물과 관계없는 타입은 컴파일 시점에 막을 수 있다.
- Animal이 제공하는 메서드를 사용할 수 있다.
- 코드 재사용성과 타입 안전성을 함께 얻을 수 있다.

---

## 5. 제네릭 메서드

제네릭 메서드는 메서드 자체에 타입 매개변수를 선언하는 방식이다.

```java
public static <T> T genericMethod(T value) {
    return value;
}
```

제네릭 타입은 객체를 생성하는 시점에 타입이 정해진다.

하지만 제네릭 메서드는 메서드를 호출하는 시점에 타입이 정해진다.

static 메서드에서는 클래스의 타입 매개변수를 사용할 수 없기 때문에, static 메서드에서 제네릭이 필요하면 제네릭 메서드를 사용해야 한다.

```java
class Box<T> {
    // static T method(T value) {} // 불가능

    static <V> V staticMethod(V value) {
        return value;
    }
}
```

---

## 6. 타입 이레이저

Java 제네릭은 컴파일 시점에 타입 안전성을 검사한다.

하지만 컴파일이 끝나면 제네릭 타입 정보는 대부분 제거된다.

이를 타입 이레이저라고 한다.

예를 들어 다음 코드는 컴파일 시점에는 `Integer` 타입으로 검사된다.

```java
GenericBox<Integer> box = new GenericBox<>();
box.set(10);
Integer result = box.get();
```

하지만 컴파일 이후에는 내부적으로 Object 기반 코드와 필요한 캐스팅으로 바뀐다고 이해할 수 있다.

```java
GenericBox box = new GenericBox();
box.set(10);
Integer result = (Integer) box.get();
```

중요한 점은 캐스팅을 개발자가 직접 작성하는 것이 아니라, 컴파일러가 제네릭 정보를 바탕으로 안전하게 추가한다는 것이다.

---

## 7. 컬렉션 프레임워크

컬렉션 프레임워크는 여러 데이터를 저장하고 다루기 위한 표준 자료구조와 인터페이스를 제공한다.

대표적인 인터페이스는 다음과 같다.

- `List`
- `Set`
- `Queue`
- `Map`

단, `Map`은 key-value 구조를 가지며 `Collection` 인터페이스를 상속하지 않는다.

컬렉션 프레임워크를 사용하면 직접 자료구조를 모두 구현하지 않아도 되고, 상황에 맞는 구현체를 선택할 수 있다.

---

## 8. ArrayList

ArrayList는 내부적으로 배열을 사용하는 List 구현체이다.

배열은 인덱스를 사용하면 빠르게 접근할 수 있다.

```java
list.get(0);
```

인덱스 조회는 빠르지만, 중간에 데이터를 추가하거나 삭제하면 뒤의 데이터를 이동해야 한다.

ArrayList의 특징은 다음과 같다.

- 인덱스 조회가 빠르다.
- 마지막에 추가하는 작업은 비교적 빠르다.
- 중간 삽입/삭제는 데이터 이동이 필요하다.
- 내부 배열 크기가 부족하면 더 큰 배열로 확장한다.

ArrayList는 조회가 많고, 중간 삽입/삭제가 적은 경우에 적합하다.

---

## 9. LinkedList

LinkedList는 노드들이 서로 연결된 구조이다.

각 노드는 데이터와 다음 노드의 참조를 가진다.

```java
class Node {
    Object item;
    Node next;
}
```

LinkedList는 배열처럼 연속된 메모리 공간에 데이터를 저장하지 않는다.

대신 노드들이 서로 다음 노드를 가리키는 방식으로 연결된다.

특징은 다음과 같다.

- 노드 추가/삭제 시 참조만 바꾸면 되는 경우가 있다.
- 중간 삽입/삭제에 유리할 수 있다.
- 특정 인덱스 조회는 처음부터 순차 탐색해야 하므로 느릴 수 있다.
- 각 노드가 다음 참조를 가지므로 추가 메모리가 필요하다.

---

## 10. List 인터페이스

List는 순서가 있고 중복을 허용하는 자료구조이다.

ArrayList와 LinkedList는 내부 구현은 다르지만, 사용자 입장에서는 List라는 같은 기능을 제공한다.

그래서 인터페이스를 사용하면 구현체를 교체하기 쉬워진다.

```java
List<String> list = new ArrayList<>();
```

필요하면 구현체를 바꿀 수 있다.

```java
List<String> list = new LinkedList<>();
```

이렇게 인터페이스에 의존하면 코드가 특정 구현체에 강하게 묶이지 않는다.

---

## 11. Set

Set은 중복을 허용하지 않는 자료구조이다.

List와 다르게 인덱스가 없다.

Set은 보통 다음과 같은 경우에 사용한다.

- 중복을 제거하고 싶을 때
- 특정 값이 존재하는지 빠르게 확인하고 싶을 때
- 순서보다 유일성이 중요할 때

대표 구현체는 다음과 같다.

- `HashSet`
- `LinkedHashSet`
- `TreeSet`

---

## 12. Hash와 HashSet

HashSet은 해시 알고리즘을 사용해서 데이터를 빠르게 저장하고 조회한다.

해시 알고리즘은 객체의 `hashCode()` 값을 이용해 저장 위치를 계산한다.

하지만 서로 다른 값이 같은 해시 인덱스를 가질 수 있다.

이를 해시 충돌이라고 한다.

해시 충돌이 발생하면 같은 위치에 여러 데이터를 보관하고, 실제 같은 값인지 `equals()`로 비교한다.

따라서 직접 만든 객체를 HashSet에 넣을 때는 `equals()`와 `hashCode()`를 함께 재정의해야 한다.

```java
@Override
public boolean equals(Object o) {
    // 값 비교
}

@Override
public int hashCode() {
    // 같은 값이면 같은 해시 코드 반환
}
```

`hashCode()`만 재정의하고 `equals()`를 재정의하지 않으면 같은 값이라고 판단하지 못할 수 있다.

반대로 `equals()`만 재정의하고 `hashCode()`를 재정의하지 않아도 해시 기반 자료구조에서 문제가 생길 수 있다.

정리하면 다음과 같다.

- `hashCode()`: 어느 위치에 저장할지 찾는 데 사용
- `equals()`: 실제 같은 값인지 비교하는 데 사용

---

## 13. Set 구현체 선택

Set 구현체는 상황에 따라 선택한다.

### HashSet

- 가장 많이 사용한다.
- 순서를 보장하지 않는다.
- 평균적으로 빠른 검색 성능을 제공한다.

### LinkedHashSet

- 입력 순서를 유지한다.
- 중복은 허용하지 않는다.

### TreeSet

- 정렬된 순서를 유지한다.
- 정렬 기준이 필요하다.

실무에서는 보통 `HashSet`을 먼저 고려하고, 입력 순서가 필요하면 `LinkedHashSet`, 정렬이 필요하면 `TreeSet`을 고려할 수 있다.

---

## 14. Map

Map은 key-value 쌍으로 데이터를 저장하는 자료구조이다.

```java
Map<String, Integer> map = new HashMap<>();
map.put("Java", 10000);
```

Map의 특징은 다음과 같다.

- key는 중복될 수 없다.
- value는 중복될 수 있다.
- key를 통해 value를 빠르게 조회할 수 있다.
- 순서를 기본적으로 보장하지 않는다.

대표 구현체는 다음과 같다.

- `HashMap`
- `LinkedHashMap`
- `TreeMap`

Map은 회원 ID로 회원 정보를 찾거나, 상품명으로 가격을 찾는 것처럼 key를 기준으로 값을 조회할 때 유용하다.

---

## 15. Stack, Queue, Deque

Stack은 후입선출 구조이다.

마지막에 넣은 데이터가 가장 먼저 나온다.

- LIFO
- Last In First Out

Queue는 선입선출 구조이다.

먼저 넣은 데이터가 먼저 나온다.

- FIFO
- First In First Out

Java의 오래된 `Stack` 클래스는 내부적으로 `Vector`를 사용하므로 권장되지 않는다.

Stack 구조가 필요하면 보통 `Deque`와 `ArrayDeque`를 사용하는 것이 좋다.

```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.pop();
```

Queue 기능도 Deque로 처리할 수 있다.

```java
Deque<Integer> queue = new ArrayDeque<>();
queue.offer(1);
queue.poll();
```

---

## 16. Iterable과 Iterator

순회는 자료구조 안의 데이터를 차례대로 접근하는 것이다.

자료구조마다 내부 구조가 다르기 때문에 직접 순회 방법을 모두 알기는 어렵다.

Java는 이를 해결하기 위해 `Iterable`과 `Iterator`를 제공한다.

```java
public interface Iterable<T> {
    Iterator<T> iterator();
}
```

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
}
```

`hasNext()`는 다음 요소가 있는지 확인한다.

`next()`는 다음 요소를 반환한다.

이 구조 덕분에 내부 구조가 다른 컬렉션도 향상된 for문으로 순회할 수 있다.

```java
for (String value : list) {
    System.out.println(value);
}
```

---

## 17. Comparable과 Comparator

객체를 정렬하려면 정렬 기준이 필요하다.

Java는 정렬 기준을 추상화하기 위해 `Comparable`과 `Comparator`를 제공한다.

### Comparable

객체 자신이 기본 정렬 기준을 가지는 경우 사용한다.

```java
public class User implements Comparable<User> {
    @Override
    public int compareTo(User other) {
        return this.age - other.age;
    }
}
```

### Comparator

기본 정렬 기준이 아닌 별도 정렬 기준을 제공할 때 사용한다.

```java
Comparator<User> idComparator = (u1, u2) -> u1.getId().compareTo(u2.getId());
```

정리하면 다음과 같다.

- `Comparable`: 객체의 기본 정렬 기준
- `Comparator`: 외부에서 제공하는 별도 정렬 기준

---

## 18. 오늘 정리

오늘 학습한 제네릭과 컬렉션 프레임워크는 Java에서 데이터를 안전하고 효율적으로 다루기 위한 핵심 기반이다.

제네릭은 타입 안전성과 코드 재사용을 동시에 제공한다.

컬렉션 프레임워크는 상황에 맞는 자료구조를 선택할 수 있게 해준다.

중요한 기준은 다음과 같다.

- 순서가 필요하고 중복을 허용하면 List
- 중복을 제거해야 하면 Set
- key-value 구조가 필요하면 Map
- 먼저 넣은 데이터를 먼저 꺼내야 하면 Queue
- 마지막에 넣은 데이터를 먼저 꺼내야 하면 Stack 또는 Deque
- 정렬 기준이 필요하면 Comparable 또는 Comparator
- 해시 기반 자료구조에서 직접 만든 객체를 사용하면 equals와 hashCode를 함께 고려

앞으로 Java 코드를 작성할 때는 단순히 컬렉션을 사용하는 것에서 끝내지 않고, 각 자료구조의 내부 구조와 성능 특성을 고려해서 선택해야겠다.
