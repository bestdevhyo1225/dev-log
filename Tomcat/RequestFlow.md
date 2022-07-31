# 클라이언트의 요청이 Tomcat 엔진을 거쳐 Spring DispatcherServlet 요청이 도달하기 까지

클라이언트로 부터 요청을 받았을때 어떻게 `Spring DispatcherServlet` 까지 도달하는지에 대해 궁금했었고, 이를 코드와 함께 분석해봤다.

## Socket

서버 애플리케이션이 실행되면, `커널` 영역에서는 통신을 위한 `Socket` 을 생성하는데, `accept()` 를 통해 연결을 받고, 데이터를 `read() / write()` 처리한다.

1. socket 생성
2. bind()
3. listen()
4. accept()
5. read() / write()
6. close()

## Java NIO

`Tomcat` 에 요청이 진입되기 전에 알아야 할 부분이 있는데, `Java NIO(New Input Output)` 개념인데, 대표적인 특징으로는 `Channel`, `Buffer`, `Selector` 가
있다. `NIO` 에서 모든 `I/O` 는 `Channel` 로 시작되며, 4가지의 타입이 있는데, 여기서 확인할 부분은 대표적으로 다음과 같다.

- `SocketChannel` : `TCP` 프로토콜을 사용해서 네트워크를 통해 데이터를 읽고 쓴다.
- `ServerSocketChannel` : 들어오는 `TCP` 연결을 수신(listening)할 수 있다. 들어오는 연결마다 `SocketChannel` 이 만들어진다.

`Selector` 는 `NIO` 에서 가장 핵심적인 컴포넌트이다. `Channel` 들을 등록하여 `Channel` 로 들어오는 `이벤트(connect / accept / read / write)` 를 감지하여,
별도의 스레드를 할당해서 `이벤트` 를 처리할 수 있도록 한다.

```java
public class EchoServer {
    public static void main(String[] args) {
        // Selector 생성
        Selector selector = Selector.open();

        // Selector 에 생성되어 있는 SocketChannel 을 등록하는데, 감지할 이벤트를 정의한다.
        // - SelectionKey.OP_CONNECT
        // - SelectionKey.OP_ACCEPT
        // - SelectionKey.OP_READ
        // - SelectionKey.OP_WRITE
        socketChannel.register(selector, SelectionKey.OP_READ);

        // select() 메서드를 통해 등록된 이벤트 중에서 하나 이상의 SocketChannel 이 준비될 때까지 Block 된다.
        // 준비된 SocketChannel 의 수를 반환한다.
        selector.select();

        // selectedKeys() 메서드를 통해 준비된 SocketChannel 를 받는다.
        Set<SelectionKey> selectedKeys = selector.selectedKeys();

        Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

        // 준비된 SocketChannel 의 이벤트를 처리한다.
        while (keyIterator.hasNext()) {

            SelectionKey key = keyIterator.next();

            if (key.isAcceptable()) {
                // a connection was accepted by a ServerSocketChannel.
            } else if (key.isConnectable()) {
                // a connection was established with a remote server.
            } else if (key.isReadable()) {
                // a channel is ready for reading
            } else if (key.isWritable()) {
                // a channel is ready for writing
            }

            keyIterator.remove();
        }
    }
}
```