# Java 스레드 상태 종류

## 6가지의 상태가 존재한다.

- `NEW`
- `RUNNABLE`
- `BLOCKED`
- `WAITING`
- `TIMED_WAITING`
- `TERMINATED`

## NEW

- 자바 스레드가 아직 시작하지 않은 상태이다.

## TERMINATED

- 실행을 마치고, 종료된 상태이다.

## WAITING

- 어떤 스레드가 다른 스레드를 기다리는 상태를 의미한다.
- `Object.wait()` (모니터)
- `Thread.join()`

## TIMED_WAITING

- `WAITING` 이랑 똑같은데, 제한 시간을 두고 다른 스레드를 기다리는 상태이다.
- 정해진 시간만큼만 다른 스레드에 일어난 이벤트를 기다리겠다.
- `Object.wait(timeout)`
- `Thread.join(timeout)`
- `Thread.sleep(time)`

## BLOCKED

- 모니터 락을 얻기 위해 기다리는 상태이다.
- `임계 영역(Critical Section)` 으로 들어가려고 모니터 락을 얻기 위해 기다리는 상태이다.

## RUNNABLE

- Java 스레드가 실행중인 상태를 의미한다.
- `CPU` 상에서 실행하는 그 실행 상태를 포함하는 개념이다.
- 다른 리소스를 기다리는 상태도 포함한다.
    - `CPU` 에서 실행되려고 기다리는 상태
    - `I/O` 작업을 요청하고, `I/O` 작업의 결과를 기다리고 있는 상태
- 운영체제의 `Running` 상태보다 훨씬 더 포괄적인 의미를 갖는다.

## Thread Dump

- 실행중인 Java 프로세스의 현재 상태를 담은 스냅샷
- 현업에서 Java 언어로 개발된 API 서버를 운영하고 있는데, API 서버의 스레드를 거의 다 사용해버리고, 새로운 API 요청에 대해서 응답도 발생하지 못하는 상황이 발생하면, 해당 서버로
  접근해서 `Thread Dump` 를 가져온다. 그리고 분석을 통해 어떤 상태에서 문제가 발생했는지를 알 수 있다.
  - ex) 데드락이 발생했다 -> `BLOCKED`, `WAITING` 상태 (예시일뿐)
- 어디가 병목이어서 스레드가 다 차버렸는지를 알 수 있다.

## 참고

- [Java 스레드 상태 설명 영상 보기](https://www.youtube.com/watch?v=_dzRW48NB9M&list=PLcXyemr8ZeoQOtSUjwaer0VMJSMfa-9G-&index=8)
