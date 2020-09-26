# JPA 조회 쿼리 최적화

예시를 들어서 일단 간단하게 요약하고, 권장 순서를 정리하려고 한다.

<br>

## 조회 쿼리 최적화

만약 `Order`, `Member`, `Delivery`, `OrderItem`, `Item`을 모두 가져와하는 상황이라면?

```java
EntityManager entityManager;

em.createQuery("select distinct o from Order o" +
              " join fetch o.member m" + // Order에서 ManyToOne
              " join fetch o.delivery d" + // Order에서 OneToOne
              " join fetch o.orderItems oi" + // Order에서 OneToMany
              " join fetch oi.item i",  // OrderItem에서 ManyToOne
              Order.class)
  .getResultList();
```

위와 같이 `Fetch Join`을 사용해서 1번 쿼리에 모두 가져오면 된다. 하지만, 페이징 처리를 해야한다면, 위의 방법을 사용하지 못한다.

<br>

### 페이징 처리를 사용하지 못하는 이유

`OneToMany`와 같은 `ToMany` 관계에서 `Fetch Join`을 하게되면, 기본적으로 데이터가 부풀려진다.

> Order에는 데이터가 2개

| orderId |      orderDate      | status |
| :-----: | :-----------------: | :----: |
|    1    | 2020-09-26 11:00:00 | ORDER  |
|    2    | 2020-09-26 11:00:00 | ORDER  |

> OrderItem에는 Order당 데이터가 2개

| orderItemId | orderId | orderPrice | count |
| :---------: | :-----: | :--------: | :---: |
|      1      |    1    |   10000    |   1   |
|      2      |    1    |   20000    |   2   |
|      3      |    2    |   20000    |   3   |
|      4      |    2    |   40000    |   4   |

> Order기준으로 OrderItem을 Join한다면?

| orderId |      orderDate      | status | orderItemId | orderPrice | count |
| :-----: | :-----------------: | :----: | :---------: | :--------: | :---: |
|    1    | 2020-09-26 11:00:00 | ORDER  |      1      |   10000    |   1   |
|    1    | 2020-09-26 11:00:00 | ORDER  |      2      |   20000    |   2   |
|    2    | 2020-09-26 11:00:00 | ORDER  |      3      |   20000    |   3   |
|    2    | 2020-09-26 11:00:00 | ORDER  |      4      |   20000    |   4   |

위와 같은 상황이 발생하고, JPA에서는 Database 결과들을 가져오기 때문에 총 4개를 가져온다. 사용자는 Order의 데이터 2개를 원했는데, 4개가 조회된 셈이다.

<br>

### JPA가 이렇게 동작하는 이유

JPA에서는 실제로 사용자가 이렇게 4개 데이터를 사용할 수도 있고, 아니면 중복된 데이터를 제거한 2개를 사용하고 싶을수도 있기 때문에 다음과 같이 동작한다.

<br>

### 페이징 처리를 해야 한다면 어떻게 해야할까

페이징 처리를 하려면, 쿼리를 분리해야한다.

1. `ToOne`의 관계에서는 `Fetch Join`을 사용해도 데이터가 부풀려지는 현상이 발생되지 않기 때문에 모두 `Fetch Join`을 적용한다.

2. `ToMany`와 같은 `Collection Fetch Join`에서는 지연로딩을 이용하지만, `Hibernate default_batch_fetch_size` 또는 `@BatchSize` 옵션을 통해 `In`절 쿼리를 발생시켜, 쿼리를 최적화한다.

```java
EntityManager entityManager;

em.createQuery("select distinct o from Order o" +
              " join fetch o.member m" + // Order에서 ManyToOne
              " join fetch o.delivery d", // Order에서 OneToOne
              Order.class)
  .getResultList();

// OrderItem, Item은 default_batch_fetch_size 옵션을 사용한 지연로딩을 통해 조회
```

<br>

## 권장 순서

강의를 진행주시는 김영한님께서는 다음과 같은 쿼리 최적화 순서를 권장하신다.

1. 엔티티 조회 방식으로 우선 접근

   1. `ToOne`의 관계들은 모두 `Fetch Join`을 적용해서 쿼리 수 최적화

   2. `ToMany`인 `Collection Fetch Join`의 경우

      2-1. 페이징이 필요하다면? `Hibernate default_batch_fetch_size` 또는 `@BatchSize` 옵션을 적용

      2-2. 페이징이 필요없다면? 판단에 의해서 그냥 `Fetch Join` 사용

2. 엔티티 조회 방식으로 해결이 안되면, DTO 조회 방식

3. 그래도 안된다면, Native SQL 또는 Spring JDBC Template 사용

<br>

## 참고

- [[인프런] 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94)
