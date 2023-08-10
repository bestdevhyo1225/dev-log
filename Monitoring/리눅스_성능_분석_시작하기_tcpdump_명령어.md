# 리눅스 성능 분석 시작하기 - tcpdump 명령어

## tcpdump

```shell
$ tcpdump
```

- 네트워크 패킷 수집 및 분석을 할 수 있다.
- 네트워크 패킷의 흐름을 전반적으로 볼 수 있다.
- `tcpdump` 옵션을 아무 옵션 없이 입력하면, 해석하기 어렵다.

```shell
$ tcpdump -nn
```

- 프로토콜과 숫자번호를 그대로 입력한다.

```shell
$ tcpdump -vvv
```

- 출력 결과에 더 많은 정보를 담는다.

```shell
$ tcpdump -A
```

- 패킷의 내용도 함께 출력한다.

```shell
$ tcpdump -vvv -nn -A host x.x.x.x and port 80
```

<img width="700" alt="스크린샷 2023-08-10 오후 7 30 45" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/5a0bd20c-b738-4f77-b9c6-d5aaa11926e2">

<img width="700" alt="스크린샷 2023-08-10 오후 7 30 35" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/ecf804a3-22f7-429f-b212-c708041e352f">

- 트러블 슈팅은 목적지와 포트가 존재한다.
- `host` 와 `port` 옵션을 사용하면, 패킷을 필터링해서 분석할 수 있다.

## wireshark

- `tcpdump` 를 조금 더 편하게 분석할 수 있도록 도와주는 도구이다.
- `tcpdump` 를 통해 `pcap` 파일을 생성하고, `wireshark` 로 분석한다.

<img width="700" alt="스크린샷 2023-08-10 오후 7 37 21" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/bd216c68-6a22-40c6-acc3-89da255ce06e">

- `sftp` 명령을 이용해서 파일을 로컬로 가져와서 분석한다.

<img width="700" alt="스크린샷 2023-08-10 오후 7 37 35" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/89458aaa-ff3f-48b9-a468-bca460368e77">

- HTTP 요청의 전체 과정을 살펴볼 수 있다.
- 위의 지표에서 `ACK` 와 `FIN + ACK` 사이에는 `keepalive_timeout` 만큼 공백이 존재한다.

## 정리

- `tcpdump` 명령을 이용해서 네트워크 패킷을 수집하고, 분석할 수 있다.
- `-vvv`, `-nn`, `-A` 옵션을 이용해서 `tcpdump` 를 조금 더 효율적으로 사용할 수 있다.
- `host`, `port` 문구를 이용해서 특정 목적지, 특정 포트로 필터링을 할 수 있다.
- `tcpdump` 로 `pcap` 파일을 생성하고, `wireshark` 로 분석할 수 있다.

## 참고

- [인프런 - 리눅스 성능 분석 시작하기](https://www.inflearn.com/course/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%84%B1%EB%8A%A5-%EB%B6%84%EC%84%9D-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0/dashboard)
