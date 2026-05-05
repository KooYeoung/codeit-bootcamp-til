# Java 기초 학습 Part 1

## 학습 날짜

2026-05-03

## 학습 주제

- Java 학습 전 기본 배경
- 컴퓨터 구조 기초
- 프로그래밍 언어와 절차적 사고
- JDK, JRE, JVM
- 빌드타임과 런타임
- Java 자료형
- 부동소수점 오차
- 문자와 문자열
- 변수와 상수
- 콘솔 입출력
- 조건문, 반복문, 배열 기초

---

## 1. Java 학습 전 기본 배경

Java를 제대로 이해하려면 문법뿐만 아니라 컴퓨터가 프로그램을 어떻게 실행하는지에 대한 기본 개념도 함께 알아야 한다.

Java 학습 전에 알아두면 좋은 개념은 다음과 같다.

- CPU
- Register
- Cache
- RAM
- SSD / HDD
- File System
- OS
- ASCII
- Unicode
- Encoding

프로그램 실행은 결국 CPU가 메모리의 값을 가져와 연산하고, 그 결과를 다시 메모리에 저장하는 과정이다.

```text
메모리에서 값 읽기
→ CPU 레지스터로 이동
→ CPU 연산
→ 결과를 다시 메모리에 저장
```

Java 백엔드 개발자로 성장하려면 Java 문법뿐만 아니라 CS 기본기도 함께 쌓아야 한다.

---

## 2. 파일, 경로, 콘솔 기본 개념

개발환경을 다루려면 파일과 경로 개념을 알아야 한다.

### 파일과 확장자

파일은 이름과 확장자로 구분된다.

```text
Main.java
README.md
test.txt
```

확장자는 파일의 성격을 나타낸다.

- `.java`: Java 소스 파일
- `.class`: Java 컴파일 결과 파일
- `.md`: Markdown 문서 파일
- `.txt`: 텍스트 파일

### 폴더, 디렉토리, 경로

폴더와 디렉토리는 거의 같은 의미로 사용된다.

경로는 파일이나 폴더가 위치한 주소이다.

```text
C:\Windows\System\notepad.exe
```

절대경로는 전체 위치를 나타내고, 상대경로는 현재 위치를 기준으로 파일이나 폴더를 찾는 방식이다.

```text
.\test.txt
..\test.txt
```

### 기본 콘솔 명령어

```bash
dir
cd 폴더명
cd ..
```

- `dir`: 현재 위치의 파일과 폴더 목록 확인
- `cd`: 디렉토리 이동
- `cd ..`: 상위 디렉토리로 이동

---

## 3. 컴퓨터 구조 기초

컴퓨터의 메모리 계층은 대략 다음과 같이 볼 수 있다.

```text
Register
→ L1, L2, L3 Cache
→ RAM
→ SSD / HDD
```

CPU에 가까울수록 속도는 빠르지만 용량은 작다.  
저장장치에 가까울수록 용량은 크지만 속도는 느리다.

프로그램이 실행된다는 것은 CPU가 명령을 수행한다는 의미이고, CPU 연산에 필요한 데이터는 메모리에서 가져와야 한다.

---

## 4. 2진수와 16진수

컴퓨터는 내부적으로 0과 1을 기반으로 동작한다.

기계어는 전기 신호의 조합이고, 이를 2진수로 표현할 수 있다.

2진수는 길어지기 쉬우므로 보통 16진수로 변환해서 표현한다.

```text
2진수 4비트 = 16진수 한 자리
```

예시:

```text
0000 → 0
0001 → 1
1010 → A
1111 → F
```

16진수는 보통 `0x`를 붙여 표현한다.

```text
0xFF
0xAC00
```

---

## 5. 프로그래밍 언어와 실행 방식

CPU가 직접 이해할 수 있는 언어는 기계어이다.

사람이 기계어를 직접 작성하기 어렵기 때문에, 사람이 이해하기 쉬운 고급어를 사용한다.

예시:

- C
- C++
- Java
- Python
- JavaScript

고급어로 작성한 소스코드는 컴파일러나 인터프리터를 통해 실행 가능한 형태로 처리된다.

### 컴파일러

컴파일러는 소스코드를 실행 가능한 형태로 번역하는 프로그램이다.

특징은 다음과 같다.

- 전체 소스코드를 변환한 후 실행
- 실행 성능 최적화에 유리
- C, C++ 등이 대표적

### 인터프리터

인터프리터는 소스코드를 한 줄씩 해석하면서 실행하는 방식이다.

특징은 다음과 같다.

- 한 줄 단위로 실행
- 컴파일 방식보다 느릴 수 있음
- JavaScript, Python 등이 대표적

### Java의 실행 방식

Java는 `.java` 파일을 `.class` 바이트코드로 컴파일한 뒤 JVM에서 실행한다.

```text
.java
→ compile
→ .class
→ JVM에서 실행
```

Java는 컴파일과 JVM 실행 과정을 함께 이해해야 한다.

---

## 6. 프로그래밍은 절차적 글쓰기

프로그래밍은 절차적 순서를 기술하는 글쓰기라고 볼 수 있다.

중요한 것은 코드를 바로 입력하는 것이 아니라, 먼저 문제를 정의하고 해결 절차를 생각하는 것이다.

예를 들어 잼을 식빵에 바르는 과정도 절차로 나누면 다음과 같다.

```text
1. 잼, 식빵, 나이프를 준비한다.
2. 잼 병의 뚜껑을 연다.
3. 나이프로 잼을 뜬다.
4. 식빵을 손에 올린다.
5. 잼을 식빵에 고르게 바른다.
```

프로그래밍도 이처럼 해결 과정을 순서대로 나누는 훈련이 필요하다.

---

## 7. JDK, JRE, JVM

### JDK

JDK는 Java Development Kit이다.

Java 프로그램을 개발하기 위한 도구 모음이다.

JDK에는 다음이 포함된다.

- JRE
- Java 컴파일러
- 개발 도구

### JRE

JRE는 Java Runtime Environment이다.

Java 프로그램을 실행하기 위한 환경이다.

JRE에는 다음이 포함된다.

- JVM
- Java 클래스 라이브러리
- 실행 관련 도구

### JVM

JVM은 Java Virtual Machine이다.

Java 바이트코드를 실행하는 가상 머신이다.

정리하면 다음과 같다.

```text
JDK
→ 개발환경

JRE
→ 실행환경

JVM
→ Java bytecode 실행
```

---

## 8. Java 개발환경과 Hello World

Java 개발을 위해서는 JDK와 IDE가 필요하다.

이번 학습에서는 IntelliJ IDEA를 기준으로 Java 개발환경을 구성했다.

기본 흐름은 다음과 같다.

```text
JDK 설치
→ IntelliJ 설치
→ 새 Java 프로젝트 생성
→ Main 클래스 작성
→ 실행
```

가장 기본적인 Java 예제는 다음과 같다.

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

---

## 9. 빌드타임과 런타임

Java 프로그램은 크게 빌드타임과 런타임을 거친다.

### 빌드타임

빌드타임에는 `.java` 파일이 `.class` 파일로 변환된다.

```text
.java
→ compile
→ .class
```

`.class` 파일은 Java bytecode이다.

### 런타임

런타임에는 JVM이 `.class` 파일을 로딩하고 실행한다.

JVM의 Class Loader가 필요한 클래스를 메모리에 올리고, Execution Engine이 바이트코드를 실행한다.

정리하면 다음과 같다.

```text
빌드타임
→ Java 소스코드를 bytecode로 변환

런타임
→ JVM이 bytecode를 실행
```

---

## 10. Java의 특징

Java의 주요 특징은 다음과 같다.

- JVM 기반에서 동작한다.
- 객체지향 언어이다.
- OS나 플랫폼 의존성이 낮다.
- C/C++와 문법적으로 유사하다.
- 메모리 관리 부담을 구조적으로 줄였다.
- 컴파일러와 인터프리터 특징을 모두 가진다.

Java는 JVM 위에서 실행되기 때문에 운영체제에 직접 의존하지 않고, 같은 바이트코드를 여러 환경에서 실행할 수 있다.

---

## 11. JVM 구성요소

JVM은 Java bytecode를 실행하기 위한 가상 머신이다.

주요 구성요소는 다음과 같다.

```text
Class Loader
Runtime Data Area
Execution Engine
Garbage Collector
JNI
Native Method Library
```

### Class Loader

`.class` 파일을 로딩하고 JVM 메모리에 적재한다.

### Runtime Data Area

JVM이 프로그램을 실행하면서 사용하는 메모리 영역이다.

대표적으로 다음 영역이 있다.

```text
Method Area
Heap Area
Stack Area
PC Register
Native Method Stack
```

### Execution Engine

바이트코드를 실행하는 역할을 한다.

### Garbage Collector

더 이상 사용하지 않는 객체를 정리하는 역할을 한다.

---

## 12. JIT Compiler

JIT는 Just In Time Compiler의 줄임말이다.

JIT Compiler는 Java bytecode를 실제 기계어로 변환해 실행 성능을 높인다.

JVM은 반복적으로 실행되는 코드를 발견하면 JIT Compiler를 사용해 최적화할 수 있다.

반복문처럼 자주 실행되는 코드에서 성능 개선 효과가 나타날 수 있다.

---

## 13. Java 자료형

Java의 자료형은 크게 기본형과 참조형으로 나눌 수 있다.

### 기본형

기본형은 값을 직접 저장하는 자료형이다.

```text
정수형: byte, short, char, int, long
실수형: float, double
논리형: boolean
```

### 참조형

참조형은 객체의 위치를 참조하는 자료형이다.

```text
String
Array
Class
Interface
List
Queue
Stack
```

기본형은 실제 값을 다루고, 참조형은 객체의 위치를 참조한다고 이해할 수 있다.

---

## 14. 정수형

정수형은 정수를 저장하기 위한 자료형이다.

대표적인 정수형은 다음과 같다.

```text
byte
short
char
int
long
```

각 자료형은 크기가 다르고, 표현할 수 있는 값의 범위도 다르다.

일반적인 정수 연산에서는 `int`를 많이 사용하고, 더 큰 범위가 필요할 때 `long`을 사용한다.

---

## 15. 실수형과 부동소수점 오차

실수형은 소수점을 포함한 값을 표현하기 위한 자료형이다.

Java의 실수형은 다음과 같다.

```text
float
double
```

`float`과 `double`은 실수를 정확한 10진수 값으로 저장하는 것이 아니라, 2진수 기반의 근사값으로 저장한다.

따라서 모든 10진수 소수를 정확히 표현할 수 없고, 실수 연산에서는 오차가 발생할 수 있다.

예를 들어 `0.1`은 사람이 보기에는 정확한 소수지만, 컴퓨터 내부의 2진수 표현에서는 딱 떨어지지 않을 수 있다.

### float과 double의 차이

```text
float
→ 32bit
→ 대략 7자리 정도의 유효 자릿수

double
→ 64bit
→ 대략 15자리 정도의 유효 자릿수
```

여기서 중요한 것은 단순히 소수점 아래 몇 자리라는 뜻이 아니라, 전체 숫자 기준으로 신뢰할 수 있는 유효 자릿수라는 점이다.

정리하면 다음과 같다.

```text
부동소수점 오차의 원인
→ 10진수 소수를 2진수로 정확히 표현하지 못하는 경우가 있기 때문

float보다 double이 더 정밀한 이유
→ 더 많은 비트를 사용해 더 많은 유효 자릿수를 표현할 수 있기 때문
```

정확한 비교가 필요한 경우 단순히 `==`로 비교하는 방식은 주의해야 한다.

---

## 16. 문자와 문자열

### char

`char`는 한 문자를 저장하기 위한 자료형이다.

```java
char ch = 'A';
```

`char`는 정수형에 속하기 때문에 문자와 숫자 간 연산이 가능하다.

예시:

```java
System.out.println((char)(65 + 1));
System.out.println((char)('A' + 2));
```

### 문자열

문자 여러 개가 연속된 형태를 문자열이라고 한다.

Java에서는 문자열을 `String` 클래스로 다룬다.

```java
String strHello = "Hello";
System.out.println(strHello + " World");
```

---

## 17. Unicode와 인코딩

컴퓨터는 문자를 숫자로 저장한다.

문자를 어떤 숫자로 표현할지 정한 규칙이 문자 코드이고, 그 값을 바이트로 어떻게 저장할지 정한 규칙이 인코딩이다.

Java의 문자는 Unicode 기반이다.

### UTF-8

UTF-8은 문자를 1~4바이트로 가변 인코딩한다.

한글 한 글자는 UTF-8에서 보통 3바이트로 표현된다.

예를 들어 `"가"`는 UTF-8에서 다음과 같은 바이트 값으로 표현될 수 있다.

```text
EA B0 80
```

### UTF-16

Java 문자열 내부 표현은 UTF-16 계열 개념과 연결된다.

문자와 문자열을 다룰 때는 인코딩에 따라 실제 바이트 수가 달라질 수 있다는 점을 알아야 한다.

---

## 18. 변수

변수는 값을 저장하기 위한 메모리 공간이다.

값이 정해지지 않았거나, 앞으로 변경될 가능성이 있는 값을 다룰 때 변수를 사용한다.

예시:

```java
int age = 20;
double score = 95.5;
String name = "Kim";
```

변수는 프로그램에서 메모리를 사용하는 가장 일반적인 방법이다.

---

## 19. 상수

상수는 값이 변하지 않는 수이다.

Java에서는 `final` 키워드를 사용해 심볼릭 상수를 만들 수 있다.

```java
final int MAX_COUNT = 10;
```

상수는 값이 바뀌면 안 되는 경우에 사용한다.

---

## 20. 식별자 작성 규칙

변수, 클래스, 메서드 이름을 식별자라고 한다.

식별자 작성 규칙은 다음과 같다.

- 영문 대소문자, 숫자, `_` 사용 가능
- 첫 글자에 숫자 사용 불가
- 공백 사용 불가
- 예약어 사용 불가
- 의미 있는 이름 사용 권장
- 너무 긴 이름은 피하고 카멜 표기법 사용 권장

예시:

```java
int data;
int userAge;
int totalScore;
```

좋지 않은 예시는 다음과 같다.

```java
int aaa;
int bbb;
int ccc;
```

의미 없는 이름은 시간이 지난 뒤 코드를 이해하기 어렵게 만든다.

---

## 21. 변수의 종류

Java에서 변수는 위치와 역할에 따라 나눌 수 있다.

```text
지역변수
매개변수
인스턴스 변수
클래스 변수
```

### 지역변수

메서드나 블록 안에서 선언되는 변수이다.

일반적으로 Stack 영역과 관련된다.

### 매개변수

메서드 호출 시 전달받는 값이다.

```java
public void printName(String name) {
    System.out.println(name);
}
```

### 인스턴스 변수

객체가 생성될 때 객체마다 따로 가지는 변수이다.

### 클래스 변수

`static` 키워드가 붙은 변수로, 클래스 단위로 공유된다.

---

## 22. 주석

주석은 코드에 설명을 남기기 위한 문법이다.

Java의 주석은 다음과 같다.

### 한 줄 주석

```java
// 한 줄 주석
```

### 여러 줄 주석

```java
/*
여러 줄 주석
*/
```

주석은 유지보수성을 높이는 데 도움이 된다.

다만 코드와 주석이 서로 다르면 혼란을 줄 수 있으므로, 코드를 수정하면 주석도 함께 수정해야 한다.

---

## 23. 콘솔과 CLI

콘솔은 명령어를 입력하고 결과를 확인하는 환경이다.

CLI는 Command Line Interface의 줄임말이다.

키보드로 입력한 값은 메모리의 I/O Buffer에 저장되고, 프로그램은 이를 읽어 처리한다.

---

## 24. System.in.read()

`System.in.read()`는 콘솔에서 입력된 값을 바이트 단위로 읽는다.

예시:

```java
public class Main {
    public static void main(String[] args) throws Exception {
        int keyCode = System.in.read();
        System.out.println(keyCode);
    }
}
```

영문자 `A`를 입력하면 ASCII 코드 값인 `65`가 출력될 수 있다.

한글은 UTF-8 인코딩 결과로 여러 바이트가 출력될 수 있다.

---

## 25. Scanner

`Scanner` 클래스는 콘솔 입력을 쉽게 처리하기 위해 사용한다.

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner s = new Scanner(System.in);

        int a = s.nextInt();
        double b = s.nextDouble();

        System.out.println("a: " + a);
        System.out.println("b: " + b);
    }
}
```

자주 사용하는 메서드는 다음과 같다.

```text
nextInt()
nextDouble()
next()
nextLine()
```

### 주의할 점

`nextInt()`나 `nextDouble()`로 숫자를 입력받은 뒤 `nextLine()`을 사용하면 입력 버퍼에 남은 개행 문자 때문에 예상과 다르게 동작할 수 있다.

따라서 숫자 입력 후 문자열 한 줄을 입력받을 때는 버퍼 처리에 주의해야 한다.

---

## 26. 조건문

조건문은 경우의 수에 따라 프로그램 흐름을 다르게 처리할 때 사용한다.

대표적인 조건문은 다음과 같다.

```java
if (조건식) {
    // 조건이 참일 때 실행
} else {
    // 조건이 거짓일 때 실행
}
```

조건이 여러 개인 경우 `else if`를 사용할 수 있다.

```java
if (score >= 90) {
    System.out.println("A");
} else if (score >= 80) {
    System.out.println("B");
} else {
    System.out.println("C");
}
```

---

## 27. 반복문

반복문은 같은 작업을 여러 번 수행할 때 사용한다.

대표적인 반복문은 다음과 같다.

```text
while
for
foreach
```

### while

조건이 참인 동안 반복한다.

```java
int i = 1;

while (i <= 10) {
    System.out.println(i);
    i++;
}
```

### for

반복 횟수가 명확할 때 많이 사용한다.

```java
for (int i = 1; i <= 10; i++) {
    System.out.println(i);
}
```

### foreach

배열이나 컬렉션의 요소를 순서대로 꺼내 사용할 때 편리하다.

```java
int[] list = {10, 20, 30, 40, 50};

for (int data : list) {
    System.out.println(data);
}
```

---

## 28. 배열

배열은 같은 자료형의 값을 여러 개 묶어서 관리하는 자료구조이다.

Java 배열은 객체로 구현되어 있으며, 배열 이름은 배열 객체에 대한 참조이다.

배열 인덱스는 0부터 시작한다.

```java
int[] array = {10, 20, 30, 40, 50};
```

배열 길이는 `length`로 확인할 수 있다.

```java
System.out.println(array.length);
```

배열 요소에 접근할 때는 인덱스를 사용한다.

```java
System.out.println(array[0]);
System.out.println(array[1]);
```

주의할 점은 배열의 범위를 벗어난 인덱스에 접근하면 오류가 발생한다는 것이다.

```java
int[] array = {10, 20, 30};

System.out.println(array[3]); // 오류
```

배열의 마지막 인덱스는 다음과 같다.

```text
배열 길이 - 1
```

---

## 29. 오늘 정리

오늘은 Java 문법을 배우기 전에 필요한 기본 배경부터 Java 기초 문법까지 넓게 학습했다.

가장 중요한 흐름은 다음과 같다.

```text
컴퓨터 구조 이해
→ 프로그래밍 언어 개념 이해
→ Java 개발환경 구성
→ JDK/JRE/JVM 이해
→ Java 자료형 학습
→ 변수와 상수 학습
→ 콘솔 입출력 학습
→ 조건문, 반복문, 배열 학습
```

Java는 단순히 문법만 외우는 것이 아니라 JVM 위에서 동작하는 언어라는 점을 이해해야 한다.

오늘 학습에서 가장 중요한 태도는 코드를 바로 작성하기 전에 문제를 절차적으로 생각하는 것이다.

프로그래밍은 단순한 타이핑이 아니라 문제를 정의하고, 해결 순서를 논리적으로 작성하는 과정이다.