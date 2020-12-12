# Reactive Streams

`Spring Webflux`를 사용하기전에 `Reactive Streams`에 대해 자세히 공부하고 싶었고, 이를 정리하려고 한다.

<br>

## Reactive Streams Interface

Reactive Streams 인터페이스는 아래와 같이 4개로 구성되어 있다. 해당 인터페이스를 직접 구현해서 `Reactive Programming`을 할 수 있다.

- Processor

- Publisher

- Subscriber

- Subscription

> Publisher

```java
public interface Publisher<T> {
  public void subscribe(Subscriber<? super T> subscriber);
}
```

> Subscriber

```java
public interface Subscriber<T> {
  public void onSubscribe(Subscription subscription);
  public void onNext(T t);
  public void onError(Throwable throwable);
  public void onComplete();
}
```

> Subscription

```java
public interface Subscription {
  public void request(long n);
  public void cancel();
}
```

<br>

## Reactive Streams 기본 흐름

> Subscriber가 Publisher를 subscribe하면, Publisher는 데이터 또는 시퀀스를 전달하게 된다. 아래의 내용을 통해 조금 더 자세하게 알아보자.

```java
MyPublisher.subscribe(MySubscriber);
```

> Publisher의 subscribe() 메소드를 호출하게 되면, 메소드 내부에서 subscriber.onSubscribe() 호출하며, Publisher에게 데이터 또는 시퀀스를 요청하는 과정이다.

```java
public class MyPublisher implements Publisher<Integer> {
  @Override
  public void subscribe(Subscriber subscriber) {
    subcriber.onSubscribe(new MySubscription(/* 생략 */));
  }
}
```

> onSubscribe() 메소드 내부에서 Subscription의 request(n) 메소드를 호출하여 데이터 전송 요청을 하게 되면, Publisher 에서는 0 ~ N개의 데이터 또는 시퀀스를 Subsciber에게 전달한다.

```java
public class MySubscriber implements Subscriber<Integer> {
  private Subscription subscription;

  @Override
  public void onSubscribe(Subscription subscription) {
    this.subscription = subscription;
    this.subscription.request(REQUEST_COUNT);
  }

  @Override
  public void onNext() {}

  @Override
  public void onError() {}

  @Override
  public void onComplete() {}
}
```

> 이 과정에서 상황에 따라 onNext(), onComplete(), onError()를 호출한다.

```java
public class MySubscribtion implements Subscription {
  @Override
  public void request(long n) {
    // 내부로직에 따라 onNext(), onError(), onComplete() 호출됨
  }

  @Override
  public void cancel() {}
}
```

이쯤에서 간단히 정리해보자면, `Subscriber`가 `Publisher`에게 `Request`하는 과정을 `Back-Pressure`라고 표현하고, `Request`하는 데이터 또는 시퀀스의 흐름을 제어할 수 있다. 즉, `Request(1)`를 요청하면 1개의 데이터만 요청할 수 있고, `Request(MAX)`를 요청하면 최대 값에 해당하는 데이터를 요청할 수 있는것이다.

> 로그를 보다보면, request(unbounded)에서 unbounded를 확인 할 수 있는데 unbounded가 의미하는 것은 무엇일까?

- 내부 로직에서 MAX값을 설정하게 된다는 것을 의미한다.
- Back-Pressure기능을 활용하지 않고, 모든 데이터를 요청하는 것이다.
- Back-Pressure기능을 사용해서, request(n)에서 n을 변경하고 싶다면, subscriptionConsumer를 넘겨줘야 한다.

<br>

## 참고

- [Project Reactor](https://brunch.co.kr/@springboot/152)
- [Reactor 3 Reference Guide](https://projectreactor.io/docs/core/release/reference/#about-doc)
