# Java Stream API, Optional, 병렬 스트림 정리

## 학습 날짜

2026-05-13

## 학습 주제

- Stream API 기본
- 스트림 생성
- 중간 연산
- 최종 연산
- 지연 연산
- flatMap
- Collectors
- 다운스트림 컬렉터
- Optional
- 디폴트 메서드
- 병렬 스트림
- 함수형 프로그래밍 관점

---

## 1. Stream API란

Stream API는 컬렉션이나 배열 같은 데이터 소스를 파이프라인 형태로 처리할 수 있게 해주는 기능이다.

여기서 Stream은 파일 입출력에서 사용하는 I/O Stream과 다른 개념이다.

Stream API를 사용하면 반복문을 직접 작성하지 않고도 다음과 같은 작업을 할 수 있다.

- 필터링
- 변환
- 정렬
- 중복 제거
- 집계
- 그룹화
- 수집

예를 들어 문자열 목록에서 `B`로 시작하는 값만 골라 대문자로 바꿀 수 있다.

```java
List<String> result = names.stream()
        .filter(name -> name.startsWith("B"))
        .map(String::toUpperCase)
        .toList();
```

이 코드는 “어떻게 반복할지”보다 “무엇을 할지”가 잘 드러난다.

이런 스타일을 선언형 프로그래밍에 가깝다고 볼 수 있다.

---

## 2. Stream 처리 흐름

Stream은 보통 다음 흐름으로 사용한다.

```text
데이터 소스
→ 스트림 생성
→ 중간 연산
→ 최종 연산
```

예시는 다음과 같다.

```java
List<String> result = names.stream()
        .filter(name -> name.startsWith("B"))
        .map(String::toUpperCase)
        .toList();
```

여기서 각 단계는 다음과 같다.

- `names.stream()`: 스트림 생성
- `filter(...)`: 중간 연산
- `map(...)`: 중간 연산
- `toList()`: 최종 연산

중간 연산은 여러 개 연결할 수 있지만, 최종 연산은 보통 마지막에 한 번 수행된다.

---

## 3. 스트림 생성

Stream은 여러 방법으로 만들 수 있다.

### 컬렉션에서 생성

```java
List<String> list = List.of("a", "b", "c");
Stream<String> stream = list.stream();
```

### 배열에서 생성

```java
String[] arr = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
```

### 직접 값으로 생성

```java
Stream<String> stream = Stream.of("a", "b", "c");
```

### 무한 스트림

```java
Stream<Integer> evenNumbers = Stream.iterate(0, n -> n + 2);
```

```java
Stream<Double> randomNumbers = Stream.generate(Math::random);
```

무한 스트림은 종료 조건이 없으면 계속 생성되므로 `limit()` 같은 연산으로 범위를 제한해야 한다.

```java
Stream.iterate(0, n -> n + 2)
        .limit(5)
        .forEach(System.out::println);
```

---

## 4. 중간 연산

중간 연산은 Stream 데이터를 가공하는 단계이다.

중간 연산은 바로 실행되지 않고, 최종 연산이 호출될 때 함께 실행된다.

대표적인 중간 연산은 다음과 같다.

### filter

조건에 맞는 요소만 남긴다.

```java
numbers.stream()
        .filter(n -> n > 5)
        .toList();
```

### map

요소를 다른 값으로 변환한다.

```java
names.stream()
        .map(String::toUpperCase)
        .toList();
```

### distinct

중복을 제거한다.

```java
numbers.stream()
        .distinct()
        .toList();
```

### sorted

정렬한다.

```java
numbers.stream()
        .sorted()
        .toList();
```

### limit

앞에서부터 지정한 개수만큼만 사용한다.

```java
numbers.stream()
        .limit(3)
        .toList();
```

### skip

앞에서부터 지정한 개수만큼 건너뛴다.

```java
numbers.stream()
        .skip(2)
        .toList();
```

---

## 5. 최종 연산

최종 연산은 Stream 파이프라인을 실제로 실행하고 결과를 만든다.

대표적인 최종 연산은 다음과 같다.

### toList

결과를 리스트로 만든다.

```java
List<String> result = names.stream()
        .filter(name -> name.startsWith("B"))
        .toList();
```

### forEach

각 요소에 대해 동작을 수행한다.

```java
names.stream()
        .forEach(System.out::println);
```

### count

요소 개수를 반환한다.

```java
long count = names.stream()
        .filter(name -> name.startsWith("B"))
        .count();
```

### reduce

여러 요소를 하나의 결과로 합친다.

```java
int sum = numbers.stream()
        .reduce(0, (a, b) -> a + b);
```

### collect

결과를 다양한 형태로 수집한다.

```java
List<String> result = names.stream()
        .collect(Collectors.toList());
```

최종 연산이 호출되면 그때 중간 연산들이 실제로 동작한다.

---

## 6. 지연 연산

Stream의 중간 연산은 바로 실행되지 않는다.

이를 지연 연산이라고 한다.

```java
Stream<Integer> stream = numbers.stream()
        .filter(n -> {
            System.out.println("filter: " + n);
            return n % 2 == 0;
        });
```

위 코드까지만 실행하면 `filter` 안의 출력문은 실행되지 않는다.

최종 연산이 호출되어야 실제 연산이 시작된다.

```java
List<Integer> result = stream.toList();
```

지연 연산의 장점은 불필요한 연산을 줄일 수 있다는 것이다.

예를 들어 `limit()`과 함께 사용하면 필요한 개수만 찾고 나머지 연산을 생략할 수 있다.

---

## 7. Stream은 재사용할 수 없다

Stream은 한 번 최종 연산을 수행하면 다시 사용할 수 없다.

```java
Stream<String> stream = names.stream();

stream.toList();
// stream.count(); // 오류 발생 가능
```

다시 처리하려면 새 Stream을 생성해야 한다.

```java
names.stream().toList();
names.stream().count();
```

Stream은 데이터 자체를 저장하는 컬렉션이 아니라, 데이터를 처리하는 흐름이라는 점을 기억해야 한다.

---

## 8. flatMap

`map`은 하나의 요소를 하나의 값으로 변환할 때 사용한다.

하지만 하나의 요소가 여러 값으로 펼쳐져야 하는 경우에는 `flatMap`을 사용한다.

예를 들어 문장 목록에서 단어 목록을 만들고 싶다고 하자.

```java
List<String> sentences = List.of("hello java", "stream api");
```

`map`을 사용하면 각 문장이 단어 배열로 변환되어 중첩 구조가 생길 수 있다.

`flatMap`을 사용하면 여러 Stream을 하나의 Stream으로 평평하게 펼칠 수 있다.

```java
List<String> words = sentences.stream()
        .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
        .toList();
```

정리하면 다음과 같다.

- `map`: 요소를 다른 요소로 변환
- `flatMap`: 요소를 Stream으로 변환한 뒤 하나로 펼침

---

## 9. Collectors

`Collectors`는 Stream 결과를 다양한 형태로 모을 때 사용한다.

대표적인 수집 방식은 다음과 같다.

### List로 수집

```java
List<String> list = stream.collect(Collectors.toList());
```

### Set으로 수집

```java
Set<String> set = stream.collect(Collectors.toSet());
```

### 특정 컬렉션으로 수집

```java
Set<String> treeSet = stream.collect(Collectors.toCollection(TreeSet::new));
```

### Map으로 수집

```java
Map<Long, String> map = users.stream()
        .collect(Collectors.toMap(User::getId, User::getName));
```

Map으로 수집할 때 key가 중복되면 예외가 발생할 수 있다.

중복 key가 가능하다면 병합 함수를 함께 지정해야 한다.

```java
Map<String, Integer> map = users.stream()
        .collect(Collectors.toMap(
                User::getName,
                User::getAge,
                (oldValue, newValue) -> newValue
        ));
```

---

## 10. groupingBy

`groupingBy`는 특정 기준으로 데이터를 그룹화할 때 사용한다.

예를 들어 학생을 학년별로 묶을 수 있다.

```java
Map<Integer, List<Student>> result = students.stream()
        .collect(Collectors.groupingBy(Student::getGrade));
```

기본적으로 그룹 안의 값은 `List`로 모인다.

그룹화 후 각 그룹 내부에서 추가 처리를 하고 싶다면 다운스트림 컬렉터를 사용한다.

```java
Map<Integer, Long> countByGrade = students.stream()
        .collect(Collectors.groupingBy(
                Student::getGrade,
                Collectors.counting()
        ));
```

이 코드는 학년별 학생 수를 구한다.

---

## 11. partitioningBy

`partitioningBy`는 조건 결과에 따라 데이터를 `true`, `false` 두 그룹으로 나눌 때 사용한다.

```java
Map<Boolean, List<Student>> result = students.stream()
        .collect(Collectors.partitioningBy(student -> student.getScore() >= 60));
```

결과는 `Map<Boolean, List<Student>>` 형태이다.

예를 들어 합격자와 불합격자를 나눌 때 사용할 수 있다.

---

## 12. 다운스트림 컬렉터

다운스트림 컬렉터는 `groupingBy`나 `partitioningBy` 이후 각 그룹 내부를 어떻게 처리할지 지정하는 Collector이다.

예를 들어 학년별 학생 수를 구할 수 있다.

```java
Map<Integer, Long> countByGrade = students.stream()
        .collect(Collectors.groupingBy(
                Student::getGrade,
                Collectors.counting()
        ));
```

학년별 점수 합계를 구할 수도 있다.

```java
Map<Integer, Integer> scoreSumByGrade = students.stream()
        .collect(Collectors.groupingBy(
                Student::getGrade,
                Collectors.summingInt(Student::getScore)
        ));
```

학년별 학생 이름 목록만 모으고 싶다면 `mapping`을 사용할 수 있다.

```java
Map<Integer, List<String>> namesByGrade = students.stream()
        .collect(Collectors.groupingBy(
                Student::getGrade,
                Collectors.mapping(Student::getName, Collectors.toList())
        ));
```

다운스트림 컬렉터를 이해하면 그룹화 이후의 요구사항을 더 유연하게 처리할 수 있다.

---

## 13. Optional이 필요한 이유

Java에서 `null`은 값이 없음을 표현할 때 자주 사용된다.

하지만 `null`에 대해 메서드를 호출하면 `NullPointerException`이 발생할 수 있다.

```java
String name = findNameById(id);
System.out.println(name.toUpperCase()); // name이 null이면 NPE
```

`Optional`은 값이 있을 수도 있고 없을 수도 있음을 명시적으로 표현한다.

```java
Optional<String> optionalName = findNameById(id);
```

반환 타입이 `Optional<String>`이면 호출하는 쪽에서 “값이 없을 수도 있구나”를 알 수 있다.

---

## 14. Optional 생성

값이 반드시 있을 때는 `Optional.of()`를 사용할 수 있다.

```java
Optional<String> opt = Optional.of("Java");
```

값이 null일 수도 있다면 `Optional.ofNullable()`을 사용한다.

```java
Optional<String> opt = Optional.ofNullable(findName);
```

빈 Optional은 다음처럼 만들 수 있다.

```java
Optional<String> empty = Optional.empty();
```

`Optional.of(null)`은 예외가 발생하므로 주의해야 한다.

---

## 15. Optional 값 처리

Optional에서 값을 처리하는 대표적인 방법은 다음과 같다.

### orElse

값이 있으면 값을 반환하고, 없으면 대체값을 반환한다.

```java
String name = optionalName.orElse("UNKNOWN");
```

### orElseGet

값이 없을 때만 대체값을 생성한다.

```java
String name = optionalName.orElseGet(() -> createDefaultName());
```

`orElse()`는 대체값이 즉시 평가될 수 있고, `orElseGet()`은 값이 없을 때만 Supplier가 실행된다.

대체값 생성 비용이 크거나 메서드 호출이 필요하다면 `orElseGet()`이 더 적절할 수 있다.

### ifPresent

값이 있을 때만 동작을 실행한다.

```java
optionalName.ifPresent(System.out::println);
```

### map

값이 있으면 변환한다.

```java
Optional<Integer> length = optionalName.map(String::length);
```

### flatMap

이미 Optional을 반환하는 메서드와 연결할 때 사용한다.

```java
Optional<Address> address = optionalUser.flatMap(User::getAddress);
```

---

## 16. Optional 사용 시 주의할 점

Optional은 null을 완전히 없애는 마법 같은 도구가 아니다.

값이 없을 수 있음을 명시적으로 표현하기 위한 도구이다.

일반적으로 Optional은 반환 타입에 사용하는 것이 적절하다.

다음과 같은 사용은 주의해야 한다.

- 필드에 Optional을 저장
- 메서드 매개변수로 Optional을 받기
- 컬렉션을 Optional로 감싸기
- `get()`을 무조건 호출하기

특히 `optional.get()`은 값이 없으면 예외가 발생하므로, 값이 있는지 확인하지 않고 사용하는 것은 좋지 않다.

---

## 17. 디폴트 메서드

디폴트 메서드는 인터페이스에 기본 구현을 제공하는 기능이다.

Java 8 이전에는 인터페이스에 메서드를 추가하면 모든 구현체가 그 메서드를 구현해야 했다.

이미 많은 클래스가 구현한 인터페이스에 새 메서드를 추가하면 기존 코드가 깨질 수 있었다.

디폴트 메서드는 이런 문제를 줄이기 위해 등장했다.

```java
public interface Notifier {
    void notify(String message);

    default void notifyWelcome() {
        notify("서비스 가입을 환영합니다.");
    }
}
```

기존 구현체는 `notifyWelcome()`을 직접 구현하지 않아도 기본 구현을 사용할 수 있다.

다만 디폴트 메서드를 너무 많이 사용하면 인터페이스가 단순한 계약을 넘어 구현 로직을 많이 가지게 될 수 있다.

따라서 기존 인터페이스에 기능을 안전하게 추가해야 할 때 사용하는 것이 좋다.

---

## 18. 병렬 스트림

병렬 스트림은 Stream 작업을 여러 스레드에서 나누어 처리하는 기능이다.

```java
int sum = numbers.parallelStream()
        .mapToInt(HeavyJob::heavyTask)
        .sum();
```

병렬 스트림은 내부적으로 Fork/Join 공용 풀을 사용한다.

작업을 나누고, 여러 스레드가 나누어진 작업을 처리한 뒤 결과를 합치는 구조이다.

CPU 연산이 많고 각 작업이 서로 독립적이라면 성능 향상에 도움이 될 수 있다.

하지만 항상 빠른 것은 아니다.

---

## 19. 병렬 스트림 사용 시 주의할 점

병렬 스트림은 다음 상황에서 주의해야 한다.

### I/O 작업이 많은 경우

파일, 네트워크, DB 같은 I/O 작업은 CPU 계산보다 대기 시간이 길다.

이런 작업을 병렬 스트림으로 처리하면 공용 풀의 스레드가 대기 상태가 되어 다른 작업에 영향을 줄 수 있다.

### 공유 상태를 변경하는 경우

여러 스레드가 동시에 같은 컬렉션이나 변수를 변경하면 동시성 문제가 발생할 수 있다.

```java
List<Integer> result = new ArrayList<>();

numbers.parallelStream()
        .forEach(result::add); // 위험할 수 있음
```

### 작업이 너무 작은 경우

작업을 나누고 합치는 비용이 실제 작업 비용보다 크면 오히려 느려질 수 있다.

### 공용 풀을 사용하는 경우

병렬 스트림은 기본적으로 공용 풀을 사용하므로, 애플리케이션의 다른 병렬 작업과 영향을 주고받을 수 있다.

정리하면 병렬 스트림은 CPU 중심의 독립적인 작업에 적합하고, I/O 대기나 공유 상태 변경이 있는 작업에는 신중해야 한다.

---

## 20. 함수형 프로그래밍 관점

함수형 프로그래밍은 프로그램을 함수의 조합으로 바라보는 방식이다.

중요한 특징은 다음과 같다.

- 순수 함수
- 불변성
- 부수 효과 최소화
- 선언형 스타일
- 고차 함수

Stream API는 함수형 프로그래밍 스타일을 Java에서 사용할 수 있게 도와준다.

예를 들어 반복문을 직접 작성하는 대신, `filter`, `map`, `collect`를 조합해 원하는 결과를 표현한다.

```java
List<String> names = students.stream()
        .filter(student -> student.getScore() >= 80)
        .map(Student::getName)
        .toList();
```

이 코드는 “학생을 반복하면서 조건을 검사하고 이름을 리스트에 추가한다”라는 과정 대신, “80점 이상 학생의 이름 목록을 얻는다”라는 의도를 드러낸다.

---

## 21. 오늘 정리

Stream API는 컬렉션 데이터를 더 선언적으로 처리할 수 있게 해준다.

핵심은 다음과 같다.

- Stream은 데이터 처리 파이프라인이다.
- 중간 연산은 바로 실행되지 않고 최종 연산 시점에 동작한다.
- `filter`는 걸러내고, `map`은 변환하고, `flatMap`은 펼친다.
- `Collectors`는 결과를 다양한 자료구조나 통계로 수집한다.
- `groupingBy`와 다운스트림 컬렉터를 사용하면 그룹별 집계가 가능하다.
- `Optional`은 값이 없을 수 있음을 명시적으로 표현한다.
- 디폴트 메서드는 기존 구현체를 깨지 않고 인터페이스에 기능을 추가할 때 유용하다.
- 병렬 스트림은 CPU 중심 작업에는 도움이 될 수 있지만, I/O 작업이나 공유 상태가 있는 경우에는 주의해야 한다.

앞으로 Stream을 사용할 때는 단순히 반복문을 줄이는 데만 집중하지 말고, 코드의 의도가 더 잘 드러나는지와 성능상 주의할 점을 함께 고려해야겠다.
