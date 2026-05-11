# Java 스레드 기초 정리

## 학습 날짜

2026-05-11

## 학습 주제

- 프로세스와 스레드
- 스레드 생성과 실행
- 스레드 제어와 생명 주기
- 메모리 가시성
- 동기화와 `synchronized`
- `Lock`, `ReentrantLock`, `LockSupport`
- 생산자-소비자 문제
- CAS와 원자적 연산
- 동시성 컬렉션
- 스레드 풀과 Executor 프레임워크

---

## 스레드를 왜 배우는가

- 프로그램은 기본적으로 위에서 아래로 한 줄씩 실행된다.
- 하지만 실제 프로그램에서는 여러 일을 동시에 처리해야 하는 경우가 많다.
- 예를 들어 음악을 들으면서 문서를 작성하거나, 서버가 여러 사용자의 요청을 동시에 처리하는 상황이 있다.
- 이때 하나의 실행 흐름만 있으면 앞 작업이 끝날 때까지 뒤 작업은 기다려야 한다.
- 스레드는 하나의 프로그램 안에서 여러 작업 흐름을 나누어 실행하기 위한 개념이라고 이해했다.

### 먼저 기억할 점

- 스레드를 사용하면 여러 작업을 동시에 처리하는 것처럼 만들 수 있다.
- 하지만 스레드가 늘어나면 실행 순서가 매번 같지 않을 수 있다.
- 여러 스레드가 같은 데이터를 함께 사용하면 동시성 문제가 생길 수 있다.
- 그래서 스레드는 생성과 실행보다 공유 자원을 안전하게 다루는 것이 더 중요하다.

---

## 프로세스와 스레드

### 프로세스

- 프로세스는 실행 중인 프로그램이다.
- 크롬, IntelliJ, 메모장처럼 실행되고 있는 프로그램 하나하나를 프로세스라고 볼 수 있다.
- 각 프로세스는 기본적으로 자기만의 메모리 공간을 가진다.
- 서로 다른 프로세스는 기본적으로 메모리를 직접 공유하지 않는다.

### 스레드

- 스레드는 프로세스 안에서 실제 작업을 수행하는 실행 흐름이다.
- 하나의 프로세스 안에는 여러 스레드가 있을 수 있다.
- 같은 프로세스 안의 스레드들은 힙 영역 같은 메모리 자원을 공유할 수 있다.
- 공유가 가능하다는 점은 장점이지만, 동시에 같은 값을 바꾸면 문제가 될 수 있다.

### 멀티태스킹과 스케줄링

- CPU 코어가 하나뿐이어도 운영체제는 여러 프로그램이 동시에 실행되는 것처럼 보이게 만들 수 있다.
- 운영체제는 매우 짧은 시간 단위로 실행할 작업을 바꿔가며 처리한다.
- 어떤 스레드를 언제 실행할지는 스케줄러가 결정한다.
- 개발자는 스레드 실행 순서를 정확히 예측하기 어렵다.

### 컨텍스트 스위칭

- 실행 중인 작업을 멈추고 다른 작업으로 바꾸는 과정을 컨텍스트 스위칭이라고 한다.
- 컨텍스트 스위칭이 너무 자주 일어나면 그 자체로 비용이 생긴다.
- 스레드를 많이 만든다고 항상 성능이 좋아지는 것은 아니라고 이해했다.

---

## 스레드 생성과 실행

### Thread 사용

- Java에서 스레드를 만들 때 `Thread` 클래스를 사용할 수 있다.
- 스레드는 이름을 가질 수 있고, 이름을 붙이면 로그를 확인할 때 어떤 스레드가 실행 중인지 보기 쉽다.

```java
Thread thread = new Thread(() -> {
    System.out.println("작업 실행");
}, "work-thread");

thread.start();
```

### Runnable 사용

- `Runnable`은 스레드가 실행할 작업을 담는 인터페이스이다.
- 스레드 자체와 실행할 작업을 분리할 수 있다는 점이 중요하다.

```java
class HelloRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("hello thread");
    }
}

Thread thread = new Thread(new HelloRunnable(), "hello-thread");
thread.start();
```

### start와 run의 차이

- `start()`를 호출해야 새로운 스레드가 만들어지고, 그 스레드 안에서 `run()`이 실행된다.
- `run()`을 직접 호출하면 새로운 스레드가 만들어지지 않고, 현재 스레드에서 일반 메서드처럼 실행된다.

```java
thread.start(); // 새 스레드에서 실행
thread.run();   // 현재 스레드에서 그냥 메서드 호출
```

### 데몬 스레드

- 데몬 스레드는 보조 작업을 수행하는 스레드이다.
- 모든 일반 스레드가 종료되면 데몬 스레드는 작업이 남아 있어도 함께 종료될 수 있다.
- 로그 정리, 자동 저장, 백그라운드 감시처럼 보조 역할에 사용할 수 있다.
- 중요한 작업을 데몬 스레드에 맡기면 중간에 종료될 수 있으므로 주의해야 한다.

---

## 스레드 기본 정보

- `Thread.currentThread()`로 현재 실행 중인 스레드를 가져올 수 있다.
- 스레드 이름, ID, 우선순위, 그룹, 상태를 확인할 수 있다.

```java
Thread current = Thread.currentThread();

System.out.println(current.threadId());
System.out.println(current.getName());
System.out.println(current.getPriority());
System.out.println(current.getState());
```

### 내가 이해한 내용

- `main` 메서드도 `main` 스레드에서 실행된다.
- 내가 직접 만든 스레드는 `start()` 전에는 `NEW` 상태이다.
- 스레드 이름을 잘 붙이면 멀티스레드 로그를 읽을 때 도움이 된다.

---

## 스레드 생명 주기

### 주요 상태

- `NEW`: 스레드 객체만 생성되고 아직 시작되지 않은 상태이다.
- `RUNNABLE`: 실행 가능하거나 실행 중인 상태이다.
- `BLOCKED`: 락을 얻지 못해 막혀 있는 상태이다.
- `WAITING`: 다른 스레드가 깨워줄 때까지 기다리는 상태이다.
- `TIMED_WAITING`: 정해진 시간 동안 기다리는 상태이다.
- `TERMINATED`: 실행이 끝난 상태이다.

### 상태 흐름

- 스레드를 생성하면 `NEW` 상태가 된다.
- `start()`를 호출하면 `RUNNABLE` 상태가 된다.
- `sleep()`을 사용하면 일정 시간 동안 `TIMED_WAITING` 상태가 될 수 있다.
- `join()`이나 `wait()` 등으로 다른 작업을 기다리면 `WAITING` 상태가 될 수 있다.
- `synchronized` 락을 기다릴 때는 `BLOCKED` 상태가 될 수 있다.
- `run()` 메서드가 끝나면 `TERMINATED` 상태가 된다.

---

## sleep과 join

### sleep

- `sleep()`은 현재 실행 중인 스레드를 일정 시간 쉬게 한다.
- 특정 스레드를 지정해서 재우는 것이 아니라, 이 코드를 실행한 스레드가 쉰다.
- `sleep()` 중 인터럽트가 발생하면 `InterruptedException`이 발생할 수 있다.

```java
Thread.sleep(1000);
```

### join

- `join()`은 다른 스레드가 끝날 때까지 기다릴 때 사용한다.
- 어떤 작업을 별도 스레드에서 실행한 뒤, 그 결과가 필요한 시점에 기다릴 수 있다.

```java
Thread thread = new Thread(() -> System.out.println("작업 완료"));
thread.start();
thread.join();

System.out.println("thread 작업이 끝난 뒤 실행");
```

### 특정 시간만 기다리기

- `join(long millis)`를 사용하면 무한정 기다리지 않고 정해진 시간만 기다릴 수 있다.
- 외부 작업이 너무 오래 걸릴 수 있는 상황에서는 시간 제한이 필요하다고 느꼈다.

---

## 인터럽트와 작업 중단

### 변수를 이용한 중단

- 작업 스레드를 멈추는 가장 단순한 방식은 공유 변수를 사용하는 것이다.
- 예를 들어 `runFlag` 값을 `false`로 바꾸면 작업 스레드가 반복문을 빠져나오게 만들 수 있다.

```java
class MyTask implements Runnable {
    volatile boolean runFlag = true;

    @Override
    public void run() {
        while (runFlag) {
            System.out.println("작업 중");
        }
        System.out.println("작업 종료");
    }
}
```

### interrupt

- `interrupt()`는 스레드에게 중단 신호를 보내는 방식이다.
- 강제로 즉시 종료하는 것이 아니라, 대상 스레드가 인터럽트 상태를 확인하고 스스로 종료 흐름을 처리해야 한다.
- `sleep()`, `wait()`, `join()`처럼 대기 중인 메서드는 인터럽트를 받으면 `InterruptedException`이 발생할 수 있다.

```java
Thread worker = new Thread(task, "worker");
worker.start();

worker.interrupt();
```

### yield

- `yield()`는 현재 스레드가 CPU 사용을 잠시 양보하겠다는 힌트를 주는 메서드이다.
- 반드시 다른 스레드가 바로 실행된다는 보장은 없다.
- 실행 순서를 제어하기 위한 확실한 방법으로 사용하기보다는, 스케줄러에게 양보 의사를 전달하는 정도로 이해했다.

---

## 메모리 가시성

### 문제가 생기는 이유

- 여러 스레드는 같은 객체를 공유하더라도, 값이 항상 즉시 서로에게 보인다고 단정할 수 없다.
- 한 스레드가 값을 바꿨는데 다른 스레드가 예전 값을 계속 볼 수 있다.
- 이런 문제를 메모리 가시성 문제라고 한다.

### volatile

- `volatile`은 한 스레드가 변경한 값을 다른 스레드가 볼 수 있도록 돕는다.
- 예를 들어 작업 중단 플래그처럼 값 하나를 읽고 쓰는 상황에서 사용할 수 있다.

```java
volatile boolean runFlag = true;
```

### volatile의 한계

- `volatile`은 가시성 문제를 해결하는 데 도움을 준다.
- 하지만 `i++` 같은 연산을 안전하게 만들어주지는 않는다.
- `i++`는 값을 읽고, 더하고, 다시 저장하는 여러 단계로 나뉘기 때문이다.
- 복합 연산에는 원자성이 필요하고, 이때는 `synchronized`, `Lock`, `AtomicInteger` 같은 도구를 고려해야 한다.

### Java Memory Model

- Java Memory Model은 여러 스레드가 메모리를 어떻게 보고, 값 변경이 어떻게 전달되는지에 대한 규칙이다.
- “멀티스레드 환경에서 값이 언제 다른 스레드에게 보이는지 정한 규칙” 정도로 이해했다.

---

## 동기화와 synchronized

### 공유 자원

- 여러 스레드가 함께 사용하는 값을 공유 자원이라고 한다.
- 인스턴스 필드, 컬렉션, 파일, DB 연결 등이 공유 자원이 될 수 있다.
- 공유 자원에 여러 스레드가 동시에 접근하면 동시성 문제가 생길 수 있다.

### 은행 출금 예제 관점

- 잔액이 1000원인 계좌에서 두 스레드가 동시에 800원을 출금한다고 생각할 수 있다.
- 두 스레드가 모두 잔액 검사를 통과하면 실제로는 잔액보다 많은 돈이 출금될 수 있다.
- 검증과 출금은 하나의 작업처럼 묶여야 한다.

### 임계 영역

- 여러 스레드가 동시에 들어가면 안 되는 중요한 코드 구간을 임계 영역이라고 한다.
- 임계 영역에는 한 번에 하나의 스레드만 들어가도록 제한해야 한다.

### synchronized 메서드

- 메서드에 `synchronized`를 붙이면 해당 메서드는 한 번에 하나의 스레드만 실행할 수 있다.

```java
class BankAccount {
    private int balance = 1000;

    public synchronized boolean withdraw(int amount) {
        if (balance < amount) {
            return false;
        }
        balance -= amount;
        return true;
    }
}
```

### synchronized 코드 블록

- 메서드 전체가 아니라 필요한 부분만 동기화할 수도 있다.
- 동기화 범위가 너무 넓으면 성능이 떨어질 수 있으므로, 실제로 보호해야 할 구간을 파악하는 것이 중요하다.

```java
synchronized (this) {
    balance -= amount;
}
```

### 주의할 점

- `synchronized`는 편리하지만 락을 기다리는 스레드는 `BLOCKED` 상태에서 계속 기다릴 수 있다.
- 특정 시간까지만 기다리거나, 중간에 대기를 취소하는 세밀한 제어는 어렵다.

---

## LockSupport와 ReentrantLock

### LockSupport

- `LockSupport`는 스레드를 대기 상태로 만들고 다시 깨울 수 있는 기본 도구이다.
- `park()`는 현재 스레드를 `WAITING` 상태로 만든다.
- `parkNanos()`는 정해진 시간 동안만 `TIMED_WAITING` 상태로 만든다.
- `unpark(thread)`는 대기 중인 스레드를 다시 실행 가능한 상태로 만든다.

```java
LockSupport.park();
LockSupport.unpark(thread);
```

### ReentrantLock이 필요한 이유

- `synchronized`는 간단하지만 무한 대기와 공정성 문제를 세밀하게 다루기 어렵다.
- `ReentrantLock`은 락 획득과 해제를 코드로 명시할 수 있다.
- 공정 모드, 대기 중단, 조건별 대기 같은 기능을 사용할 수 있다.

```java
Lock lock = new ReentrantLock();

lock.lock();
try {
    // 임계 영역
} finally {
    lock.unlock();
}
```

### tryLock

- `tryLock()`은 락을 얻을 수 있으면 얻고, 아니면 기다리지 않고 실패할 수 있다.
- 시간 제한을 두고 락 획득을 시도할 수도 있다.
- 무한정 기다리면 안 되는 상황에서 유용하다고 이해했다.

### 주의할 점

- `lock()`을 호출했다면 반드시 `finally`에서 `unlock()`을 호출해야 한다.
- 중간에 예외가 발생해도 락이 풀려야 다른 스레드가 계속 작업할 수 있다.

---

## 생산자-소비자 문제

### 기본 개념

- 생산자는 데이터를 만든다.
- 소비자는 데이터를 사용한다.
- 생산자와 소비자 사이에는 데이터를 잠시 보관하는 버퍼가 있다.
- 버퍼는 보통 크기가 제한되어 있다.

### 문제 상황

- 생산자가 너무 빠르면 버퍼가 가득 차서 더 이상 데이터를 넣을 수 없다.
- 소비자가 너무 빠르면 버퍼가 비어서 가져갈 데이터가 없다.
- 이때 생산자와 소비자는 적절히 기다리고, 반대쪽 작업이 진행되면 다시 깨어나야 한다.

### wait와 notify

- `wait()`는 현재 스레드를 대기 상태로 만든다.
- `notify()`는 대기 중인 스레드 하나를 깨운다.
- `wait()`와 `notify()`는 `synchronized` 블록 안에서 사용해야 한다.

```java
synchronized (lock) {
    while (queue.isEmpty()) {
        lock.wait();
    }
    String data = queue.poll();
    lock.notify();
}
```

### wait-notify의 한계

- 하나의 대기 집합을 사용하면 생산자가 생산자를 깨우거나, 소비자가 소비자를 깨우는 비효율이 생길 수 있다.
- 어떤 스레드가 깨어날지 정확히 제어하기 어렵다.
- 그래서 생산자용 대기 공간과 소비자용 대기 공간을 분리하는 방식이 필요하다.

### Condition

- `ReentrantLock`의 `Condition`을 사용하면 대기 공간을 나눌 수 있다.
- 생산자는 소비자 조건을 깨우고, 소비자는 생산자 조건을 깨우는 식으로 더 정확한 알림을 만들 수 있다.

```java
Lock lock = new ReentrantLock();
Condition notFull = lock.newCondition();
Condition notEmpty = lock.newCondition();
```

### BlockingQueue

- `BlockingQueue`는 생산자-소비자 문제를 해결하기 위해 제공되는 동시성 큐이다.
- 큐가 가득 차면 `put()`이 기다리고, 큐가 비어 있으면 `take()`가 기다린다.
- 직접 `wait`, `notify`, `Condition`을 구현하는 것보다 안전하고 간단하다.

```java
BlockingQueue<String> queue = new ArrayBlockingQueue<>(10);

queue.put("data");
String data = queue.take();
```

---

## 원자적 연산과 CAS

### 원자적 연산

- 원자적 연산은 더 이상 쪼갤 수 없는 하나의 작업처럼 실행되는 연산이다.
- 멀티스레드 환경에서 중간에 다른 스레드가 끼어들 수 없다.

### 원자적이지 않은 연산

- `i = 1`처럼 단순 대입은 원자적으로 볼 수 있다.
- 하지만 `i = i + 1`, `i++`는 원자적이지 않다.
- 값을 읽고, 계산하고, 저장하는 단계로 나뉘기 때문이다.

```java
i++; // 읽기, 증가, 저장으로 나뉘므로 원자적이지 않다.
```

### AtomicInteger

- `AtomicInteger`는 정수 값을 원자적으로 변경할 수 있게 해준다.
- 단순 카운터처럼 여러 스레드가 함께 값을 증가시키는 상황에서 사용할 수 있다.

```java
AtomicInteger count = new AtomicInteger(0);

count.incrementAndGet();
```

### CAS

- CAS는 Compare-And-Swap의 줄임말이다.
- 현재 값이 내가 기대한 값과 같을 때만 새 값으로 바꾸는 방식이다.
- 값이 다르면 다른 스레드가 먼저 바꾼 것으로 보고 다시 시도할 수 있다.
- 락을 사용하지 않고도 원자적 변경을 할 수 있는 기반 기술이라고 이해했다.

### CAS의 주의점

- 경쟁이 적을 때는 빠를 수 있다.
- 하지만 많은 스레드가 계속 동시에 변경을 시도하면 반복 재시도가 많아질 수 있다.
- 복잡한 여러 값을 한 번에 일관되게 바꿔야 하는 경우에는 락이 더 적절할 수 있다.

---

## 동시성 컬렉션

### 일반 컬렉션의 문제

- `ArrayList`, `HashMap` 같은 일반 컬렉션은 여러 스레드가 동시에 수정할 때 안전하지 않을 수 있다.
- `add()`처럼 단순해 보이는 메서드도 내부적으로는 여러 단계로 실행될 수 있다.
- 여러 스레드가 동시에 접근하면 데이터가 덮어써지거나, 크기 값이 꼬일 수 있다.

### synchronized 컬렉션

- `Collections.synchronizedList()` 같은 방식으로 동기화된 컬렉션을 만들 수 있다.
- 하지만 모든 상황에서 성능이 좋은 것은 아니며, 전체에 락을 거는 방식은 병목이 될 수 있다.

### 동시성 컬렉션

- Java는 멀티스레드 환경을 고려한 동시성 컬렉션을 제공한다.
- `ConcurrentHashMap`, `CopyOnWriteArrayList`, `BlockingQueue` 등이 있다.
- 동시성 컬렉션은 내부적으로 스레드 안전성을 고려해 설계되어 있다.

### 내가 이해한 내용

- 일반 컬렉션을 멀티스레드에서 공유할 때는 안전한지 먼저 확인해야 한다.
- 직접 동기화하기보다 Java가 제공하는 동시성 컬렉션을 사용하는 것이 더 안전할 수 있다.
- 다만 여러 연산을 묶어서 하나의 트랜잭션처럼 처리해야 한다면 컬렉션 하나만으로는 부족할 수 있다.

---

## 스레드 풀과 Executor 프레임워크

### 직접 스레드를 만들 때의 문제

- 스레드는 생성 비용이 크다.
- 스레드마다 호출 스택 같은 메모리 공간이 필요하다.
- 스레드 생성은 운영체제 자원도 사용한다.
- 요청마다 새 스레드를 만들면 사용자가 갑자기 늘어났을 때 CPU와 메모리가 버티기 어렵다.
- 실행 중인 스레드를 관리하고 종료하는 것도 어려워진다.

### 스레드 풀

- 스레드 풀은 스레드를 미리 만들어 두고 재사용하는 방식이다.
- 작업이 들어오면 큐에 보관하고, 준비된 스레드가 작업을 가져가 실행한다.
- 스레드 수를 제한할 수 있어 시스템 자원을 더 안정적으로 관리할 수 있다.

### ExecutorService

- `ExecutorService`는 스레드 풀을 쉽게 다루기 위한 Java의 프레임워크이다.
- 개발자는 직접 스레드를 만들기보다 작업을 제출한다.
- 실제 실행은 Executor가 관리한다.

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

executor.submit(() -> System.out.println("작업 실행"));
executor.shutdown();
```

### Runnable의 불편함과 Callable

- `Runnable`의 `run()`은 반환 값이 없다.
- 또한 체크 예외를 밖으로 던질 수 없다.
- 결과가 필요한 작업은 `Callable`을 사용할 수 있다.

```java
Callable<Integer> task = () -> 10 + 20;
Future<Integer> future = executor.submit(task);

Integer result = future.get();
```

### Future

- `Future`는 아직 완료되지 않았을 수 있는 작업의 결과를 나타낸다.
- `get()`을 호출하면 작업이 끝날 때까지 기다렸다가 결과를 가져온다.
- `cancel()`로 작업 취소를 요청할 수 있다.
- 작업 중 예외가 발생하면 `get()`을 호출할 때 예외를 확인하게 된다.

### 작업 컬렉션 처리

- 여러 작업을 한 번에 제출하고 결과를 모아야 할 수 있다.
- `invokeAll()`은 여러 작업이 모두 끝날 때까지 기다린다.
- `invokeAny()`는 여러 작업 중 하나라도 성공하면 그 결과를 반환할 수 있다.

---

## ExecutorService 종료

### graceful shutdown

- 서버를 종료할 때 진행 중인 작업을 갑자기 끊으면 문제가 생길 수 있다.
- 새 작업은 받지 않고, 이미 진행 중인 작업은 마무리한 뒤 종료하는 방식을 graceful shutdown이라고 한다.

### 종료 메서드

- `shutdown()`은 새 작업을 받지 않고 이미 제출된 작업을 마무리한 뒤 종료한다.
- `shutdownNow()`는 실행 중인 작업에 인터럽트를 보내고, 대기 중인 작업을 반환하며 즉시 종료를 시도한다.
- `isShutdown()`은 종료 요청 여부를 확인한다.
- `isTerminated()`는 모든 작업이 끝나 완전히 종료되었는지 확인한다.
- `awaitTermination()`은 정해진 시간 동안 종료를 기다린다.

### 종료 흐름 예시

```java
executor.shutdown();

if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
    executor.shutdownNow();
}
```

### 스레드 풀 전략

- 고정 풀 전략은 스레드 수를 고정해서 예측 가능한 자원 사용을 만든다.
- 캐시 풀 전략은 필요에 따라 스레드를 늘리고, 쉬는 스레드를 정리한다.
- 사용자 정의 풀 전략은 core pool size, maximum pool size, 작업 큐, 거절 정책 등을 직접 조절한다.

### 예외 정책

- 스레드 풀이 더 이상 작업을 받을 수 없을 때 거절 정책이 필요하다.
- 작업을 버릴지, 예외를 던질지, 호출한 스레드가 직접 실행할지 같은 전략을 선택할 수 있다.
- 요청이 몰릴 수 있는 서버에서는 거절 정책도 설계의 일부라고 느꼈다.

---

## 전체 흐름 정리

- 스레드는 하나의 프로세스 안에서 여러 작업 흐름을 만들기 위한 도구이다.
- 스레드를 직접 만들 수 있지만, 실행 순서와 공유 자원 문제를 조심해야 한다.
- `start()`는 새 스레드를 시작하고, `run()` 직접 호출은 일반 메서드 호출이다.
- 스레드는 `NEW`, `RUNNABLE`, `WAITING`, `TIMED_WAITING`, `BLOCKED`, `TERMINATED` 같은 상태를 가진다.
- `join()`은 다른 스레드의 종료를 기다리고, `sleep()`은 현재 스레드를 잠시 쉬게 한다.
- 작업 중단은 공유 플래그나 `interrupt()`를 통해 안전하게 요청하는 방식으로 처리해야 한다.
- `volatile`은 메모리 가시성에는 도움이 되지만 원자성을 보장하지 않는다.
- 공유 자원을 안전하게 다루려면 `synchronized`, `Lock`, 동시성 컬렉션, 원자적 클래스 등을 상황에 맞게 사용해야 한다.
- 생산자-소비자 문제는 멀티스레드에서 대기와 알림을 이해하기 좋은 대표 문제이다.
- 실제 많은 작업을 처리할 때는 직접 스레드를 계속 만들기보다 Executor 프레임워크와 스레드 풀을 사용하는 것이 좋다.
