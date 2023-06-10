# 모니터 (Monitor)

## 모니터란?

- `상호 배제(Mutual Exclusion)` 를 보장한다.
- 조건에 따라 스레드가 `대기(Waiting)` 상태로 전환하는 기능을 제공한다.

## 모니터는 언제 사용되나?

- 한 번의 하나의 스레드만 실행될 때 사용된다.
- 여러 스레드와 `협업(Cooperation)` 이 필요할 때 사용한다.

## 모니터 구성요소

- Mutex
- Condition Variable

## Mutex

- `임계 영역(Critical Section)` 에서 `상호 배제(Mutual Exclusion)` 를 보장하는 장치
- `임계 영역(Critical Section)` 에 진입하려면, `Mutex Lock` 을 취득해야 한다.
- `Mutex Lock` 을 취득하지 못한 스레드는 `Queue` 에 들어간 후, `대기` 상태로 전환된다.
- `Mutex Lock` 을 취득한 스레드가 작업을 끝내고 `Lock` 을 반환하면, `Queue` 에 `대기` 상태로 있던 스레드 중 하나가 실행된다.

## Condition Variable

- `Wating Queue` 를 기본적으로 가진다.
- `Wating Queue` 는 조건이 충족되길 기다리는 스레드들이 `대기` 상태로 머무는 곳이다.

## Condition Variable 에서 주요 동작(Operation)

### Wait

- 스레드가 자기 자신을 `Condition Variable` 의 `Wating Queue` 에 넣고, `대기` 상태로 전환한다.

### Signal

- `Wating Queue` 에서 `대기` 중인 스레드 중에 하나를 깨우는 역할을 한다.

### BroadCast

- `Wating Queue` 에서 `대기` 중인 스레드를 전부 깨우는 역할을 한다.

## 두 개의 Queue

### Entry Queue

- `임계 영역(Critical Section)` 에 진입을 기다리는 큐이며, `Mutex` 에서 관리하는 `Queue` 이다.

### Wating Queue

- 조건이 충족되기를 기다리는 `Queue` 이며, `Condition Variable` 이 관리하는 `Queue` 이다.

## Bounded Producer, Consumer Problem

### Producer Problem

<img width="777" alt="스크린샷 2023-06-10 오후 5 29 12" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/64fa5a74-75bf-473d-9025-de21e05f4751">

- `Producer` 가 아이템을 계속 생산해서 `Buffer` 가 가득차서 공간이 없는 상황이다.
- `Buffer` 가 비워지기를 기다리면서 `Producer` 가 계속 확인하는 문제가 있다.

### Consumer Problem

<img width="777" alt="스크린샷 2023-06-10 오후 5 31 29" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/aa391bbe-a848-4ef7-9fde-ec077dc6978f">

- `Consumer` 가 `Buffer` 에 아이템이 하나도 없는 상황이다.
- `Consumer` 는 `Buffer` 에 아이템이 있는지 계속해서 확인해야 하는 문제가 있다.

### 문제를 해결할 방법?

<img width="1000" alt="스크린샷 2023-06-10 오후 5 35 12" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/a9ff3c1a-3a7a-4722-97cf-5e3bbd6f991b">

<img width="1000" alt="스크린샷 2023-06-10 오후 5 52 10" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/d7b3632c-6d12-455e-8a56-ea30ed31c529">

- `Monitor` 를 통해 해결할 수 있다.
- 위 코드의 `producer()`, `consumer()` 흐름을 정확히 이해할 것

## 자바에서 모니터란?

- `Monitor` 라는 것이 사실 프로그래밍 언어 레벨에서 지원되는 것이다.
- 따라서 `Monitor` 를 직접 구현하는 일은 잘 없다.
- 프로그래밍 언어에서 지원하는 `Monitor` 를 사용만 하면 되는 것이다.

### 자바 모니터

- 자바에서 모든 객체는 내부적으로 `Monitor` 를 하나씩를 가지고 있다.
- 자바 `Monitor` 의 `상호 배제(Mutual Exclusion)` 기능은 `synchronized` 키워드로 사용한다.
- 자바 `Monitor` 는 `Condition Variable` 를 하나만 갖는다.

### 자바 모니터의 세 가지 동작

- `Wait`
- `Notify` (`Signal` 과 동일)
- `NotifyAll` (`BroadCast` 와 동일)

## Bounded Producer, Consumer Problem With Java

<img width="1000" alt="스크린샷 2023-06-10 오후 6 05 00" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/7786712f-fe31-440e-963e-555b6f1fd4c0">

- `synchronized` 키워드를 통해 `상호 배제(Mutual Exclusion)` 가 보장된다.
- `wait()`, `notifyAll()` 메소드를 통해 `Condition Variable` 의 `Wating Queue` 로 스레드를 보내거나, `Waiting Queue` 에 있는 스레드를 깨운다.
- 자바 `Monitor` 를 사용할 때, 두 가지 이상의 `Condition Variable` 이 필요하다면, 따로 구현이 필요하다.
- `java.util.concurrent` 패키지에는 동기화 기능이 탑재된 여러 클래스들이 있으니 참고해서 사용하면 된다.

## 참고

- [모니터(Monitor) 설명 영상 보기](https://www.youtube.com/watch?v=Dms1oBmRAlo&list=PLcXyemr8ZeoQOtSUjwaer0VMJSMfa-9G-&index=6)
