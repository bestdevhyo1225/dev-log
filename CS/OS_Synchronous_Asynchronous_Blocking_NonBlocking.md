# Synchronous / Asynchronous, Blocking / Non-Blocking 정리

## Synchronous, Asynchronous

`Synchronous(동기)` 와 `Asynchronous(비동기)` 는 **`통신 메커니즘`** 에 대한 것이다. 즉, `어떤 방식으로 어떻게 통신하는지` 에 대한 것으로 이해하면 된다.

### Synchronous

`Synchronous(동기)` 방식은 `함수를 호출한 곳(클라이언트)` 에서 결과를 받을 때까지 기다려야 한다는 특징을 가지고 있다.

<img width="566" alt="image" src="https://user-images.githubusercontent.com/23515771/169636822-3e7accea-e13d-4b5b-bad8-edbb80e52cf2.png">

### Asynchronous

`Asynchronous(비동기)` 방식은 결과 없이 우선 리턴을 한다. 그리고 처리가 되면, 콜백을 통해 `함수를 호출한 곳(클라이언트)` 에 결과를 전달한다.

<img width="566" alt="image" src="https://user-images.githubusercontent.com/23515771/169636916-187b281a-6c52-4623-8d6f-e7f8e5d23c9d.png">

## Blocking, Non-Blocking

`Blocking` 과 `Non-Blocking` 은 **`호출한 주체의 상태`** 의 초점을 맞추면 된다.

### Blocking

`Blocking` 은 결과를 얻기 전까지 현재 스레드 또는 호출한 주체의 `상태` 를 정지 시킨다. 즉, 결과를 받을 때까지 아무 작업도 하지 않고 기다리고 있는 상태이다.

### Non-Blocking

`Non-Blocking` 은 현재 스레드 또는 호출한 주체의 `상태` 가 정지되지 않으며, 결과를 받을 때까지 다른 작업을 수행할 수 있다.
