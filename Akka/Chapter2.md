# 핑퐁 액터

## ActorSystem 만들기

- 액터를 사용하기 위해서는 `ActorSystem` 이라는 객체를 만들어야 한다.
- 모든 액터는 `ActorSystem` 내부에서 동작을 수행한다.
- `ActorSystem` 은 `액터를 담는 컨테이너` 라고 생각해도 좋다.
- 액터들은 `Akka` 라이브러리가 제공하는 `스레드 스케줄링, 구성 파일, 로그 서비스 등을 공유한다.`

```java
ActorSystem actorSystem=ActorSystem.create("TestSystem");
```

- 하나의 애플리케이션에서 `하나의 ActorSystem` 을 사용하는 것이 일반적이다.
- 필요에 따라 하나의 애플리케이션에서 `여러 개의 ActorSystem` 을 만들어서 사용하는 것도 가능하다.
- 애플리케이션 안에서 여러 개의 `ActorSystem` 이 존재하거나, 해당 `ActorSystem` 이 원격 호스트를 포함하는 클러스터에 참여하는 경우에는 `이름이 중요하다.`

## Actor 만들기

```java
ActorRef ping=actorSystem.actorOf(Props.create(PingActor.class),"pingActor");
```

- `Actor` 를 만들기 위해서는 `Props`, `Actor 이름` 이 필요하다.

### Props

- `Actor` 를 만드는 데 필요한 구성 요소를 담는 `일종의 구성 파일 같은 클래스` 이다.
- 한 번 생성하면 값을 변경할 수 없는 `불변 객체` 이다. 따라서 필요한 곳에서 자유롭게 공유할 수 있다.

```java
// 기본 생성자
Props.create(PingActor.class);
```

```java
// 인수를 받아들이는 생성자
Props.create(PingActor.class, "args", "arg2");
```

### Actor 이름

- `Actor` 의 이름은 고유한 문자열이어야 한다.
- `Actor` 들은 트리와 비슷한 계층 구조를 형성한다. 그렇기 때문에 `경로` 라는 개념이 존재한다.
  - `Actor` 이름은 이러한 경로에서 사용될 때 의미를 갖는다.
- `actorOf(Props, 이름)` 메서드는 `Actor` 를 만들기 위한 팩토리 메서드이다.
  - `ActorSystem` 이나 `ActorContext` 객체로부터 호출할 수 있다.

```java
actorSystem.actorOf(Props.create(PingActor.class), "pingActor");
```

```java
context().actorOf(Props.create(PongActor.class, getSelf()), "pongActor");
```

## ActorRef

- `ActorSystem` 내에서 사용하는 `Actor` 는 모두 `ActorRef` 이다.
- `ActorRef` 는 `Actor` 객체를 감싸고 있는 형태라고 보면된다.
- `Actor` 객체에 퍼블릭 메서드를 선언해도 `접근할 수 없다.` 왜냐하면 `ActorRef` 이기 때문이다.
- `Actor` 객체를 `new 연산자로 생성할 수 없다.`
  - `akka.actor.ActorInitializationException` 예외가 발생한다.
  - `Actor` 는 오직 `actorOf` 메서드에 `Props` 와 `Actor 이름` 을 전달한 방식으로만 생성할 수 있고, `ActorRef` 를 생성한다.

## 장소 투명성

- 내가 사용하는 `Actor` 가 물리적으로 어디에 존재하는지 알 필요가 없음을 가리키는 개념이다.
- `ActorRef` 만 가지고 작업을 수행하기 때문에 `Actor` 가 어디에 있는지 알 필요가 없다.
- `Actor` 는 동일한 JVM에 존재할 수 있고, 동일한 컴퓨터 내에 다른 JVM에 존재할 수 있으며, 다른 컴퓨터의 JVM에 존재할 수도 있다.
- 하나의 JVM에서는 장소 투명성이 큰 의미를 갖지 않지만, `여러 개의 JVM이 클러스터를 형성하는 경우에는 위력을 나타낸다.`
  - 모든 코드가 `ActorRef` 를 대상으로 작성되어 있기 때문에 실제 `Actor` 의 물리적인 장소를 자유롭게 옮길 수 있다.

## 메시지 전송

- 일반적인 객체지향 프로그래밍에서는 `객체를 만든 다음 그 객체가 가지고 있는 메서드를 호출` 한다.
- `Actor` 세계에서는 `액터를 만든 다음 그 액터에 메시지를 전송` 한다.
- `tell`, `ask` 메서드가 있다.
- `tell` 메서드는 `메시지, 발신자` 를 인수로 받아들인다.

```java
ping.tell("start", ActorRef.noSender());
```

- `UntypedActor` 또는 `UntypedAbstractActor` 를 상속 받은 경우, `Actor` 가 받아 들이는 메시지 타입이 미리 정해져 있지 않기 때문에 `어떤 타입의 객체라도 보낼 수 있다.`
- 발신자는 반드시 `ActorRef` 타입을 가져야 한다.
  - 즉, 발신자는 어떠한 `Actor` 이다.

### 발신자 종류

- `getSelf()` : `Actor` 자신의 `ActorRef` 객체를 리턴한다. 
- `getSender()` : 현재 처리중인 메시지를 보내 온 발신자를 다음 `Actor` 에게 그대로 포워드 할 때 사용한다.
- `ActorRef.noSender()` : 발신자가 아무 의미 없는 경우에 사용한다. (null 값)

## 메일박스

- `Actor` 마다 수신 받은 메시지를 저장할 수 있는 고유 공간이다.
- 여러 개의 `Actor` 가 하나의 메일 박스를 공유하는 경우도 있다.
- 메모리 사용량이나 역류(Back-Pressure) 등을 고민할 때 메일박스를 염두해 두고, 일반적인 경우에는 존재를 신경 쓰는 경우가 거의 없다.
- 자바의 `ConcurrentLinkedQueue` 를 이용해서 구현 되었다. 메일박스 종류에 따라서 다른 클래스를 이용하기도 한다.
- `Actor` 하나를 떼어 놓고 생각해보면, Node.js의 이벤트 처리 방식과 비슷하다.
  - 이벤트 큐, 디스패치 큐
  - 원리적으로 이 모든 패턴은 비슷한 철학을 공유한다.
  - 전달되는 메시지가 메일박스에 차곡차곡 쌓이고 `Actor` 는 **`그 메시지를 한 번에 하나씩 처리하는 것이다.`**

## onReceive

- `ActorSystem` 은 내부에 `스레드 풀을 보유` 하고 있다. `디스패처` 라고 부르기도 한다.
- `Actor` 는 스레드를 할당 받아 처리하고, 스레드 풀에 다시 반납한다.
- 동시에 실행되는 스레드의 수는 CPU 코어의 숫자에 의해서 제약된다.
- `onReceive` 를 할당하는 스레드는 `반드시 1개` 로 국한된다.
  - 디스패처가 하나의 `Actor` 에 2개 또는 그 이상의 스레드를 동시에 할당하지 않기 때문이다.
  - `Akka` 혹은 `ActorSystem` 에서는 **`전통적인 멀티스레드 환경 문제(ex 동시성 이슈) 고민하지 않아도 되는 이유이다.`**
  - 사용했던 동일한 스레드를 계속해서 할당하는 것이 아니고, `시점이 달라지면 다른 스레드가 반드시 1개씩 할당되는 것이다.`

```java
@Override
public void onReceive(Object message) throws Exception {
    TODO("...")    
}
```

### 짚고 넘어가야할 중요한 부분

- `Actor` 의 철학은 기본적으로 `모든 것이 비동기적` 이고, `어느 것도 중단(Blocking)되지 않는다는 데` 있다.

## Actor LifeCycle

- 생성 (created)
- 재시작 (restarted)
- 멈춤 (stopped)
- `Actor` 가 재시작하는 경우에는 `ActorSystem` 어딘가에서 에러가 발생해서 다시 `Actor` 가 만들어지는 것을 의미한다.
  - `ActorRef` 가 내부에 저장된 `Actor` 객체를 버리고, 새로운 `Actor` 객체를 생성한다.
- `Actor` 의 멈춤은 최종적인 종료를 의미한다.
