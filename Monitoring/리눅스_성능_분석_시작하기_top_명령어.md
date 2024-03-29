# 리눅스 성능 분석 시작하기 - top 명령어

## top

```shell
$ top
```

- 프로세스들의 `상태`, `CPU`, `메모리` 사용률을 확인할 수 있다.
- `Tasks` 항목을 통해 프로세스 개수를 알 수 있다.
- `S(상태)`, `%CPU(CPU 사용률)` 컬럼이 중요하다.

## Hot key 사용법

- 키패드 숫자 `1` 을 누르면, `%CPU(s)` 가 `각각의 CPU` 로 변한다.
- 영문자 `d` 를 누르면, 인터벌을 변경할 수 있다.
    - 기본 값은 3초이다.

## top 명령을 통해 알 수 있는 것들

- `CPU` 사용량을 알 수 있다.
- `CPU` 사용량 중에서도 `us`, `sy`, `ni`, `id`, `wa`, `hi`, `si`, `st` 항목이 있는데, `us`, `wa` 항목을 눈여겨 봐야하고, 중요하다.
- `us` 는 `user` 를 의미하며, 프로세스의 `일반적인 CPU` 사용량이다.
- `wa` 는 `wating` 을 의미하며, `I/O 작업을 대기할때의 CPU` 사용량이다.

## us가 높다?

- `CPU` 를 많이 쓰고 있기 때문에 더 좋은 `CPU` 를 가진 서버로 변경해야 한다.

## wa가 높다?

- `I/O` 가 많기 때문에 더 좋은 블록 디바이스를 가진 서버로 변경해야 한다.
    - 예시) EC2에서 `gp2` EBS를 `gp3` EBS로 변경한다.

## CPU가 고르게 사용되고 있는지?

- `%CPU(s)` 가 `50%` 이여도 `CPU0` 가 `0%` 이고, `CPU1` 가 `100%` 인 상황이 있으면, `CPU1` 만 높기 때문에 정상이 아니다.

## 프로세스 상태

- `D` : uninterruptible sleep
    - `I/O` 대기 상태를 의미한다.
    - `Load Average` 에 포함된다.
- `R` : running
    - `CPU` 사용중임을 의미한다.
    - `Load Average` 에 포함된다.
- `S` : sleeping
    - 특별한 작업을 하지 않고, 자고 있는 상태를 의미한다.
- `Z` : zombie
    - CPU를 사용하거나, 메모리를 사용하고 있지 않고 그냥 남아있는 상태이다.
    - 특별히 시스템의 해를 끼치지 않지만, 이슈를 일으킬 가능성이 있다.

## 프로세스 생명주기

<img width="777" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/82cbed4b-4c3c-40e8-b353-b4d097cb0a54">

1. `fork()` 시스템 콜을 통해서 프로세스를 생성한다.
2. 프로세스는 `W(wating)` 상태가 되고, `CPU` 를 할당받기 위해서 대기한다.
3. 프로세스가 `CPU` 를 할당받고 작업을 하게되면, `R(running)` 상태가 되고 작업을 수행한다.
4. `I/O` 요청이 발생했다면, `D(uninterruptible sleep)` 상태가 된다.
    - `S(sleeping)` 상태가 되면, 5,6번 작업을 동일하게 수행한다.
5. `I/O Waiting` 상태가 완료되면, 다시 `W(wating)` 상태로 넘어간다.
6. 프로세스는 `CPU` 를 할당받고 다시 작업을 수행하다가 할당받은 `CPU Time` 을 다 썼으면, 다시 `W(wating)` 상태가 되며, `CPU` 를 할당받기 위해서 대기한다.

## 좀비 프로세스

- `부모` 프로세스가 종료 되었는데도 살아있는 `자식` 프로세스를 의미한다.

## 좀비 프로세스가 생기지 않는 일반적인 상황

<img width="500" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/89f1897b-772a-4874-b815-d636a78ce101">

- 위의 경우에는 `부모` 프로세스가 `자식` 프로세스가 종료될 때까지 대기하고, 그 이후에 `부모` 프로세스를 종료하기 때문에 문제가 없다.

## 좀비 프로세스가 생기는 상황

<img width="500" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/97e75db7-3679-477e-886c-df0a802b7da0">

- 커널에서는 일반적인 상황이면 `부모` 프로세스가 종료될 때, `자식` 프로세스도 강제로 종료시킨다.
- 그런데 `자식` 프로세스가 버그와 여러가지 조건에 걸려서 `자식` 프로세스가 남아있는 경우가 발생한다.
- 시스템 리소스를 사용하지 않지만, `PID 고갈을 일으킬 수 있다.`
- 참고로 `sudo sysctl -a | grep -i pid_max` 명령을 입력하면, `kernel.pid_max` 값을 확인할 수 있다.
- `kernel.pid_max` 값은 `동시에 존재할 수 있는 프로세스의 개수` 를 의미한다.
- 즉, `kernel.pid_max` 값이 `32523` 개인 상황에서 좀비 프로세스가 `25,000` 개이면 문제가 있는 상황이다.

## 정리

- `top` 명령을 이용해서 `프로세스의 상태`, `CPU`, `메모리` 사용량을 알 수 있다.
- `CPU` 사용량 중 `us` 가 많다면 `CPU` 를 많이 사용하는 워크로드를 의미하고, `wa` 가 높다면 `I/O` 를 많이 사용하는 워크로드를 의미한다.
- 멀티코어 환경이라면 `모든 CPU` 를 사용하고 있는지 확인해야 한다.
    - `%CPU(s)` 사용률 보다도 `각각의 CPU` 사용률이 중요하다.
- 프로세스 상태에는 `D`, `R`, `S`, `Z` 상태가 있다.

## 참고

- [인프런 - 리눅스 성능 분석 시작하기](https://www.inflearn.com/course/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%84%B1%EB%8A%A5-%EB%B6%84%EC%84%9D-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0/dashboard)
