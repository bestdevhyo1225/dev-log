# 리눅스 성능 분석 시작하기 - netstat 명령어

## netstat

```shell
$ sudo netstat -napo
```

- 네트워크 연결 정보를 확인할 수 있다.
- `Active Internet Connections` 영역은 `네트워크 소켓` 정보를 나타낸다.
- `Active Unix domain sockets` 영역은 `유닉스 소켓` 정보를 나타낸다.

## 어떻게 해석할 수 있을까?

```shell
$ sudo netstat -napo | grep -i est
```

- `로컬 IP`, `로컬 Port`, `원격 IP`, `원격 Port` 및 `소켓 상태` 를 확인할 수 있다.

## 가장 자주보는 소켓 상태

- `LISTEN`
    - 통신이 이뤄지려면 누군가 소켓으로 듣고 있어야한다.
- `ESTABLISHED`
    - 연결된 상태를 의미한다.
- `TIME_WAIT`
    - `TCP 4-Way Handshake` 에서 가장 마지막 단계를 의미한다.

## keepalive_timeout

- HTTP 요청에 대해 커넥션이 새로 맺지 않아도 되고, `keepalive_timeout` 만큼 새로운 요청을 기다린다.

## keepalive_timeout이 지나면?

- 더 이상 연결을 유지할 필요가 없다고 생각하고, 서버가 먼저 연결을 끊는다.
- 위와 같은 처리로 인해 `TIME_WAIT` 상태의 소켓이 생성된다.
    - 많이 생기는 이유는 무엇이며, 어떻게 하면 줄일 수 있을지를 고민해야 한다.
    - `서버에서 먼저 연결을 끊고 있는 것은 아닐까?` 를 살펴보고, 이러한 방향으로 튜닝을 해야한다.

## 보면 곤란한 상태는 무엇인가?

- 애플리케이션 관점에서 보면 `CLOSE_WAIT` 상태가 많으면, 문제가 있는 상황이다.
- `TCP 4-Way Handshake` 과정에서 연결을 해제할 때, `클라이언트의 ESTABLISHED` 에서 `서버의 CLOSE_WAIT` 간에는 `FIN` 패킷이 전송된다. `서버의 CLOSE_WAIT`
  에서 `클라이언트의 FIN_WAIT2` 간에는 `ACK` 패킷이 전송되며, 마지막으로 `서버의 LAST_WAIT` 에서 `클라이언트의 TIME_WAIT` 도 마찬가지로 `ACK` 패킷이 전송된다. 즉, 패킷을
  주고 받으며 소켓 상태를 변경한다.
- 그런데 `서버의 CLOSE_WAIT` 에서 `서버의 LAST_WAIT` 간에는 패킷이나 조건 없이 자연스럽게 상태가 넘어간다.

## 그래서 CLOSE_WAIT는 애플리케이션 이상 동작을 의미한다.

- 정상적으로 소켓을 정리하는 등 연결을 끊기 위한 동작을 하지 못한다는 의미이다.

## CLOSE_WAIT는 꼭 해결해야 하는 문제이다.

- `CLOSE_WAIT` 는 정리되지 않으며, 계속 쌓인다.
- 만약에 정상적인 상황이라면, `CLOSE_WAIT` 에서 `LAST_WAIT` 상태로 넘어가고, 없어져야 한다.
- 서버 행업, 포트 고갈 등 서비스에 영향을 끼치는 문제를 일으킬 수 있기 때문에 꼭 원인을 확인하고 해결해야 한다.

## 정리

- `netstat` 명령은 커넥션의 상태와 종단 간 IP 정보 등 서버의 네트워크 연결 정보를 확인할 수 있다.
- `LISTEN`, `ESTABLISHED`, `TIME_WAIT` 는 흔히 만나게 되는 소켓 상태이다.
    - 위의 3가지 상태가 많다고 문제가 생기는 것은 아니다.
    - `TIME_WAIT` 의 경우에은 왜 많은지 고민할 필요는 있다.
- `CLOSE_WAIT` 가 발생한다면, 꼭 원인을 확인하고 조치해야 한다.

## 참고

- [인프런 - 리눅스 성능 분석 시작하기](https://www.inflearn.com/course/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%84%B1%EB%8A%A5-%EB%B6%84%EC%84%9D-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0/dashboard)
