# 클라이언트의 요청이 Tomcat을 거쳐 Spring DispatcherServlet에 도달하기까지

클라이언트로 부터 요청을 받았을때 어떻게 `Spring DispatcherServlet` 까지 도달하는지에 대해 궁금했었고, 이를 코드와 함께 분석해봤다.

## Socket

`클라이언트 - 서버` 프로그램이 실행되면, `커널` 영역에서는 통신을 위한 `Socket` 을 생성하는데, 아래와 같은 과정을 거치게 된다.

<img width="444" alt="스크린샷 2022-07-31 오후 7 01 14" src="https://user-images.githubusercontent.com/23515771/182021108-899ea452-a530-4f5c-a7ea-a7c2c4750782.png">

여기서 `서버` 프로그램의 과정을 확인해보면 `Socket` 생성 후, `bind()`, `listen()` 과정을 거친다. 그리고 `accept()` 를 통해 클라이언트로 부터 `Connection` 을 받고,
이후에 데이터를 `read() / write()` 처리한다.

## Java NIO

`Tomcat` 에 요청이 진입되기 전에 알아야 할 부분이 있는데, `Java NIO(New Input Output)` 개념이다. 대표적인 특징으로는 `Channel`, `Buffer`, `Selector` 가
있다. `NIO` 에서 모든 `I/O` 는 `Channel` 로 시작되며, 4가지의 타입이 있는데, 여기서 확인할 부분은 다음과 같다.

- `SocketChannel` : `TCP` 프로토콜을 사용해서 네트워크를 통해 데이터를 읽고 쓴다.
- `ServerSocketChannel` : 들어오는 `TCP` 연결을 수신(listening)할 수 있다. 들어오는 연결마다 `SocketChannel` 이 만들어진다.

`Selector` 는 `NIO` 에서 가장 핵심적인 컴포넌트이다. `Channel` 을 등록한 다음에 `Channel` 로 들어오는 `이벤트(connect / accept / read / write)` 를
감지하여,
`이벤트` 를 처리할 수 있도록 한다.

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

## Tomcat

이제부터 클라이언트의 요청이 `Tomcat` 을 거쳐 `Spring` 의 `DispatcherServlet` 로 전달되는 과정을 확인해보려고 한다.

### NioEndpoint, AbstractEndpoint 클래스

우선 커널 영역에 있는 `Socket` 을 통해 데이터가 전달되면 `NioEndpoint` 클래스의 진입점으로 들어오게 된다. 그리고 해당 클래스는 `AbstractEndpoint` 클래스를 상속받고
있다.`NioEndpoint` 클래스의 핵심은 `Acceptor`, `Poller`, `SocketProcessor(Worker)` 이다. 이 `3개의 객체` 를 통해 데이터를 전달한다.

#### Acceptor 클래스

- `ServerSocketChannel` 에 `accept()` 를 통해서 커넥션을 얻고, 데이터를 송수신 할 `SocketChannel` 를 생성한다.
- `Runnable` 인터페이스를 구현했고, `데몬 스레드` 로 실행된다.

#### Poller 클래스

- `PollerEvent` 를 `Queue` 에 담고, 꺼내서 처리한다.
- `Selector` 컴포넌트로 `SocketChannel` 을 관리한다. (등록 및 이벤트 확인)
- `Runnable` 인터페이스를 구현했고, `데몬 스레드` 로 실행된다.

#### SocketProcessor 클래스 (Worker)

- `NioEndpoint` 내부에는 우리가 평상시에 `application.yml` 에서 지정하는 `tomcat` 옵션들이 있는데, 여기서 `maxThreads` 및 `minSpareThreads` 값을
  확인하여, `Thread Pool` 을 미리 생성하고 확보해두고 있는다.
- `Poller` 에 의해서 처리할 이벤트가 생기면, `Thread Pool` 에서 `Thread` 를 하나 할당하고, 내부의 `doRun()` 메소드에 의해 이벤트를 처리한다.
- `SocketProcessorBase<S>` 클래스를 상속받았고, 최상위로 올라가면 `Runnable` 인터페이스를 구현했다.

### 클라이언트 요청이 Tomcat에서 Spring의 DispatcherServlet까지 도달하는 과정

> Acceptor 클래스 내부의 `run` 메서드

```java
public class Acceptor<U> implements Runnable {

    @Override
    public void run() {
        try {
            // 생략...

            while (/* 생략 */) {
                // 생략...
                try {
                    U socket = null;
                    try {
                        // Accept the next incoming connection from the server
                        // socket
                        socket = endpoint.serverSocketAccept();
                    } catch (Exception ioe) {
                        // 생략...
                    }

                    // 생략...

                    if (/* 생략 */) {
                        // setSocketOptions() will hand the socket off to
                        // an appropriate processor if successful
                        if (!endpoint.setSocketOptions(socket)) {
                            // 생략
                        }
                    } else {
                        // 생략
                    }
                } catch (Throwable t) {
                    // 생략...
                }
            }
        } finally {

        }
    }
}
```

가장 먼저 `endpoint.serverSocketAccept()` 를 통해 새롭게 생성된 `SocketChannel` 반환 받는다. 그리고 `endpoint.setSocketOptions(socket)` 에 의해서
이후의 요청을 처리한다.

> NioEndpoint 클래스 내부의 `setSocketOptions` 메서드

```java
public class NioEndpoint extends AbstractJsseEndpoint<NioChannel, SocketChannel> {

    @Override
    protected boolean setSocketOptions(SocketChannel socket) {
        NioSocketWrapper socketWrapper = null;
        try {
            NioChannel channel = null;
            if (nioChannels != null) {
                channel = nioChannels.pop();
            }
            if (channel == null) {
                // 생략...

                if (isSSLEnabled()) {
                    channel = new SecureNioChannel(bufhandler, this);
                } else {
                    channel = new NioChannel(bufhandler);
                }
            }
            NioSocketWrapper newWrapper = new NioSocketWrapper(channel, this);
            channel.reset(socket, newWrapper);
            connections.put(socket, newWrapper);
            socketWrapper = newWrapper;

            // SocketChannel 을 Selector 에 등록하기 위해서는 non-blocking 모드어야 한다.
            socket.configureBlocking(false);

            if (getUnixDomainSocketPath() == null) {
                socketProperties.setProperties(socket.socket());
            }

            socketWrapper.setReadTimeout(getConnectionTimeout());
            socketWrapper.setWriteTimeout(getConnectionTimeout());
            socketWrapper.setKeepAliveLeft(NioEndpoint.this.getMaxKeepAliveRequests());

            // 핵심!
            // SocketWrapper 객체를 Poller 객체에 등록한다.
            poller.register(socketWrapper);
            return true;
        } catch (Throwable t) {
            // 생략
        }
        // Tell to close the socket if needed
        return false;
    }
}
```

위의 코드에서 일부 생략한 부분이 있지만, 여기서 핵심은 `NioChannel` 를 생성하고, `SocketChannel` 과 `NioChannel` 를 `NioSocketWrapper` 객체로 감싼
후, `poller.register(socketWrapper)` 코드에 의해서 `Poller` 로 전달된다.

> NioEndpoint 클래스 내부에 있는 Poller 클래스의 `register` 메소드

```java
public class NioEndpoint extends AbstractJsseEndpoint<NioChannel, SocketChannel> {

    public class Poller implements Runnable {

        public void register(final NioSocketWrapper socketWrapper) {
            socketWrapper.interestOps(SelectionKey.OP_READ);//this is what OP_REGISTER turns into.
            PollerEvent event = null;
            if (eventCache != null) {
                event = eventCache.pop();
            }
            if (event == null) {
                // NioSocketWrapper 객체를 생성자 인자로 받아 PollerEvent 객체를 생성한다.
                event = new PollerEvent(socketWrapper, OP_REGISTER);
            } else {
                event.reset(socketWrapper, OP_REGISTER);
            }

            // PollerEvent 등록
            addEvent(event);
        }
    }
}
```

중요한 점이 `Poller` 클래스 내부의 `register` 메소드에서 `NioSocketWrapper` 객체를 생성자 인자로 받아 `PollerEvent` 객체를 생성하고, `addEvent(event)`
메서드를 통해 `PollerEvent` 를 어딘가에 저장한다.

> NioEndpoint 클래스 내부에 있는 Poller 클래스의 `addEvent` 메소드

```java
public class NioEndpoint extends AbstractJsseEndpoint<NioChannel, SocketChannel> {

    public class Poller implements Runnable {

        private final SynchronizedQueue<PollerEvent> events = new SynchronizedQueue<>();

        private void addEvent(PollerEvent event) {
            events.offer(event);

            // 로직이 있지만 생략...
        }
    }
}
```

위에서 말한 것처럼 `addEvent` 메서드를 호출하면, 어딘가에 `PollerEvent` 를 저장한다고 했는데, 실제로는
`SynchronizedQueue` 에 `PollerEvent` 를 등록한다.

> NioEndpoint 클래스 내부에 있는 Poller 클래스의 `run`, `events` 메소드

```java
public class NioEndpoint extends AbstractJsseEndpoint<NioChannel, SocketChannel> {

    public class Poller implements Runnable {

        private final SynchronizedQueue<PollerEvent> events = new SynchronizedQueue<>();

        public boolean events() {
            boolean result = false;

            PollerEvent pe = null;
            for (int i = 0, size = events.size(); i < size && (pe = events.poll()) != null; i++) {
                result = true;

                NioSocketWrapper socketWrapper = pe.getSocketWrapper();
                SocketChannel sc = socketWrapper.getSocket().getIOChannel();

                int interestOps = pe.getInterestOps();

                if (sc == null) {
                    // 생략
                } else if (interestOps == OP_REGISTER) {
                    try {
                        sc.register(getSelector(), SelectionKey.OP_READ, socketWrapper);
                    } catch (Exception x) {
                        // 생략
                    }
                } else {
                    final SelectionKey key = sc.keyFor(getSelector());
                    if (key == null) {
                        // 생략
                    } else {
                        // 생략
                    }
                }

                // 생략
            }

            return result;
        }

        @Override
        public void run() {
            // Loop until destroy() is called
            while (true) {

                boolean hasEvents = false;

                try {
                    if (!close) {
                        // events() 메서드 호출
                        hasEvents = events();

                        if (wakeupCounter.getAndSet(-1) > 0) {
                            // selectorNow() 메서드 호출
                            keyCount = selector.selectNow();
                        } else {
                            // selector() 메서드 호출
                            keyCount = selector.select(selectorTimeout);
                        }
                        wakeupCounter.set(0);
                    }

                    // 나머지 코드 생략

                } catch (Throwable x) {
                    // 코드 생략
                }

                Iterator<SelectionKey> iterator =
                    keyCount > 0 ? selector.selectedKeys().iterator() : null;

                while (iterator != null && iterator.hasNext()) {
                    SelectionKey sk = iterator.next();
                    iterator.remove();
                    NioSocketWrapper socketWrapper = (NioSocketWrapper) sk.attachment();

                    if (socketWrapper != null) {
                        processKey(sk, socketWrapper);
                    }
                }

                // 생략
            }

            // 생략
        }
    }
}
```

가장 처음에 실행되는 `events()` 는 `Queue` 에서 `PollerEvent` 를 꺼내서 이벤트에 따라 `Selector` 에 등록하거나
`Selector` 에서 `Key` 들을 가져온다. 그 다음에 `selectNow()` 또는 `select(selectorTimeout)` 메서드에 의해서
등록된 이벤트중에서 하나 이상의 `SocketChannel` 이 준비될 때까지 Block 상태가 되며, 준비된 채널이 있는 경우 채널의 수를 반환한다.
그리고 `selector.selectedKeys()` 를 통해 실제로 준비된 `SocketChannel` 들을 받아서
`processKey(SelectionKey, NioSocketWrapper)` 메서드를 호출한다.

> NioEndpoint 클래스 내부에 있는 Poller 클래스의 `processKey` 메소드

```java
public class NioEndpoint extends AbstractJsseEndpoint<NioChannel, SocketChannel> {

    public class Poller implements Runnable {

        private final SynchronizedQueue<PollerEvent> events = new SynchronizedQueue<>();

        protected void processKey(SelectionKey sk, NioSocketWrapper socketWrapper) {
            try {
                if (close) {
                    // 생략
                } else if (sk.isValid()) {
                    if (sk.isReadable() || sk.isWritable()) {
                        if (socketWrapper.getSendfileData() != null) {
                            // 생략
                        } else {
                            // 생략

                            boolean closeSocket = false;

                            if (sk.isReadable()) {
                                if (socketWrapper.readOperation != null) {
                                    // 생략
                                } else if (socketWrapper.readBlocking) {
                                    // 생략
                                }
                                // processSocket 메서드를 호출하는 부분
                                else if (!processSocket(socketWrapper, SocketEvent.OPEN_READ, true)) {
                                    closeSocket = true;
                                }
                            }
                            if (!closeSocket && sk.isWritable()) {
                                if (socketWrapper.writeOperation != null) {
                                    // 생략
                                } else if (socketWrapper.writeBlocking) {
                                    // 생략
                                }
                                // processSocket 메서드를 호출하는 부분
                                else if (!processSocket(socketWrapper, SocketEvent.OPEN_WRITE, true)) {
                                    closeSocket = true;
                                }
                            }

                            // 생략
                        }
                    }
                } else {
                    // 생략
                }
            } catch (CancelledKeyException ckx) {
                // 생략
            } catch (Throwable t) {
                // 생략
            }
        }
    }
}
```

`processKey` 내부에서는 `SelectionKey.isReadable()` 또는 `SelectionKey.isWritable()` 조건에 따라
`SocketChannel` 에서 전달된 데이터를 처리하는 `processSocket(NioSocketWrapper, SocketEvent, dispatcherFlag)` 메서드를 호출한다.

> AbstractEndpoint 클래스의 `processSocket` 메서드

```java
public abstract class AbstractEndpoint<S, U> {

    public boolean processSocket(SocketWrapperBase<S> socketWrapper,
                                 SocketEvent event, boolean dispatch) {
        try {
            if (socketWrapper == null) {
                return false;
            }

            SocketProcessorBase<S> sc = null;

            if (processorCache != null) {
                sc = processorCache.pop();
            }

            if (sc == null) {
                // SocketProcessor 객체를 만든다.
                sc = createSocketProcessor(socketWrapper, event);
            } else {
                sc.reset(socketWrapper, event);
            }

            // Executor 객체를 가져온다.
            Executor executor = getExecutor();
            if (dispatch && executor != null) {
                // Executor 객체를 통해 ThreadPool 에서 Thread 하나를 할당 받아서 실행된다.
                executor.execute(sc);
            } else {
                sc.run();
            }
        } catch (RejectedExecutionException ree) {
            // 생략
            return false;
        } catch (Throwable t) {
            // 생략
            return false;
        }

        return true;
    }
}
```

`SocketProcessor` 객체가 존재하지 않으면, `createSocketProcessor` 메서드를 통해 `SocketProcessor` 를 생성하고,
`Executor` 객체를 통해 `ThreadPool` 에서 하나의 `Thread` 를 할당받아 실행된다.

```yaml
server:
  port: 9000
  tomcat:
    threads:
      min-spare: 20
      max: 100
```

- **우리가 평상시 `application.yml` 에 설정한 값을 참고해서 `ThreadPool` 을 미리 생성해두고, `SocketProcessor` 객체가 생성될 때마다 `Thread` 를 하나씩
  할당해준다.**

- **즉, `Request` 당 하나의 `Thread` 를 할당해준다고 이해하면 된다.**

> NioEndpoint 클래스 내부에 있는 SocketProcessor 클래스의 `doRun` 메소드

```java
public class NioEndpoint extends AbstractJsseEndpoint<NioChannel, SocketChannel> {

    protected class SocketProcessor extends SocketProcessorBase<NioChannel> {

        public SocketProcessor(SocketWrapperBase<NioChannel> socketWrapper, SocketEvent event) {
            super(socketWrapper, event);
        }

        @Override
        protected void doRun() {
            try {
                // 나머지 코드 생략

                if (handshake == 0) {
                    SocketState state = SocketState.OPEN;

                    // Socket 으로부터 요청을 처리한다.
                    if (event == null) {
                        state = getHandler().process(socketWrapper, SocketEvent.OPEN_READ);
                    } else {
                        state = getHandler().process(socketWrapper, event);
                    }

                    if (state == SocketState.CLOSED) {
                        // 생략
                    }
                } else if (handshake == -1) {
                    getHandler().process(socketWrapper, SocketEvent.CONNECT_FAIL);

                    // 생략
                } else if (handshake == SelectionKey.OP_READ) {
                    // 생략
                } else if (handshake == SelectionKey.OP_WRITE) {
                    // 생략
                }
            } catch (CancelledKeyException cx) {
                // 생략
            } catch (VirtualMachineError vme) {
                // 생략
            } catch (Throwable t) {
                // 생략
            } finally {
                // 생략
            }
        }
    }
}
```

`doRun` 메서드에서는 `getHandler().process(NioSocketWrapper, SocketEvent)` 메서드를 호출하는데,
`Socket` 으로부터 클라이언트 요청을 처리 본격적으로 처리하게 된다. `getHandler()` 에서 반환되는 객체는 `coyote` 패키지에 있는
`AbstractProtocol` 클래스의 `process` 메서드를 호출한다. 
