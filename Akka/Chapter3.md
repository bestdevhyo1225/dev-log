# 아카 계층구조

## 아카 계층구조

- `Actor` 는 메시지를 수신한다.
- `Actor` 는 메시지를 송신한다.
- `Actor` 는 다른 `Actor` 를 만든다.
- `PingActor` 가 `PongActor` 의 부모이고, `PongActor` 는 `PingActor` 의 자식이다.
- 어떠한 `Actor` 가 다른 `Actor` 를 만들면, 둘 간의 관계는 `부모-자식` 관계가 된다.

## 보내고 잊기

- `ActorSystem` 에서 `tell` 메서드를 이용해서 메시지를 전달하는 것은 철저하게 `비동기적` 이다.
- `tell` 메서드를 호출하면, 해당 메시지가 실제 목적지에 도달하는지 여부와 상관없이 즉시 리턴된다. 즉, `보내고 잊기` 라고 일컬어진다.
  - `보내고 잊기` 는 `ActorSystem` 을 구성하는 핵심 원리이다.
- 편지를 보내는 과정이라 비슷하다. 편지를 우편에 넣고 자리를 떠난다.
- 만약 편지를 보내는 과정이 동기적이라면, 수신인이 편지를 받을때 까지 다른 일을 할 수 없다.
- 비동기적인 동작을 동기적인 방식의 프로그래밍과 혼동하는 사례가 많은데, 대표적으로 아래의 2가지가 있다.
  - `작업완료 확인` 의 어려움
  - `메시지 순서 확인` 의 어려움

## 작업의 완료

- `Ping1Actor` 가 자기 할 일을 다 했다고 말하는 것과 `Ping2Actor` 와 `Ping3Actor` 가 어떤 업무를 수행하는 것 사이에는 아무런 함수 관계가 없다.
  - 모든 `Actor` 가 `독립적` 이고, `비동기적` 인 동작으로 분해되기 때문이다.

## 메시지 순서

```text
P: PingActor
P1: Ping1Actor
P2: Ping2Actor
P3: Ping3Actor
w: "work" 메시지
d: "done" 메시지
(X m Y): 액터 X가 Y에게 메시지 m을 보내는 동작
```

- 위와 같이 수학적 기호르 정해 놓고, 다음과 같이 설명할 수 있다.

```text
a. (P w P1)
b. (P1 w P2)
c. (P1 w P3)
d. (P1 d P)
e. (P2 d P)
f. (P3 d P)
```

- 위와 같이 메시지가 순서대로 전달될 것이라고 생각하는데, 메시지 전달되는 순서는 `비결정적` 이다.
- `d`, `e`, `f` 사이에는 어떠한 순서도 보장되어 있지 않다.
- `Akka` 에서 보장하는 메시지 순서는 오직 `발신자` 와 `수신자` 가 동일한 경우로 국한된다.

```text
(A m1 B)
(A m2 B)
```

- 위의 경우에는 `m1` 이 전송되고, 그 다음에 `m2` 가 전송된다.

```text
(A m1 B)
(B m1 C)
(A m2 B)
(B m2 C)
```

- 위와 같은 상황에서 `C` 의 입장에서는 `m1` 이 `m2` 보다 먼저 도착할 수 없다고 봐야한다.
  - `A` 에서 직접 `C` 로 보낸다면, `m1` 이 `m2` 보다 먼저 도착한다고 말할 수 있다.
- 어떤 `Actor` 의 Pair 가 있을 때, `첫 번째 Actor` 가 `두 번째 Actor` 에게 `직접` 메시지를 보내는 경우에만 순서가 보장된다.
- 중간에 `다른 Actor` 가 끼어드는 경우에는 `보장이 성립되지 않는다.`
- `메시지 전달이 보장되지 않기 때문에` 메시지 중 어떤 것이라도 중간에 사라질 수 있다.

## 테크닉

- 실무에서는 `Actor` 아래의 상황을 고려해서 더 정교한 구조를 사용해야 한다.
  - ex) `done 메시지가 도착해서 count 값을 1로 증가 시켰는데, 다시 work 메시지가 오면 어떻게 할 것인가?`
- `tell` 메서드의 두 번째 인수 `ActorRef` 에 들어가는 주소는 메시지를 물리적으로 전달해주는 `Actor` 가 아닌 경우가 많다.
  - 예제에서는 `PingActor` 의 주소를 `Ping2Actor` 와 `Ping3Actor` 에게 전달했기 때문에 모든 작업이 완료되는 것을 `PingActor` 가 정확하게 인지할 수 있도록 만들었다.

## Actor 내부 상태와 스레드

- `count` 와 같은 객체의 내부 변수는 `Actor` 의 행위를 조절하는 `상태` 에 해당한다.
- 멀티 스레드를 고려해 `AtomicInteger`, `ConcurrentHashMap` 을 사용하는 경우가 있지만, `Actor` 에서는 그러한 보호 장치가 필요 없다.
  - `Actor` 에 내부 상태에 접근하는 것은 언제나 `하나의 스레드` 로 국한되어 있다. 따라서 `멀티 스레드와 관련된 고민을 할 필요가 없어지는 것이다.`
  - 사람들이 `동시성 코드` 를 작성하기 위한 차세대 방법으로 `Akka` 를 추천할 때, 염두에 두는 내용이 바로 이 부분이다.
- `Akka` 가 멀티 스레드와 관련된 문제를 `모두 해결해주는 것은 절대 아니다.`
  - `onReceive` 메서드에서 어떤 외부의 객체에 접근하는 경우가 얼마든지 있을 수 있기 때문이다.
  - 그 객체가 `멀티 스레드 문제로부터 보호되어 있다는 사실을 반드시 확인해야 하고`, 보호되어 있지 않으면 스스로 보호방법을 강구해야 한다.
  - `Actor` 는 위의 경우까지 고려해서 문제를 해결해주지 않는다.