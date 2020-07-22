# Network ACL & Security Group 개념 정리

<br>

## Network ACL

- 외부 간 통신을 담당하는 보안 기능이다.

- `서브넷` 단위로 정책을 설정한다.

- `Stateless` 필터링 방식이다.

  - `Stateless` : 요청 정보를 따로 저장하지 않는다. 따라서 응답하는 트래픽에 대한 필터링을 설정해야 한다.

- 인바운드 / 아웃바운드 정책에 `1024 ~ 65535` 포트를 허용해야 통신이 가능하다.

<br>

## Security Group

- 내부 간 통신을 담당한다.

- `서버` 단위로 정책을 설정할 수 있다.

- `Stateful` 필터링 방식이다.

  - `Stateful` : 요청 정보를 저장하여 응답하는 트래픽을 제어하지 않는다.

<br>

## 외부에서 내부로 들어오는 경우

정리할 것

<br>

## 내부에서 외부로 들어오는 경우

정리할 것

<br>

## 결론

- 서브넷이 다르다면? `Network ACL` 정책이 적용된다.

- 서브넷이 같다면? `Security Group` 정책이 적용된다.

<br>

## 참고

- [https://cleverdj.tistory.com/122](https://cleverdj.tistory.com/122)
