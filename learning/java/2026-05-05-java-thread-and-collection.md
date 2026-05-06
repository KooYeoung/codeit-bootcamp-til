# Java 스레드와 동기화 기초

## 학습 날짜

2026-05-05

## 학습 주제

- 멀티스레드 기본 이론
- Process와 Thread
- Runnable과 Thread
- 스레드 상태와 생명주기
- Daemon Thread
- sleep, interrupt, join
- JVM과 스레드
- Java 메모리 모델
- volatile
- Race Condition
- synchronized
- Monitor Lock
- AtomicInteger
- ReentrantLock
- wait, notify
- LockSupport
- 연결 리스트와 동기화
- 제네릭
- 컬렉션 프레임워크

---

## 1. 멀티스레드 기본 개념

멀티스레드는 하나의 프로세스 안에서 여러 실행 흐름이 동시에 동작하는 구조이다.

프로세스는 실행 중인 프로그램이고, 스레드는 프로세스 안에서 실행되는 개별적인 연산 흐름이다.

하나의 프로세스는 최소 하나 이상의 스레드를 가진다.

Java 프로그램도 JVM이라는 프로세스 위에서 실행되며, Java 스레드는 JVM 안에서 실행되는 흐름으로 볼 수 있다.

멀티스레딩을 사용하는 이유는 다음과 같다.

- CPU 사용 효율을 높일 수 있음
- 느린 입출력 작업을 별도 흐름으로 분리할 수 있음
- 사용자 입력, 파일 처리, 네트워크 처리 등을 동시에 다룰 수 있음
- 여러 작업을 병렬 또는 동시적으로 처리할 수 있음

다만 여러 스레드가 같은 데이터를 공유하면 동기화 문제가 발생할 수 있다.

---

## 2. Process와 Thread

### Process

프로세스는 실행 중인 프로그램이다.

운영체제는 프로세스 단위로 가상 메모리 공간과 접근 권한을 제공한다.

각 프로세스는 기본적으로 독립된 메모리 공간을 가진다.

### Thread

스레드는 프로세스 안에서 실행되는 최소 실행 흐름이다.

하나의 프로세스 안에 여러 스레드가 존재할 수 있다.

스레드들은 같은 프로세스의 메모리 공간을 공유한다.

정리하면 다음과 같다.

- 프로세스: 실행 중인 프로그램
- 스레드: 프로세스 안에서 실행되는 개별 흐름
- 하나의 프로세스에는 최소 하나의 스레드가 존재
- 여러 스레드는 같은 프로세스의 메모리 공간을 공유

---

## 3. 멀티스레딩과 멀티프로세싱

### 멀티스레딩

멀티스레딩은 하나의 프로세스 안에서 여러 스레드가 동작하는 방식이다.

장점은 다음과 같다.

- 같은 메모리 공간을 공유하므로 데이터 공유가 쉬움
- 프로세스 간 통신보다 상대적으로 가벼움
- 입출력 대기 중에도 다른 작업을 수행할 수 있음

단점은 다음과 같다.

- 하나의 스레드 오류가 프로세스 전체에 영향을 줄 수 있음
- 공유 데이터 접근 시 동기화 문제가 발생할 수 있음

### 멀티프로세싱

멀티프로세싱은 여러 프로세스가 각각 독립적으로 동작하는 방식이다.

장점은 다음과 같다.

- 프로세스별 메모리 공간이 분리됨
- 하나의 프로세스 문제가 다른 프로세스에 영향을 덜 줌

단점은 다음과 같다.

- 프로세스 간 통신이 필요함
- 스레드보다 상대적으로 무거울 수 있음

---

## 4. Java에서 스레드 생성

Java에서 스레드를 만드는 대표적인 방법은 두 가지이다.

- `Thread` 클래스 상속
- `Runnable` 인터페이스 구현

### Runnable 구현

`Runnable`은 스레드가 실행할 작업을 정의하는 인터페이스이다.

`run()` 메서드 안에 스레드가 수행할 작업을 작성한다.

예시:

```java
class MyThread implements Runnable {
    @Override
    public void run() {
        System.out.println("MyThread.run()");
    }
}
```

실행할 때는 `Thread` 객체에 `Runnable` 구현체를 전달하고 `start()`를 호출한다.

```java
Runnable task = new MyThread();
Thread thread = new Thread(task);
thread.start();
```

### start()와 run()의 차이

`start()`는 새로운 스레드를 시작한다.

`run()`은 스레드가 실행할 작업 내용이다.

중요한 점은 `run()`을 직접 호출하면 새로운 스레드가 생성되지 않는다는 것이다.

정리하면 다음과 같다.

- `start()`: 새로운 스레드 생성 후 실행
- `run()`: 스레드가 실행할 작업 내용
- `run()` 직접 호출: 일반 메서드 호출

---

## 5. 스레드 속성

스레드는 여러 속성을 가진다.

대표적인 속성은 다음과 같다.

- ID
- 이름
- 우선순위
- 상태
- 그룹

스레드 이름은 디버깅이나 로그 확인 시 유용하다.

현재 실행 중인 스레드는 다음과 같이 확인할 수 있다.

```java
Thread currentThread = Thread.currentThread();
```

스레드 이름은 `setName()`으로 지정할 수 있고, `getName()`으로 확인할 수 있다.

---

## 6. 스레드 우선순위

Java는 스레드 우선순위를 제공한다.

대표적인 상수는 다음과 같다.

- `Thread.MAX_PRIORITY`
- `Thread.NORM_PRIORITY`
- `Thread.MIN_PRIORITY`

우선순위가 높다고 해서 반드시 먼저 실행되는 것은 아니다.

실제 스케줄링은 JVM과 운영체제의 정책에 영향을 받는다.

따라서 스레드 우선순위에 의존해서 프로그램의 정확한 실행 순서를 보장하려고 하면 안 된다.

---

## 7. 스레드 상태와 생명주기

Java 스레드의 대표적인 상태는 다음과 같다.

- `NEW`
- `RUNNABLE`
- `BLOCKED`
- `WAITING`
- `TIMED_WAITING`
- `TERMINATED`

### NEW

스레드 객체가 생성되었지만 아직 `start()`가 호출되지 않은 상태이다.

### RUNNABLE

스레드가 실행 가능한 상태이다.

실제로 CPU를 사용 중일 수도 있고, CPU 할당을 기다리고 있을 수도 있다.

### BLOCKED

Lock을 얻기 위해 대기하는 상태이다.

예를 들어 다른 스레드가 이미 `synchronized` 영역을 실행 중이면, Lock을 기다리는 스레드는 `BLOCKED` 상태가 될 수 있다.

### WAITING

다른 스레드의 작업이나 알림을 기다리는 상태이다.

대표적으로 `wait()`, `join()`이 있다.

### TIMED_WAITING

정해진 시간 동안 대기하는 상태이다.

대표적으로 `sleep(ms)`, `wait(ms)`, `join(ms)`이 있다.

### TERMINATED

스레드의 `run()` 메서드가 끝나 실행이 종료된 상태이다.

---

## 8. Daemon Thread

Daemon Thread는 일반 스레드를 보조하는 스레드이다.

모든 일반 사용자 스레드가 종료되면 Daemon Thread는 함께 종료될 수 있다.

Daemon Thread는 `start()` 호출 전에 설정해야 한다.

```java
Thread thread = new Thread(task);
thread.setDaemon(true);
thread.start();
```

주로 백그라운드 작업이나 보조 작업에 사용된다.

---

## 9. sleep()

`sleep()`은 현재 실행 중인 스레드를 일정 시간 동안 멈춘다.

`sleep()` 중인 스레드는 `TIMED_WAITING` 상태가 된다.

```java
Thread.sleep(1000);
```

주의할 점은 `sleep()`이 정확한 실행 순서를 보장하지 않는다는 것이다.

지정한 시간보다 더 오래 대기할 수 있으며, 운영체제 스케줄링에 영향을 받는다.

또한 `sleep()`은 스레드를 잠시 멈추지만, 이미 획득한 Lock을 반납하지는 않는다.

따라서 스레드의 종료 순서나 실행 순서를 맞추기 위해 `sleep()`에 의존하는 코드는 위험하다.

---

## 10. interrupt()

`interrupt()`는 스레드에 인터럽트 신호를 보내는 메서드이다.

대기 중인 스레드가 `sleep()`, `wait()`, `join()` 상태라면 `InterruptedException`이 발생할 수 있다.

이를 이용해 스레드에게 종료 요청을 전달할 수 있다.

```java
thread.interrupt();
```

스레드를 강제로 즉시 죽이는 것보다, 스레드 내부에서 인터럽트 신호를 확인하고 안전하게 종료 흐름을 만드는 방식이 좋다.

---

## 11. join()

`join()`은 특정 스레드가 종료될 때까지 현재 스레드가 기다리도록 한다.

```java
thread.join();
```

예를 들어 main 스레드가 worker 스레드들의 작업이 끝난 뒤 결과를 출력해야 한다면 `join()`을 사용할 수 있다.

정리하면 다음과 같다.

- `sleep()`: 현재 스레드를 잠시 멈춤
- `interrupt()`: 스레드에 중단 요청 신호를 보냄
- `join()`: 다른 스레드가 끝날 때까지 기다림

---

## 12. JVM과 스레드

JVM은 사용자 모드에서 실행되는 하나의 프로세스이다.

Java 스레드는 JVM 안에서 실행되며, 운영체제 스케줄링의 영향을 받는다.

스레드마다 독립적으로 가지는 영역은 다음과 같다.

- Stack
- PC Register
- Native Method Stack

여러 스레드가 공유하는 영역은 다음과 같다.

- Heap
- Method Area

정리하면 다음과 같다.

- 객체는 Heap에 생성됨
- 메서드 호출 정보와 지역변수는 Stack과 관련됨
- 클래스 정보와 static 정보는 Method Area와 관련됨
- Stack은 스레드마다 따로 존재
- Heap은 여러 스레드가 공유

---

## 13. Context Switching

Context Switching은 CPU가 실행 중인 스레드를 바꾸는 과정이다.

현재 스레드의 실행 상태를 저장하고, 다른 스레드의 실행 상태를 복원해야 한다.

이 과정에는 비용이 발생한다.

스레드가 많아질수록 Context Switching 비용이 커질 수 있다.

따라서 스레드를 무조건 많이 만드는 것이 항상 좋은 것은 아니다.

---

## 14. Java 메모리 모델

Java 메모리 모델은 멀티스레드 환경에서 변수에 접근하는 규칙을 정의한다.

중요한 개념은 메인 메모리와 작업 메모리이다.

### 메인 메모리

공유 변수의 원본 값이 저장되는 공간이다.

### 작업 메모리

각 스레드가 사용하는 변수의 사본이 저장되는 공간이다.

스레드는 메인 메모리의 값을 직접 계속 사용하는 것이 아니라, 값을 작업 메모리로 가져와 연산할 수 있다.

작업 메모리의 변경이 메인 메모리에 즉시 반영되지 않을 수 있다.

이 때문에 한 스레드가 변경한 값을 다른 스레드가 바로 보지 못하는 문제가 발생할 수 있다.

---

## 15. volatile

`volatile`은 멀티스레드 환경에서 한 스레드가 바꾼 값을 다른 스레드가 볼 수 있게 하기 위해 사용한다.

`volatile`이 붙은 변수는 값을 읽고 쓸 때 다른 스레드가 변경 내용을 더 잘 확인할 수 있도록 동작한다.

예시:

```java
static volatile boolean exitFlag = false;
```

`volatile`을 사용하면 한 스레드에서 변경한 값을 다른 스레드가 확인할 수 있도록 도와준다.

이런 특징을 가시성이라고 부른다.

더 깊게 들어가면 값을 읽고 쓰는 순서 보장과도 관련이 있지만, 처음에는 "다른 스레드가 바뀐 값을 볼 수 있게 해준다"라고 이해해도 된다.

하지만 `volatile`은 원자성을 보장하지 않는다.

예를 들어 `counter++` 같은 복합 연산은 여전히 경쟁 조건이 발생할 수 있다.

정리하면 다음과 같다.

- `volatile`: 한 스레드에서 바꾼 값을 다른 스레드가 볼 수 있게 도와줌
- `volatile`: 값을 읽고 쓰는 순서 보장과 관련이 있음
- `volatile`: 복합 연산의 원자성은 보장하지 않음

---

## 16. Race Condition

Race Condition은 여러 스레드가 같은 공유 데이터에 동시에 접근하면서 결과가 실행 순서에 따라 달라지는 문제이다.

대표적인 예시는 `counter++`이다.

`counter++`는 한 줄처럼 보이지만 실제로는 다음 단계로 나뉜다.

```text
현재 값 읽기
→ 값 증가
→ 다시 저장
```

여러 스레드가 동시에 이 과정을 수행하면 증가 결과가 누락될 수 있다.

Race Condition은 컴파일 오류가 아니라 논리 오류로 나타나기 때문에 더 주의해야 한다.

---

## 17. synchronized

`synchronized`는 임계 영역을 만들어 여러 스레드가 동시에 접근하지 못하게 하는 키워드이다.

임계 영역은 여러 스레드가 동시에 실행하면 문제가 발생할 수 있는 코드 구간이다.

### synchronized 메서드

```java
public synchronized void method() {
    // 임계 영역
}
```

메서드 전체가 임계 영역이 된다.

### synchronized 블록

```java
synchronized (this) {
    // 임계 영역
}
```

필요한 코드 일부만 임계 영역으로 지정할 수 있다.

임계 영역은 가능한 작게 잡는 것이 좋다.

임계 영역이 커질수록 다른 스레드가 기다리는 시간이 늘어나 성능이 떨어질 수 있다.

---

## 18. Monitor Lock

Java의 `synchronized`는 객체의 Monitor Lock을 사용한다.

한 객체의 Monitor Lock을 얻을 수 있는 스레드는 한 번에 하나뿐이다.

동기화 구간에 들어가려는 스레드는 먼저 Monitor Lock을 획득해야 한다.

다른 스레드가 이미 Lock을 가지고 있다면, Lock을 얻으려는 스레드는 `BLOCKED` 상태가 될 수 있다.

중요한 점은 Lock의 기준이다.

- 인스턴스 synchronized 메서드: 해당 인스턴스의 Lock 사용
- static synchronized 메서드: 클래스 객체의 Lock 사용
- synchronized 블록: 괄호 안에 지정한 객체의 Lock 사용

동기화할 때는 실제로 여러 스레드가 공유하는 객체를 기준으로 Lock을 잡아야 한다.

---

## 19. AtomicInteger

`AtomicInteger`는 원자적 정수 연산을 제공하는 클래스이다.

대표적으로 `incrementAndGet()`을 사용할 수 있다.

```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();
```

`AtomicInteger`는 하나의 숫자 값을 안전하게 증가시키거나 감소시킬 때 사용할 수 있다.

`synchronized`처럼 스레드를 막는 방식이 아니라, Non-blocking 방식으로 동작할 수 있다.

단순 카운터처럼 원자적 연산이 필요한 경우 유용하다.

다만 여러 변수 값을 함께 안전하게 바꿔야 하는 작업까지 자동으로 해결하지는 못한다.

이런 경우에는 `synchronized`나 `ReentrantLock`으로 전체 작업 흐름을 보호해야 할 수 있다.

---

## 20. ReentrantLock

`ReentrantLock`은 명시적으로 Lock을 걸고 해제할 수 있는 동기화 도구이다.

```java
lock.lock();

try {
    // 임계 영역
} finally {
    lock.unlock();
}
```

`unlock()`은 반드시 실행되어야 하므로 보통 `finally` 블록에서 호출한다.

`ReentrantLock`은 `synchronized`보다 더 세밀한 제어가 가능하다.

예를 들어 `tryLock()`을 사용하면 일정 시간 동안 Lock 획득을 시도하고, 실패했을 때 다른 처리를 할 수 있다.

```java
if (lock.tryLock(1000, TimeUnit.MILLISECONDS)) {
    try {
        // 임계 영역
    } finally {
        lock.unlock();
    }
}
```

`tryLock()`은 Lock 획득 성공 여부를 반환하므로, Lock을 얻은 경우에만 `unlock()`을 호출해야 한다.

정리하면 다음과 같다.

- `synchronized`: 문법적으로 간단한 동기화
- `ReentrantLock`: Lock 획득과 해제를 명시적으로 제어
- `tryLock()`: Lock 획득 실패 시 다른 흐름 처리 가능

---

## 21. wait()와 notify()

`wait()`와 `notify()`는 스레드 간 대기와 알림을 처리하기 위한 메서드이다.

`wait()`는 현재 스레드를 대기 상태로 만든다.

`wait()`는 대기 상태로 들어가면서 현재 스레드가 가지고 있던 monitor lock을 반납한다.

`notify()`는 대기 중인 스레드 중 하나를 깨운다.

다만 `notify()`가 원하는 특정 스레드를 깨운다는 보장은 없으므로, 상황에 따라 `notifyAll()`이 더 안전할 수 있다.

이 메서드들은 `synchronized` 블록 안에서 사용해야 한다.

```java
synchronized (this) {
    while (!condition) {
        wait();
    }
}
```

`wait()`는 spurious wakeup 가능성이 있으므로 보통 `if`가 아니라 `while`로 조건을 다시 확인한다.

```java
synchronized (this) {
    notify();
}
```

잘못 사용하면 서로 영원히 기다리는 Deadlock이 발생할 수 있다.

---

## 22. Deadlock

Deadlock은 둘 이상의 스레드가 서로 상대방의 작업이나 Lock 해제를 기다리면서 더 이상 진행하지 못하는 상태이다.

스레드 동기화에서 가장 피해야 할 문제 중 하나이다.

Deadlock은 컴파일 오류가 아니라 실행 중 논리 오류로 나타나기 때문에 발견하기 어렵다.

Deadlock을 줄이려면 다음을 주의해야 한다.

- Lock 획득 순서를 일정하게 유지
- 불필요하게 긴 임계 영역 피하기
- wait/notify 흐름 명확히 설계
- Lock을 획득한 뒤 반드시 해제
- timeout이 있는 tryLock 고려

---

## 23. LockSupport

`LockSupport`는 스레드 제어를 위한 유틸리티 클래스이다.

대표 메서드는 다음과 같다.

- `park()`
- `unpark()`
- `parkNanos()`
- `parkUntil()`

### park()

현재 스레드를 대기 상태로 만든다.

### unpark()

특정 스레드를 다시 실행 가능 상태로 만든다.

`wait()`와 `notify()`는 `synchronized` 기반으로 작동하지만, `LockSupport`는 별도의 `synchronized` 블록 없이 사용할 수 있다.

스레드 대기와 재개를 더 유연하게 제어할 수 있다.

---

## 24. 연결 리스트

연결 리스트는 Node들이 참조를 통해 연결된 선형 자료구조이다.

배열과 달리 각 Node가 다음 Node를 참조한다.

양방향 연결 리스트에서는 이전 Node와 다음 Node를 모두 참조한다.

기본 구조는 다음과 같다.

```text
Node
- data
- prev
- next
```

연결 리스트는 노드의 참조를 바꾸는 방식으로 삽입과 삭제를 처리할 수 있다.

---

## 25. 연결 리스트와 동기화

여러 스레드가 동시에 연결 리스트에 노드를 추가하거나 삭제하면 문제가 발생할 수 있다.

예를 들어 한 스레드는 노드를 추가하고, 다른 스레드는 동시에 노드를 삭제하면 `prev`, `next` 참조 관계가 깨질 수 있다.

이런 공유 자료구조는 동기화가 필요하다.

동기화 방법은 다음과 같다.

- `synchronized` 메서드 사용
- `synchronized` 블록 사용
- `ReentrantLock` 사용

공유 자료구조를 다룰 때는 읽기, 추가, 삭제 작업이 동시에 실행될 수 있는지 항상 고려해야 한다.

---

## 26. Non-blocking 동기화

Blocking 동기화는 스레드를 멈추고 다시 실행 상태로 전환하는 비용이 발생한다.

Non-blocking 동기화는 스레드를 멈추지 않고 작업을 진행하면서 원자성을 보장하려는 방식이다.

대표적으로 Atomic 클래스가 있다.

Non-blocking 방식은 성능 면에서 유리할 수 있지만, 모든 상황에 적합한 것은 아니다.

단순 원자 연산인지, 복잡한 임계 영역인지에 따라 적절한 방식을 선택해야 한다.

---

## 27. 제네릭

제네릭은 클래스나 메서드에서 사용할 타입을 외부에서 지정할 수 있게 해주는 문법이다.

제네릭을 사용하면 컴파일 시점에 타입을 검사할 수 있다.

예를 들어 `Object`를 사용하면 다양한 타입을 담을 수 있지만, 꺼낼 때 형 변환이 필요하다.

형 변환 과정에서 런타임 오류가 발생할 수 있다.

제네릭을 사용하면 이런 문제를 줄일 수 있다.

```java
List<String> names = new ArrayList<>();
```

위 코드는 `names`에 문자열만 담겠다는 의미이다.

제네릭의 장점은 다음과 같다.

- 타입 안정성 향상
- 불필요한 형 변환 감소
- 컴파일 시점 오류 확인 가능
- 코드 재사용성 향상

대표적인 타입 매개변수 이름은 다음과 같다.

- `T`: Type
- `E`: Element
- `K`: Key
- `V`: Value

---

## 28. 컬렉션 프레임워크

컬렉션 프레임워크는 여러 데이터를 효율적으로 저장하고 다루기 위한 Java 표준 자료구조 모음이다.

대표적인 계열은 다음과 같다.

- `List`
- `Set`
- `Map`

### List

`List`는 순서가 있는 데이터 집합이다.

중복을 허용한다.

대표 구현체는 다음과 같다.

- `ArrayList`
- `LinkedList`

### Set

`Set`은 중복을 허용하지 않는 데이터 집합이다.

대표 구현체는 다음과 같다.

- `HashSet`
- `TreeSet`

### Map

`Map`은 Key와 Value 쌍으로 데이터를 저장한다.

Key는 중복될 수 없고, Value는 중복될 수 있다.

대표 구현체는 다음과 같다.

- `HashMap`
- `TreeMap`

자료구조마다 특성이 다르기 때문에 상황에 맞는 컬렉션을 선택해야 한다.

---

## 29. 오늘 정리

오늘은 Java에서 멀티스레드 프로그램을 작성할 때 필요한 기본 개념과 동기화 방법을 학습했다.

가장 중요한 흐름은 다음과 같다.

```text
Process와 Thread 이해
→ Thread 생성과 실행
→ 스레드 상태와 제어 이해
→ JVM 메모리 모델 이해
→ volatile로 가시성 문제 이해
→ Race Condition 이해
→ synchronized로 임계 영역 보호
→ AtomicInteger와 ReentrantLock 이해
→ wait/notify와 LockSupport로 스레드 흐름 제어
→ 연결 리스트 같은 공유 자료구조 동기화
→ 제네릭과 컬렉션 프레임워크 기초 학습
```

오늘 학습에서 가장 중요한 점은 멀티스레드 환경에서는 코드가 위에서 아래로 단순히 실행된다고 생각하면 안 된다는 것이다.

여러 스레드가 동시에 실행되면 실행 순서가 달라질 수 있고, 공유 데이터에 접근할 때는 가시성, 원자성, 동기화 문제를 반드시 고려해야 한다.
