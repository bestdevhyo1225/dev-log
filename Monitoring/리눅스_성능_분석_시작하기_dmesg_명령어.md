# 리눅스 성능 분석 시작하기 - dmesg 명령어

## dmesg

```shell
$ dmesg
```

- 커널에서 발생하는 다양한 메시지들을 출력한다.
    - `OOME` 발생 여부, `SYN Flooding` 여부
- `dmesg -T` 명령을 입력하면, 날짜 시간을 편리하게 볼 수 있다.

## 중점적으로 봐야하는 메시지

- `OOME(Out-of-Memory Error)`
- `SYN Flooding`

## OOME(Out-of-Memory Error)

- 가용한 메모리가 부족해서 더 이상 프로세스에게 할당해 줄 메모리가 없는 상황을 의미한다.
- `OOM Killer` 가 동작한다.
    - 누군가가 사용하고 있는 메모리를 회수한다. -> 종료시킨다.
- `OOM 상황 발생` -> `프로세스 선택(누구를 종료시켜야할 지?)` -> `프로세스 종료` -> `시스템 안정화(서비스가 안정화 되는 것은 아니다.)`

## 종료시킬 프로세스를 선택하는 기준

- `oom_score` 기준으로 선택한다.
    - `OOM Killer` 가 종료시킬 프로세스를 선택하는 기준
    - 스코어가 더 높은 프로세스가 더 먼저 종료된다.
- `proc` 디렉토리에서 `cat oom_score` 명령어로 스코어를 확인할 수 있다.
    - `proc` : 프로세스의 메타데이터를 볼 수 있다.

## OOM 발생 메시지를 더 자세히 보는 명령어

```shell
$ dmesg -TL | grep -i oom
```

## SYN Flooding

- 공격자가 대량의 `SYN` 패킷만 보내서 소켓을 고갈시키는 공격

## TCP 3-way Handshake

- 기본적으로 아래와 같이 동작한다.
    1. Client ------> `SYN` ------> Server
    2. Client <--- `SYN + ACK` <--- Server
    3. Client ------> `ACK` ------> Server
- 그리고
  커널에서는 `listen 함수` -> `SYN Backlog(SYN 패킷을 받았을때, SYN 패킷을 쌓는다.)` -> `Listen Backlog(ACK 패킷을 받았을때, ACK 패킷을 쌓는다.)` -> `accept 함수(애플리케이션에 통신 처리권을 넘겨준다.)`
  순으로 동작한다.

## SYN Flooding 공격이 발생하는 상황

- 클라이언트에서 서버로 `ACK` 패킷을 보내지 않는다면, `SYN Backlog` 에 있는 소켓 정보가 `Listen Backlog` 로 넘어가지 않고, `SYN Backlog` 에 계속 쌓인다.
- `Backlog` 는 크기가 제한되어 있는 `Queue` 이다.
- 즉, 꽉차면 `SYN` 패킷을 저장할 수 없고, `SYN` 패킷을 `Drop` 한다.
- 위와 같은 상황이 발생하면, 더 이상 새로운 연결 요청을 처리할 수 없다.

## SYN Cookie

- `SYN Flooding` 공격을 막기위한 방법이다.
- `SYN` 패킷의 정보를 바탕으로 쿠키를 만든다.
- 그리고 그 값을 `SYN + ACK` 의 시퀀스 번호로 만들어서 응답한다.

## SYN Cookie가 동작하는 TCP 3-way Handshake

- 기본적으로 아래와 같이 동작한다.
    1. Client --------------> `SYN` --------------> Server
    2. Client <------ `SYN + ACK (Cookie)` <------ Server
    3. Client -------> `ACK (Cookie + 1)` -------> Server
- 서버에서는 보내줬던 쿠키 값을 계산해서 확인한다.
    - `Timestamp`, `IP Address` 기반으로 계산한다.
- 위와 같이 처리하면, `SYN Backlog` 를 거치지 않고, `Listen Backlog` 를 통해서 애플리케이션에 패킷에 대한 제어권을 넘겨준다.

## SYN Cookie가 동작하게 되면?

- `SYN Backlog` 에 쌓이지 않는다. 그래서 자원 고갈이 발생하지 않는다.
- `TCP Option Header` 를 무시하기 때문에 `Windows Scaling` 등 성능 향상을 위한 옵션이 동작하지 않는다.
- `SYN Cookie` 가 동작하도록 `true` 와 같은 활성화 처리를 했더라도 바로 동작하지 않고, `SYN Flooding` 이 발생하는 것 같을때, `SYN Cookie` 가 동작을 시작한다.
    - 즉, 평상시에는 `SYN Cookie` 가 동작할 일은 없다.

## SYN Cookie와 관련해서 가치 판단을 해야한다.

- `SYN Flooding` 공격에 의해 서비스가 불통되는 것보다는 느려지는게 더 나은 선택이 될 수 있다.

## SYN Flooding 메시지를 더 자세히 보는 명령어

```shell
$ dmesg -TL | grep -i "syn flooding"
```

- 그런데 요즘에는 `AWS ALB`, `AWS WAF`, `AWS Shield` 같은 것들이 있기 때문에 잘 방어가 되는 편이다.
- 그래서 최근에는 이런 메시지를 많이 볼수는 없다.
    - 다른 환경이라면, 볼 수 있다.

## 정리

- `dmesg` 명령을 이용해서 `OOME`, `SYN Flooding` 공격이 발생하지 않는지 확인한다.
- `OOME` 가 발생한다면, 더 많은 메모리를 확보한다.
- `SYN Flooding` 공격이 발생하면, 방화벽을 확인한다.

## 참고

- [인프런 - 리눅스 성능 분석 시작하기](https://www.inflearn.com/course/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%84%B1%EB%8A%A5-%EB%B6%84%EC%84%9D-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0/dashboard)
