# 스핀락(SpinLock), 뮤텍스(Mutex), 세마포어(Semaphore)

## 경쟁 조건 (Race Condition)

- `여러 프로세스/스레드` 가 동시에 같은 데이터를 조작할 때, 타이밍이나 접근 순서에 따라 결과가 달라질 수 있는 상황

## 동기화 (Synchronization)

- `여러 프로세스/스레드` 를 동시에 실행해도 `공유 데이터의 일관성을 유지하는 것`

## 임계 영역 (Critical Section)

- 공유 데이터의 일관성을 보장하기 위해 `하나의 프로세스/스레드` 만 진입해서 실행 가능한 영역 (`Mutual Exclusion`)

## Mutual Exclusion을 보장하는 방법?

- `잠금(Lock)` 을 사용하자

```
do {
    acquire lock
        critical section
    release lock 
        remainder section
} while(TRUE)
```

## 스핀락 (SpinLock)

- 락을 가질 수 있을 때까지 반복해서 시도한다.
- 락을 기다리는 동안 CPU를 낭비한다는 단점이 있다.

## 뮤텍스 (Mutex)

- 락을 가질 수 있을 때까지 휴식을 취하는 방식이다.

## 뮤텍스가 스핀락보다 항상 좋은걸까?

- 꼭 그런것은 아니다.
- `멀티 코어` 환경이고, `임계 영역(Critical Section)` 에서의 작업이 컨텍스트 스위칭보다 더 빨리 끝난다면, 스핀락이 뮤텍스보다 더 이점이 있다.

## 세마포어 (Semaphore)

- `Signal Mechanism` 을 가진, 하나 이상의 `프로세스/스레드` 가 `임계 영역(Critical Section)` 에 접근 가능하도록 하는 장치
- 세마포어는 순서를 정해줄 때 사용한다.
- `signal()`, `wait()` 는 같은 프로세스 내에서 실행될 필요없다. 세마포어가 순서를 정해주기 때문이다.

<img width="800" alt="스크린샷 2023-06-07 오후 9 25 18" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/181abf87-f00d-456b-afdd-eaabd83ac090">

## 뮤텍스(Mutex)와 이진 세마포어(Binary Semaphore)는 같은 것이 아닌가?

- 같은 것이 아니다.
- 뮤텍스는 락을 가진자만 락을 해제할 수 있지만, 세마포어는 그렇지 않다.
- 뮤텍스는 `Priority Inheritance` 속성을 가진다. 세마포어는 그 속성이 없다.
- 아래의 그림에서 `P2` 의 `우선순위` 를 `Low -> High (P1 우선순위 만큼)` 로 올려주는 것을 `Priority Inheritance` 라고 한다.
- 이에 따라 `P2` 는 `임계 영역(Critical Section)` 을 빨리 빠져나올 수 있다.
- 세마포어에서는 누가 락을 해제할 지 알 수 없기 때문에 `Priority Inheritance` 속성을 가지고 있지 않다.

<img width="800" alt="스크린샷 2023-06-07 오후 9 31 49" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/8bf3e8ee-6f17-40bd-9b99-1108e875ac18">

## 정리

- 상호 배제만 필요하다면, `뮤텍스(Mutex)` 를 권장한다.
- 작업 간의 실행 순서 동기화가 필요하다면, `세마포어(Semaphore)` 를 권장한다.
- 스핀락, 뮤텍스, 세마포어의 구체적인 동작 방식은 OS와 프로그래밍 언어에 따라 조금씩 다를 수 있으므로 관련 문서를 잘 읽어봐야 한다.

## 참고

- [스핀락(SpinLock), 뮤텍스(Mutex), 세마포어(Semaphore)](https://www.youtube.com/watch?v=gTkvX2Awj6g&list=PLcXyemr8ZeoQOtSUjwaer0VMJSMfa-9G-&index=6)
