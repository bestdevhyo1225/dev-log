# Java  NIO

## Channel, Buffer

- 기존 Java IO는 `Stream` 기반
- Java NIO 에서는 `Channel` 과 `Buffer` 를 기반으로 동작한다.
- `Channel` 과 `Buffer` 를 통해 데이터가 읽고 쓰여진다.
- `Channel` 에서 `Buffer` 로 데이터를 보내거나, `Buffer` 를 통해 `Channel` 에서 데이터를 읽는다.

## Non-blocking I/O

- 하나의 스레드는 `Buffer` 에 데이터를 읽도록 `Channel` 에 요청할 수 있다.
- `Channel` 을 통해 `Buffer` 에서 데이터를 읽는 동안 스레드는 다른 작업을 수행할 수 있다.
- `Buffer` 로 부터 데이터가 읽어지면, 스레드는 `Buffer` 에 있는 데이터를 처리할 수 있다.

## Selector

- 여러 개의 `Channel` 에서 이벤트를 모니터링할 수 있는 객체이다.
- 이벤트는 `accept, connect, read, write` 와 같은 것들이 있다.
- 그렇기 때문에 하나의 스레드에서 여러 `Channel` 에 대한 모니터링이 가능하다.

## Java NIO 핵심

- Channel
- Buffer
- Selector

## Channel vs Stream

- `Channel` 은 읽고 쓰기가 가능하다.
- `Stream` 은 단방향 통신만 가능하다. (읽거나 쓰거나)
- `Channel` 은 비동기적으로 읽거나 쓸 수 있다.
- `Channel` 은 항상 `Buffer` 로 부터 데이터를 읽거나 `Buffer` 에 데이터를 쓴다.

## Channel 종류

- `FileChannel`
    - 파일에 데이터를 읽고 쓴다.
- `DatagramChannel`
    - UDP를 이용해 네트워크를 통해 데이터를 읽고 쓴다.
- `SocketChannel`
    - TCP를 이용해 네트워크를 통해 데이터를 읽고 쓴다.
- `ServerSocketChannel`
    - 들어오는 TCP 연결을 수신(listening)할 수 있다. 들어오는 연결마다 `SocketChannel` 이 만들어진다.

## SocketChannel

- 표준 자바 네트워킹에서 `Socket` 과 역할이 같다.
- TCP 네트워크 소켓에 연결된 객체

## ServerSocketChannel

- 표준 자바 네트워킹에서 `ServerSocket` 과 역할이 같다.
- TCP 연결을 수신 대기할 수 있는 `Channel`

## 참고

- [Java NIO와 멀티플렉싱 기반의 다중 접속 서버](https://jongmin92.github.io/2019/03/03/Java/java-nio/)
- [간단하게 만들어 본 kolint 기반의 nio-client-server 프로젝트](https://github.com/bestdevhyo1225/kotlin-nio-server-client)
