# Java 파일 I/O와 소켓 프로그래밍 기초

## 학습 날짜

2026-05-06

## 학습 주제

- 파일 시스템 기초
- File 클래스
- Buffered I/O와 Non-buffered I/O
- Blocking I/O와 Non-blocking I/O
- I/O Stream
- InputStream / OutputStream
- Reader / Writer
- 보조 스트림
- 객체 직렬화
- NIO
- Buffer / Channel / Selector
- FileChannel
- NIO.2와 CompletionHandler
- 네트워크 기초
- TCP/IP
- Socket
- TCP 소켓 프로그래밍
- Echo 서버 / 클라이언트
- 멀티스레드 서버
- 채팅 서버 / 클라이언트
- I/O 멀티플렉싱 서버

---

## 1. 파일 시스템 기초

파일은 2차 메모리에 저장되는 정보 단위이다.

파일은 논리적으로 1차원 선형 구조로 저장되며, 데이터를 순서대로 읽고 쓰는 흐름을 Stream이라고 볼 수 있다.

HDD는 논리적으로 Track과 Sector 구조를 가진다.

- Track: 디스크의 원형 저장 구간
- Sector: Track을 나눈 저장 단위

파일 시스템은 파일명, 확장자, 경로, 저장 위치 등을 관리한다.

파일은 생성 시 크기가 0이며, 데이터를 쓰면 파일 크기가 증가한다.

---

## 2. PATH

PATH는 파일이나 디렉토리가 위치한 경로를 의미한다.

개발에서는 절대경로와 상대경로를 구분해서 사용할 수 있어야 한다.

- 절대경로: 루트부터 전체 위치를 나타내는 경로
- 상대경로: 현재 위치를 기준으로 나타내는 경로

예시:

    C:/Tmp/test.txt
    ./test.txt
    ../test.txt

파일 시스템은 OS에 의존적인 부분이 많기 때문에 Windows, macOS, Linux에서 경로 표현 방식이 다를 수 있다.

---

## 3. Buffered I/O와 Non-buffered I/O

### Buffered I/O

Buffered I/O는 데이터를 바로 장치에 쓰지 않고, 메모리 버퍼에 모았다가 한 번에 처리하는 방식이다.

작은 데이터를 여러 번 쓰는 경우, 매번 OS에 쓰기 요청을 보내면 성능이 떨어질 수 있다.

버퍼를 사용하면 장치 접근 횟수를 줄일 수 있어 전체 I/O 성능을 높일 수 있다.

### Non-buffered I/O

Non-buffered I/O는 버퍼를 거치지 않고 바로 입출력 요청을 처리하는 방식이다.

효율은 떨어질 수 있지만, 빠른 쓰기 요청 자체가 더 중요한 경우에는 사용할 수 있다.

중요한 점은 Buffered I/O와 Blocking I/O는 서로 다른 개념이라는 것이다.

    Buffered I/O
    → 데이터를 모아서 처리하는 성능 최적화 개념

    Blocking I/O
    → 작업이 끝날 때까지 스레드가 기다리는 실행 방식

---

## 4. Blocking I/O와 Non-blocking I/O

### Blocking I/O

Blocking I/O는 입출력 작업이 끝날 때까지 현재 스레드가 대기하는 방식이다.

파일 읽기, 파일 쓰기, 네트워크 수신 같은 작업이 완료될 때까지 다음 코드로 넘어가지 않는다.

구조는 단순하지만, 입출력 작업이 느리면 스레드가 오래 멈춰 있을 수 있다.

### Non-blocking I/O

Non-blocking I/O는 입출력 작업이 완료되지 않아도 스레드가 계속 다른 작업을 할 수 있는 방식이다.

입출력 자체는 OS가 담당하고, 애플리케이션은 준비된 이벤트를 확인하거나 콜백을 통해 결과를 처리할 수 있다.

Non-blocking I/O는 스레드 수를 줄이고 Context Switching 비용을 줄이는 데 도움이 될 수 있다.

---

## 5. 장치 파일

장치 파일은 특정 장치를 파일처럼 다룰 수 있게 추상화한 인터페이스이다.

키보드, 모니터 같은 장치도 응용 프로그램 관점에서는 읽기와 쓰기 대상으로 볼 수 있다.

예를 들어 콘솔 출력도 결국 특정 장치 파일에 데이터를 쓰는 것으로 볼 수 있다.

`System.out`은 콘솔에 연결된 출력 스트림이며, 내부적으로 `PrintStream`으로 다룰 수 있다.

---

## 6. File 클래스

`File` 클래스는 `java.io` 패키지에서 제공하는 파일과 디렉토리 관리 클래스이다.

파일 내용 입출력 자체가 아니라 경로, 메타데이터, 파일 생성, 삭제, 디렉토리 목록 조회 등에 사용된다.

실제 파일 내용 입출력은 Stream, Reader, Writer, Channel이 담당한다.

주요 메서드는 다음과 같다.

- `exists()`
- `createNewFile()`
- `delete()`
- `getName()`
- `getPath()`
- `getAbsolutePath()`
- `length()`
- `isFile()`
- `isDirectory()`
- `canRead()`
- `canWrite()`
- `mkdir()`
- `mkdirs()`
- `list()`
- `listFiles()`
- `renameTo()`

파일 시스템은 OS에 종속적이며, 파일 접근은 프로세스 권한에 따라 제한될 수 있다.

디렉토리 안에 파일이 존재하면 디렉토리 삭제가 실패할 수 있다.

따라서 디렉토리를 삭제하려면 내부 파일을 먼저 삭제한 뒤 디렉토리를 삭제해야 한다.

---

## 7. 파일 접근과 경쟁 조건

여러 스레드가 같은 파일에 동시에 쓰기 작업을 하면 데이터가 섞이는 문제가 발생할 수 있다.

파일에 대한 접근 권한은 프로세스 수준에서 통제되지만, 같은 프로세스 안의 여러 스레드는 같은 파일에 동시에 접근할 수 있다.

따라서 한 프로세스 안에서 여러 스레드가 같은 파일에 쓰기 작업을 할 경우 동기화가 필요하다.

파일 쓰기 메서드는 내부적으로 여러 작업으로 구성될 수 있으므로, 공유 파일을 다룰 때는 임계 영역을 고려해야 한다.

---

## 8. I/O Stream

Stream은 데이터가 순서대로 흐르는 1차원 선형 구조이다.

파일, 네트워크, 콘솔 입출력 등은 Stream 개념으로 이해할 수 있다.

Stream은 방향에 따라 입력과 출력으로 나눌 수 있다.

    InputStream
    → 외부에서 프로그램으로 데이터를 읽어옴

    OutputStream
    → 프로그램에서 외부로 데이터를 내보냄

데이터 단위에 따라 바이트 스트림과 문자 스트림으로 구분할 수 있다.

    byte 기반
    → InputStream / OutputStream

    char 기반
    → Reader / Writer

---

## 9. InputStream

`InputStream`은 바이트 단위 입력을 처리하는 클래스 계열이다.

주요 메서드는 다음과 같다.

- `read()`
- `read(byte[])`
- `read(byte[], offset, len)`
- `close()`

`read()`는 읽어온 바이트 값을 `int`로 반환한다.

더 이상 읽을 데이터가 없으면 `-1`을 반환한다.

파일에서 바이트 데이터를 읽을 때는 `FileInputStream`을 사용할 수 있다.

---

## 10. OutputStream

`OutputStream`은 바이트 단위 출력을 처리하는 클래스 계열이다.

주요 메서드는 다음과 같다.

- `write(int)`
- `write(byte[])`
- `write(byte[], offset, len)`
- `flush()`
- `close()`

`write()`는 데이터를 출력 대상으로 쓴다.

`flush()`는 버퍼에 남은 데이터를 강제로 출력 대상으로 내보낸다.

`close()`는 스트림을 닫고 자원을 해제한다.

파일에 바이트 데이터를 쓸 때는 `FileOutputStream`을 사용할 수 있다.

---

## 11. BufferedOutputStream

`BufferedOutputStream`은 출력 데이터를 버퍼에 모았다가 한 번에 쓰는 보조 스트림이다.

작은 데이터를 반복해서 쓰는 경우 성능 저하를 줄일 수 있다.

파일뿐만 아니라 네트워크 소켓에서도 버퍼링을 적용할 수 있다.

다만 버퍼 크기가 커진다고 해서 성능이 무조건 계속 좋아지는 것은 아니다.

---

## 12. Reader와 Writer

`Reader`와 `Writer`는 문자 기반 입출력을 처리하는 클래스 계열이다.

텍스트 데이터를 다룰 때는 인코딩을 고려해야 하므로 바이트 스트림보다 문자 스트림이 더 적합할 수 있다.

### Reader

`Reader`는 문자를 읽기 위한 클래스이다.

주요 메서드는 다음과 같다.

- `read()`
- `read(char[])`
- `read(char[], offset, len)`
- `close()`

대표 구현체는 다음과 같다.

- `FileReader`
- `BufferedReader`
- `InputStreamReader`

### Writer

`Writer`는 문자를 쓰기 위한 클래스이다.

주요 메서드는 다음과 같다.

- `write(int)`
- `write(char[])`
- `write(String)`
- `flush()`
- `close()`

대표 구현체는 다음과 같다.

- `FileWriter`
- `BufferedWriter`
- `PrintWriter`
- `OutputStreamWriter`

---

## 13. 보조 스트림

보조 스트림은 기본 스트림을 감싸서 기능을 추가하는 스트림이다.

기본 스트림은 실제 입출력 대상과 연결되고, 보조 스트림은 그 위에 추가 기능을 제공한다.

보조 스트림으로 추가할 수 있는 기능은 다음과 같다.

- 버퍼링을 통한 성능 향상
- 기본형 데이터 입출력
- 객체 직렬화
- 바이트 스트림과 문자 스트림 변환
- 형식화된 출력

대표적인 보조 스트림은 다음과 같다.

- `BufferedInputStream`
- `BufferedOutputStream`
- `DataInputStream`
- `DataOutputStream`
- `ObjectInputStream`
- `ObjectOutputStream`
- `InputStreamReader`
- `OutputStreamWriter`
- `PrintStream`
- `PrintWriter`

보조 스트림은 여러 개를 조합해서 사용할 수 있다.

---

## 14. DataInputStream과 DataOutputStream

`DataOutputStream`은 기본형 데이터를 타입별로 쓸 수 있게 해준다.

`DataInputStream`은 타입별로 저장된 데이터를 다시 읽을 수 있게 해준다.

지원하는 대표 메서드는 다음과 같다.

- `writeInt()`
- `writeDouble()`
- `writeBoolean()`
- `writeUTF()`
- `readInt()`
- `readDouble()`
- `readBoolean()`
- `readUTF()`

중요한 점은 데이터를 읽을 때 쓴 순서와 타입을 정확히 맞춰야 한다는 것이다.

예를 들어 다음 순서로 썼다면:

    double
    int
    String

읽을 때도 같은 순서와 타입으로 읽어야 한다.

    readDouble()
    readInt()
    readUTF()

순서나 타입이 다르면 데이터가 올바르게 복원되지 않는다.

---

## 15. 객체 직렬화

직렬화는 객체를 바이트 스트림으로 변환하는 과정이다.

역직렬화는 바이트 스트림을 다시 객체로 복원하는 과정이다.

Java에서 객체를 직렬화하려면 해당 클래스가 `Serializable` 인터페이스를 구현해야 한다.

`Serializable`은 필드와 메서드가 없는 마커 인터페이스이다.

직렬화는 다음과 같은 상황에서 사용될 수 있다.

- 객체 단위 파일 입출력
- TCP/IP 소켓 통신에서 송수신 단위로 사용
- HTTP 세션에 객체 저장
- 캐시 저장
- Redis 같은 외부 저장소 사용

---

## 16. serialVersionUID

`serialVersionUID`는 직렬화된 객체의 클래스 버전을 식별하는 값이다.

예시:

    private static final long serialVersionUID = 1L;

직렬화된 객체를 역직렬화할 때, 저장된 클래스 버전과 현재 클래스 버전이 다르면 오류가 발생할 수 있다.

필드 구성이 변경되거나 클래스 구조가 바뀔 경우 버전 관리가 필요하다.

---

## 17. transient

`transient`는 직렬화 대상에서 특정 필드를 제외할 때 사용하는 키워드이다.

예시:

    transient private String password;

직렬화에서 제외하는 것이 좋은 값은 다음과 같다.

- 비밀번호
- 인증 정보
- 보안 관련 값
- 실행 환경에 의존적인 값
- 객체 참조 주소처럼 저장 의미가 없는 값

`transient` 필드는 역직렬화 시 기본값이 된다.

---

## 18. System.out의 실체

`System.out`은 콘솔 출력에 연결된 `PrintStream` 객체이다.

즉, 콘솔 출력도 결국 스트림을 통해 처리된다.

`System.setOut()`을 사용하면 출력 대상을 콘솔이 아닌 파일로 변경할 수 있다.

이 개념을 통해 콘솔도 일종의 장치 파일처럼 다룰 수 있다는 점을 이해할 수 있다.

---

## 19. NIO

NIO는 Java New I/O를 의미한다.

기존 `java.io`는 Stream 기반의 Blocking I/O 중심이다.

반면 NIO는 Buffer와 Channel을 중심으로 동작한다.

NIO의 핵심 구성 요소는 다음과 같다.

- `Buffer`
- `Channel`
- `Selector`

### Buffer

데이터를 읽고 쓰기 위한 메모리 블록이다.

### Channel

입출력을 수행하는 통로이다.

기존 Stream과 달리 읽기와 쓰기 모두 가능한 양방향 구조를 가질 수 있다.

다만 모든 Channel이 양방향은 아니며, Channel 종류와 open 옵션에 따라 읽기와 쓰기 가능 여부가 달라진다.

### Selector

여러 채널의 입출력 준비 상태를 감지하는 객체이다.

하나의 스레드에서 여러 채널을 관리할 수 있게 해준다.

---

## 20. 기존 IO와 NIO 차이

기존 `java.io`와 NIO의 차이는 다음과 같다.

    java.io
    → Stream 기반
    → 단방향 입출력
    → Blocking 중심
    → 구조가 단순하지만 대량 연결 처리에는 비효율 가능

    java.nio
    → Buffer / Channel 기반
    → 양방향 입출력 가능
    → Non-blocking 지원
    → 한 스레드에서 여러 채널 처리 가능

NIO는 네트워크 서버처럼 많은 연결을 효율적으로 처리해야 하는 상황에서 유용하다.

---

## 21. Buffer

Buffer는 데이터를 임시로 저장하는 메모리 영역이다.

Buffer는 배열처럼 1차원 선형 구조로 이해할 수 있다.

Buffer의 주요 상태 값은 다음과 같다.

- `capacity`
- `limit`
- `position`
- `mark`

### capacity

버퍼의 전체 크기이다.

### position

현재 읽거나 쓸 위치이다.

### limit

읽거나 쓸 수 있는 한계 위치이다.

### mark

특정 위치를 기억하기 위한 값이다.

Buffer는 데이터를 쓰는 모드와 읽는 모드가 다르기 때문에, 상태 전환 메서드를 이해해야 한다.

대표 메서드는 다음과 같다.

- `flip()`
- `clear()`
- `rewind()`
- `remaining()`
- `hasRemaining()`

---

## 22. FileChannel

`FileChannel`은 NIO에서 파일 입출력을 처리하는 채널이다.

`FileChannel`은 파일 읽기, 쓰기, 위치 이동을 지원한다.

주요 메서드는 다음과 같다.

- `read(ByteBuffer)`
- `write(ByteBuffer)`
- `position()`
- `position(long)`
- `size()`
- `force()`
- `truncate(long)`
- `map()`
- `transferTo()`
- `transferFrom()`

기존 `FileInputStream`이나 `FileOutputStream`에서 채널을 얻어 사용할 수 있다.

`transferTo()`와 `transferFrom()`은 파일 복사 시 Zero-copy 방식의 성능 이점을 얻을 수 있다.

---

## 23. 단일 채널 입출력

하나의 `FileChannel`을 열어 읽기와 쓰기를 모두 처리할 수 있다.

이때 파일 위치를 나타내는 `position`을 직접 조정해야 할 수 있다.

예를 들어 파일에 데이터를 쓴 뒤 같은 채널에서 다시 읽으려면, 읽을 위치로 `position`을 되돌려야 한다.

---

## 24. NIO.2와 비동기 I/O

NIO.2는 Java 7 이상에서 제공되는 비동기 I/O 기능이다.

비동기 I/O는 입출력 요청을 보낸 뒤 완료를 기다리지 않고 다른 작업을 계속할 수 있다.

작업이 완료되면 콜백을 통해 결과를 전달받는다.

대표 클래스는 다음과 같다.

- `AsynchronousFileChannel`
- `AsynchronousSocketChannel`
- `AsynchronousServerSocketChannel`

---

## 25. CompletionHandler

`CompletionHandler`는 NIO.2에서 비동기 I/O 작업 결과를 처리하는 콜백 인터페이스이다.

입출력 작업이 완료되면 `completed()`가 호출된다.

입출력 작업이 실패하면 `failed()`가 호출된다.

구성은 다음과 같다.

- `completed(result, attachment)`: 작업 성공 시 호출
- `failed(exception, attachment)`: 작업 실패 시 호출
- `result`: 입출력 성공 결과
- `attachment`: 요청 시 함께 전달한 첨부 객체

비동기 I/O에서는 작업 완료 시점을 개발자가 직접 기다리는 것이 아니라, 완료 이벤트를 콜백으로 받아 처리한다.

---

## 26. 네트워크 기초

네트워크는 계층적 구조로 이해할 수 있다.

하위 계층이 성립해야 상위 계층이 동작할 수 있다.

예를 들어 IP 통신이 가능해야 TCP 통신이 가능하다.

개발자 관점에서 중요한 개념은 다음과 같다.

- IP 주소
- Port 번호
- Protocol
- Socket
- TCP
- UDP

---

## 27. IP 주소와 Port 번호

IP 주소는 네트워크에서 호스트를 식별하기 위한 주소이다.

호스트는 네트워크를 사용하는 컴퓨터나 장치를 의미한다.

Port 번호는 하나의 호스트 안에서 네트워크 endpoint나 service를 식별하기 위한 번호이다.

OS는 Port 번호를 소켓과 프로세스에 연결해 어느 프로그램으로 데이터를 전달할지 결정한다.

정리하면 다음과 같다.

    IP 주소
    → 통신 대상 컴퓨터 식별

    Port 번호
    → 해당 컴퓨터 안의 통신 대상 endpoint/service 식별

예시:

    127.0.0.1:8080

- `127.0.0.1`: 내 컴퓨터
- `8080`: 특정 프로그램이 사용하는 Port 번호

---

## 28. Socket

Java의 `Socket`은 클래스이며, 개념적으로는 OS 소켓 API를 사용할 수 있게 해주는 추상화로 이해할 수 있다.

소켓은 네트워크 통신을 위한 장치 파일처럼 이해할 수 있다.

파일 I/O에서 읽기와 쓰기를 사용하듯, 소켓 통신에서도 데이터를 읽고 쓴다.

파일이 경로를 기준으로 접근된다면, 소켓은 IP 주소와 Port 번호를 기준으로 통신 대상을 구분한다.

정리하면 다음과 같다.

    파일
    → 경로를 기준으로 접근

    소켓
    → IP 주소와 Port 번호를 기준으로 통신

---

## 29. TCP

TCP는 연결 기반 프로토콜이다.

TCP는 다음 특징을 가진다.

- 연결 개념이 있다.
- 데이터 순서를 관리한다.
- 데이터 손실을 확인하고 재전송할 수 있다.
- 신뢰성 있는 전송을 목표로 한다.
- 상태 전이 개념을 가진다.
- 메시지 경계가 없는 byte stream이다.

TCP는 데이터를 바이트 흐름으로 전달하므로, 애플리케이션에서 메시지 단위를 구분해야 한다.

TCP 연결 설정은 3-way handshake를 통해 이루어진다.

TCP 연결 종료는 4-way handshake를 통해 이루어진다.

다만 네트워크는 언제든 예외 상황이 발생할 수 있다.

예를 들어 클라이언트 PC 전원이 갑자기 꺼지면 서버는 즉시 연결 종료를 알지 못할 수 있다.

이런 문제를 해결하기 위해 애플리케이션 수준에서 heartbeat를 사용할 수 있다.

---

## 30. TCP 소켓 프로그래밍

TCP 서버는 `ServerSocket`을 사용해 클라이언트 연결을 기다린다.

클라이언트 연결 요청이 들어오면 `accept()`가 연결을 수락하고, 실제 통신에 사용할 `Socket` 객체를 반환한다.

TCP 클라이언트는 서버의 IP 주소와 Port 번호로 접속한다.

기본 흐름은 다음과 같다.

    ServerSocket 생성
    → accept()로 클라이언트 연결 대기
    → Socket 반환
    → InputStream / OutputStream으로 데이터 송수신
    → close()

클라이언트 흐름은 다음과 같다.

    Socket 생성
    → 서버 IP와 Port로 연결
    → InputStream / OutputStream으로 데이터 송수신
    → close()

---

## 31. Echo 서버와 클라이언트

Echo 서비스는 클라이언트가 보낸 메시지를 서버가 그대로 다시 클라이언트에게 돌려주는 구조이다.

가장 기본적인 네트워크 통신 예제로 사용할 수 있다.

동작 흐름은 다음과 같다.

    클라이언트 메시지 전송
    → 서버 메시지 수신
    → 서버가 동일한 메시지 응답
    → 클라이언트 응답 수신

Echo 서버는 TCP 소켓의 기본 입출력 구조를 이해하는 데 도움이 된다.

---

## 32. 단일 스레드 서버의 한계

단일 스레드 서버는 한 번에 하나의 클라이언트만 처리할 수 있다.

한 클라이언트와 통신하는 동안 `accept()`나 다른 클라이언트 처리가 지연될 수 있다.

따라서 여러 클라이언트를 동시에 처리하려면 클라이언트별로 작업 스레드를 분리하는 구조가 필요하다.

---

## 33. 멀티스레드 기반 서버

멀티스레드 서버는 클라이언트가 연결될 때마다 별도 스레드를 생성해 처리하는 구조이다.

장점은 여러 클라이언트를 동시에 처리할 수 있다는 것이다.

단점은 클라이언트 수가 많아지면 스레드가 많아지고 Context Switching 비용이 증가할 수 있다는 것이다.

이 구조는 이해하기 쉽지만, 고성능 서버에서는 한계가 있을 수 있다.

---

## 34. 채팅 서버와 클라이언트

채팅 서버는 Echo 서버와 달리 한 클라이언트가 보낸 메시지를 다른 클라이언트들에게 전달해야 한다.

이를 브로드캐스팅이라고 한다.

채팅 서버에서 필요한 기능은 다음과 같다.

- 클라이언트 연결 관리
- 클라이언트별 송수신 처리
- 메시지 브로드캐스팅
- 연결 종료 처리

채팅 클라이언트는 사용자 입력과 서버 메시지 수신을 동시에 처리해야 한다.

따라서 보통 송신 스레드와 수신 스레드를 분리한다.

---

## 35. I/O 멀티플렉싱 서버

I/O 멀티플렉싱은 하나의 스레드가 여러 채널의 입출력 이벤트를 감지하고 처리하는 구조이다.

NIO의 `Selector`를 사용하면 여러 클라이언트 연결을 하나의 스레드에서 관리할 수 있다.

멀티스레드 서버는 클라이언트 수가 많아지면 스레드 수가 증가한다.

반면 I/O 멀티플렉싱 서버는 적은 수의 스레드로 많은 연결을 처리할 수 있다.

정리하면 다음과 같다.

    멀티스레드 서버
    → 클라이언트마다 스레드 생성
    → 구조는 단순하지만 스레드 비용 증가

    I/O 멀티플렉싱 서버
    → 하나의 스레드가 여러 채널 이벤트 처리
    → 구조는 복잡하지만 많은 연결 처리에 유리

---

## 36. 오늘 정리

오늘은 Java에서 파일과 네트워크를 다루기 위한 기본 개념을 학습했다.

가장 중요한 흐름은 다음과 같다.

    파일 시스템 이해
    → File 클래스로 파일 관리
    → Stream으로 파일 입출력 처리
    → 보조 스트림으로 기능 확장
    → 객체 직렬화 이해
    → NIO의 Buffer / Channel / Selector 이해
    → TCP/IP와 Socket 이해
    → Echo 서버와 클라이언트 구현 흐름 이해
    → 멀티스레드 서버와 I/O 멀티플렉싱 서버 구조 이해

오늘 학습에서 가장 중요한 점은 파일과 소켓을 모두 입출력 대상으로 볼 수 있다는 것이다.

파일은 경로를 기준으로 접근하고, 소켓은 IP와 Port를 기준으로 접근하지만, 둘 다 읽기와 쓰기를 통해 데이터를 주고받는 구조로 이해할 수 있다.
