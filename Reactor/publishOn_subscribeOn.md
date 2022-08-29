# publishOn, subscribeOn 차이

## publishOn

### 사용 방법

- `publishOn(Schedulers.newSingle(name))`
- `publishOn(Schedulers.parallel())`
- `publishOn(Schedulers.boundedElastic())`

### 특징

- 데이터 `소비(consume)` 가 느릴 경우에 사용한다. (즉, `Subscriber` 에서 소비하는 속도가 느리다는 의미)
- `Subscriber` 가 별도의 스레드를 통해 데이터를 소비한다.

### 테스트

```kotlin
it("subscriber 처리가 별도의 스레드에서 실행된다. (publishOn 사용)") {
    // 'publishOn' 은 데이터 소비가 느릴 경우에 사용한다.
    Flux.range(1, 10)
        .log()
        .publishOn(Schedulers.newSingle("pub"))
        .subscribe { consumer -> logger.info { "consumer: $consumer" } }
}
```

### 결과

- `Publisher` 에서 `onSubscribe(...)`, `request(...)`, `onNext(...)` 를 통해 데이터를 생성할 때는 기존에 `시작했던 스레드(pool-1-thread-1)`에서
  처리가 된다.

<img width="898" alt="스크린샷 2022-08-30 오전 12 50 44" src="https://user-images.githubusercontent.com/23515771/187242024-87acc2eb-98a5-4a39-87ae-0e1ff401d433.png">

- `Subscriber` 에서는 `publishOn(Schedulers.newSingle("pub"))` 코드에 의해 생성된 `스레드(pub-1)` 에서 데이터를 소비한다.

<img width="731" alt="스크린샷 2022-08-30 오전 12 52 11" src="https://user-images.githubusercontent.com/23515771/187242306-9bdfafb6-1b89-4e22-bdcc-0950ad43601f.png">

## subscribeOn

### 사용 방법

- `subscribeOn(Schedulers.newSingle(name))`
- `subscribeOn(Schedulers.parallel())`
- `subscribeOn(Schedulers.boundedElastic())`

### 특징

- 데이터 `생성(publish)` 이 느릴 경우에 사용한다. (즉, `Publisher` 에서 데이터 생성 속도가 느리다는 의미)
- `Publisher` 가 별도의 스레드를 통해 데이터를 생성한다.

### 테스트

```kotlin
it("publisher 처리가 별도의 스레드에서 실행된다. (subscribeOn 사용)") {
    // 'subscribeOn' 은 데이터 생성이 느릴 경우에 사용한다.
    Flux.range(1, 10)
        .log()
        .publishOn(Schedulers.newSingle("pub"))
        .subscribeOn(Schedulers.newSingle("sub"))
        .subscribe { consumer -> logger.info { "consumer: $consumer" } }
}
```

### 결과

- `Publisher` 에서 `onSubscribe(...)`, `request(...)`, `onNext(...)` 를 통해 데이터를 생성할
  때 `subscribeOn(Schedulers.newSingle("sub"))` 코드에 의해서 생성된 `스레드(sub-1)` 에서 데이터를 생성한다.

<img width="1009" alt="스크린샷 2022-08-30 오전 12 58 24" src="https://user-images.githubusercontent.com/23515771/187243518-5b70c3dc-7243-4fe3-8f0a-019aa1465d24.png">

## 참고

- [리액티브 프로그래밍 시리즈 3 - 스레드 스케쥴링 (Thread scheduling)](https://do-study.tistory.com/117)
  - 해당 링크에서 `2. Reactor publishOn, subscribeOn` 참고 
