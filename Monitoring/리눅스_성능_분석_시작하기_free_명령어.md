# 리눅스 성능 분석 시작하기 - free 명령어

## free

```shell
$ free -m # -m 옵션을 추가하면, MB 단위로 볼 수 있다.
```

- 시스템의 메모리 정보를 출력한다.

## free와 available 차이점

- `free` 는 어느 누구도 사용하고 있지 않은 메모리를 의미한다.
- `available` 은 애플리케이션에게 실질적으로 할당 가능한 메모리를 의미한다.

## free와 available이 따로 있는 이유는 무엇일까?

- `free` 와 `available` 차이점을 이해하기 위해서는 `buff/cache` 영역을 이해해야 한다.
- `buff/cache` 에서 `buff` 는 블록 디바이스(디스크)가 가지고 있는 블록 자체에 대한 캐시를 의미한다.
- `buff/cache` 에서 `cache` 는 I/O 성능 향상을 위해 사용하는 `페이지 캐시` 를 의미한다.

## 페이지 캐시

<img width="1000" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/6c798ee9-5b0f-440c-8875-ef84bd7e190d">

- `open` 시스템 콜을 통해서 블록 디바이스 안에 있는 `test.txt` 파일을 읽는다.

<img width="1000" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/d3c7c3bf-9d3b-48db-893a-0f80f8ed9b31">

- 실제로 바로 읽는 것은 아니며, `페이지 캐시` 를 통해 읽게된다.
- 읽기 요청을 하게 되면, 블록 디바이스의 `test.txt` 파일을 `페이지 캐시` 에 저장하고, 애플리케이션은 `페이지 캐시` 에서 `test.txt` 파일을 읽는다.
    - `페이지 캐시` 는 메모리 `buff/cache` 영역중에 `cache` 영역이다.

## 만약에 애플리케이션 B가 test.txt 파일을 읽어 들이려고 한다면?

- `페이지 캐시` 가 없다면, 블록 디바이스에서 `test.txt` 파일을 읽는다.
- 블록 디바이스는 디스크이기 때문에 아무리 빨라도 메모리보다 빠를 수 없다.
- 그래서 `페이지 캐시` 가 있다면, 애플리케이션 B도 `페이지 캐시` 에서 `test.txt` 파일을 읽는다.

## 페이지 캐시를 통해 무엇을 얻을 수 있을까?

- I/O 성능 향상을 얻을 수 있다.
- 한 번 읽었던 파일은 `페이지 캐시` 에 저장함으로써 블록 디바이스가 아닌 메모리에서 파일의 내용을 가져온다.

## 다시 돌아가서 free와 available이 따로 존재하는 이유는 무엇일까?

- `buff/cache` 는 I/O 성능 향상을 위해 존재한다.
- 애플리케이션에서 메모리를 필요로 하는 상황이 발생하면, `buff/cache` 영역을 해제하고, 애플리케이션이 사용할 수 있는 영역으로 바꾼다.
- `OOME` 가 발생해서 다른 프로세스를 죽여서 메모리를 확보하는 것보다 I/O 성능이 떨어지더라도 메모리를 필요로 하는 애플리케이션에게 메모리를 주는 것이 맞다.

## 그래서 available은 무엇인가?

- 애플리케이션에 줄 수 있는 메모리 양을 의미한다.
- `available` = `free` + `buff/cache`
    - 위의 식이 완전히 성립되진 않지만, 유사하다.

## Swap 영역

- 메모리가 부족한 상황에서 사용되는 가상 메모리 공간이다.
- 주로 블록 디바이스의 일부 영역을 사용한다.
- `OOME` 가 발생하는 과정은 다음과 같다.
    - `buff/cache 해제` -> `Swap 영역 사용` -> `OOME 발생`

## Swap 영역에 할당되는 과정

<img width="1000" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/9a781b5c-e88f-445d-865e-67ac3357c90d">

- 프로세스 B가 커널을 통해 메모리 주소 0번을 요청한다.
- 메모리 주소 0번의 경우, 이미 프로세스 A의 메모리 주소 0번을 사용하고 있다.
- 프로세스 A에서 사용중인 메모리 주소 0번을 `Swap Out` 을 통해 `Swap` 영역에 보관한다.

<img width="1000" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/66d88a3b-fa0c-4516-8992-00b47fbbc59f">

- 메모리 주소 0번은 비어있는 상태가 된다.

<img width="1000" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/f51a879d-5426-4d77-87d0-7d434df8a926">

- 메모리 주소 0번에는 프로세스 B를 할당한다.

## 프로세스 A가 0번 메모리 영역의 데이터를 원한다면?

<img width="1000" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/421a61db-6432-49d2-8ac2-aa3973c53b25">

- 프로세스 A가 커널을 통해 메모리 주소 0번을 요청한다.
- 메모리 주소 0번의 경우, 이미 프로세스 B의 메모리 주소 0번을 사용하고 있다.
- 프로세스 B에서 사용중인 메모리 주소 0번을 `Swap Out` 을 통해 `Swap` 영역에 보관한다.

<img width="1000" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/09d560ce-bd6c-4fdf-b1eb-17fd393ee36e">

- 메모리 주소 0번은 비어있는 상태가 된다.

<img width="1000" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/4175becd-183c-4040-8d45-e9b277951547">

- `Swap In` 을 통해 프로세스 A의 메모리 주소 0번 데이터를 할당한다.
    - `Swap In` 이 되어도 `Swap` 영역에서 프로세스 A의 메모리 주소 0번 데이터는 삭제되지 않는다.
    - 이 영역을 `Swap Cached` 영역이라고 부른다.

## Swap 영역을 사용하게 되면

- 프로세스가 메모리 참조를 위해 블록 디바이스에 대한 I/O가 빈번하게 발생하기 때문에 성능 저하가 발생한다.
- `Swap` 영역은 메모리가 아니고, 블록 디바이스에 위치하기 때문에 블록 디바이스로 접근하고, 이로 인해 성능이 저하되는 것이다.
- 즉, `Swap In`, `Swap Out` 이 발생할 때마다 블록 디바이스에 대한 I/O가 발생한다.

## Swap 영역과 관련된 고민 포인트

- 성능이 떨어지더라도 `OOM Killer` 에 의해 죽는 것을 막을 것이냐?
- 성능이 떨어질 바에는 `OOM Killer` 에 의해 죽는 것이 나은 것이냐?

## 최근 트렌드는 Swap 영역을 비활성화

- `AWS EC2` 의 경우, `Swap` 영역이 디폴트로 없다.
- 컨테이너는 빠르게 다시 띄울 수 있기 때문에 쿠버네티스 환경에서는 대부분 `Swap` 영역을 비활성한다.

## 정리

- `free` 명령을 사용해서 시스템의 메모리 사용 현황을 볼 수 있다.
- `available` 은 실질적으로 프로세스에게 할당할 수 있는 메모리 양이다.
- `buff/cache` 가 높다면, I/O가 빈번하게 발생한다는 의미이다.
- `Swap` 영역을 사용한다는 것은 메모리가 부족하다는 신호이다.

## 참고

- [인프런 - 리눅스 성능 분석 시작하기](https://www.inflearn.com/course/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%84%B1%EB%8A%A5-%EB%B6%84%EC%84%9D-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0/dashboard)
