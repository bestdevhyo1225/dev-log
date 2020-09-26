# OSIV (Open Session In View)

- Open Session In View : Hibernate

- Open EntityManager In View : JPA

- 관례상 `OSIV` 라고 함

<br>

## OSIV란 뭔가요?

JPA가 언제 DB Connetion을 가져오고, 언제 Connection을 반환하는지? 여기서 부터 생각해봐야하는데 **가장 기본적으로 Transaction을 시작할 때, JPA 영속성 컨텍스트가 DB로 부터 Connection을 가져오고, OSIV 옵션 상태에 따라 DB Connection을 어디서 반환할지 결정한다.** 즉, OSIV는 어느 시점에 DB Connection을 반환해야 할지 도와주는 옵션이라고 생각하면 될 것 같다.

<br>

## OSIV ON

```yml
spring.jpa.open-in-view: true
```

default 값은 기본적으로 true이며, Transaction이 끝나고, Controller로 넘어가도 Connection을 반환하지 않는다. 즉, Transaction이 끝나도 DB Connection이 살아 있으며, 영속성 컨텍스트 또한 살아있다.

- API 경우, User에게 Response를 보낼때까지 살아있음

- View Template의 경우, 렌더링까지 살아있음

### 장점

지연로딩은 영속성 컨텍스트가 살아 있어야 가능하고, 영속성 컨텍스트는 기본적으로 DB Connection을 유지하고 있기 때문에, 어떤 계층에서든지 사용할 수 있기 때문에 개발하기가 용이하다.

### 단점

해당 전략은 오랜시간동안 DB Connection 리소스를 사용하기 때문에 실시간 트래픽이 중요한 애플리케이션에서는 Connection이 모자를 수 있고, 이러한 부분들이 장애로 이어질 수 있다.

<br>

## OSIV OFF

```yml
spring.jpa.open-in-view: false
```

Transaction이 시작한 후, 끝날때까지 영속성 컨텍스트가 유지된다. 즉, Transaction이 끝나면, 영속성 컨텍스트가 날라가고 DB Connection을 반환한다.

### 장점

DB Connection이 짧아서 리소스를 절약할 수 있다는 가장 큰 장점이 있다.

### 단점

OSIV가 false이면, 모든 지연로딩을 Transaction안에서 처리해야한다.

<br>

## 대안

### Command와 Query를 분리하자!!

`OrderService`

- OrderService : 핵심 비즈니스 Service

- OrderQueryService : 화면, API에 맞춘 Service (읽기 전용)

`OrderRepository`

- `OrderRepository`: 핵심 비즈니스 로직을 처리하는 Repository

- `OrderQueryRepository`: 화면, API에 맞춘 조회성 쿼리들을 포함한 Repositiory

<br>

## 참고

- [[인프런] 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94)
