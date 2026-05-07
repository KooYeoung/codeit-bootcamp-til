# [WIL] 1주차

## 기간

- 2026-04-30 ~ 2026-05-06

## 이번 주에 한 일

- 코드잇 부트캠프를 시작하고 Git/GitHub 기본 흐름을 학습했다.
- Git의 기본 작업 흐름인 수정, staging, commit, branch, merge를 정리했다.
- GitHub 원격 저장소, Pull Request, Fork, 코드 리뷰, 브랜치 전략을 학습했다.
- Java 학습을 시작하기 전 컴퓨터 구조, 파일 시스템, JDK/JRE/JVM 개념을 복습했다.
- Java 기초 문법, 객체지향, 스레드, 컬렉션, 파일 I/O, NIO, Socket 개념을 학습했다.
- 매일 TIL을 작성하고, 주요 주제는 learning 문서로 따로 정리했다.

## What I Learned

### Git / GitHub

- Git은 파일의 변경 이력을 관리하는 분산 버전 관리 시스템이라고 이해했다.
- `git add`, `git commit`, `git status`, `git diff`를 중심으로 기본 작업 흐름을 익혔다.
- branch는 독립적인 작업 공간이고, merge는 다른 브랜치의 작업 내용을 합치는 과정이라고 정리했다.
- `reset`은 커밋 이력을 되돌리는 방식이고, `revert`는 기존 이력을 유지한 채 취소 커밋을 만드는 방식이라고 이해했다.
- GitHub는 원격 저장소를 기반으로 협업할 수 있게 해주는 서비스이며, Pull Request는 코드 리뷰와 병합 요청의 중심 흐름이라고 이해했다.
- Feature Branch Workflow, Git Flow, Fork 기반 협업 흐름을 학습하면서 협업에서는 main 브랜치를 안정적으로 유지하는 것이 중요하다고 느꼈다.

### Java

- Java는 소스 코드를 바이트코드로 컴파일한 뒤 JVM에서 실행하는 언어라고 이해했다.
- JDK, JRE, JVM의 역할을 구분하고 Java 프로그램이 실행되는 흐름을 정리했다.
- 변수, 자료형, 형변환, 연산자, 조건문, 반복문, 배열 같은 기초 문법을 복습했다.
- 객체지향에서는 클래스, 객체, 인스턴스, 생성자, 접근 제어, 상속, 다형성, 추상 클래스, 인터페이스 개념을 학습했다.
- 스레드는 프로세스 안에서 실행되는 흐름이며, 여러 스레드가 공유 데이터에 접근할 때 동기화가 필요하다고 이해했다.
- `synchronized`, `AtomicInteger`, `ReentrantLock`, `wait()`, `notify()`의 차이를 정리했다.
- 제네릭과 컬렉션 프레임워크를 통해 타입 안정성과 자료구조 선택의 중요성을 배웠다.
- 파일 I/O, Stream, Reader/Writer, 직렬화, NIO, Buffer, Channel, Selector, Socket, TCP 서버 구조를 학습했다.

## 어려웠던 점

- Git 명령어는 각각 이해했지만, 실제 협업 흐름 안에서 언제 어떤 명령어를 써야 하는지 연결하는 데 시간이 필요했다.
- `reset`과 `revert`, `fetch`와 `pull`, Fork 기반 PR과 브랜치 기반 PR처럼 비슷해 보이는 개념을 구분하는 것이 어려웠다.
- Java는 문법뿐 아니라 JVM, 메모리 구조, 객체지향 개념까지 함께 이해해야 해서 학습 범위가 넓게 느껴졌다.
- 스레드 동기화에서는 `volatile`, `synchronized`, `AtomicInteger`, `ReentrantLock`의 사용 기준이 헷갈렸다.
- NIO의 Buffer, Channel, Selector 구조는 기존 Stream 방식과 달라 처음에는 흐름을 잡기 어려웠다.

## 해결 방법

- Git은 명령어별로 외우기보다 작업 흐름 안에서 역할을 연결해 정리했다.
- PR, branch, commit은 각각 다른 단위라는 점을 기준으로 협업 흐름을 다시 정리했다.
- Java는 문법을 단독으로 보기보다 JVM 실행 구조, 메모리, 객체 생성 흐름과 연결해서 이해하려고 했다.
- 스레드 동기화는 공유 데이터를 안전하게 다루기 위한 방법이라는 기준으로 개념을 나누어 정리했다.
- I/O와 NIO는 Stream 중심 방식과 Buffer/Channel 중심 방식의 차이를 기준으로 구분했다.

## START

- 다음 주부터 Java 기초 문법과 객체지향 예제를 직접 더 많이 작성해보기
- Git 원격 저장소와 PR 기반 협업 흐름을 실제 저장소에서 반복 연습하기
- Spring 학습 전에 Java 핵심 개념을 다시 점검하기

## STOP

- 강의 내용을 그대로 길게 옮겨 적는 방식 줄이기
- TIL에 learning 수준의 상세 설명을 너무 많이 넣는 것 줄이기
- 개념을 이해하지 못한 상태에서 용어만 외우는 방식 줄이기

## CONTINUE

- 매일 TIL을 작성하며 하루 학습 흐름을 정리하기
- 자세히 다시 볼 주제는 learning 문서로 분리해서 정리하기
- 헷갈린 개념은 어려웠던 점과 해결 방법에 남기기
- Git과 Java 개념을 예제와 함께 복습하기

## 다음 주 계획

- Git 원격 저장소, branch, Pull Request 흐름 복습
- Java 기초 문법 예제 직접 작성
- 객체지향 개념을 작은 코드 예제로 다시 정리
- 스레드와 컬렉션 주요 개념 복습
- Spring 학습 전 Java 기본기 재점검
- 간단한 Spring Boot CRUD 흐름 예습
