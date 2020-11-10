# 플러시(Flush)

> **플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다.**

1. 변경 감지가 동작해서 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교해서 수정된 엔티티를 찾는다. 수정된 엔티티는 수정 쿼리를 만들어 쓰기 지연 SQL 저장소에 등록한다.

2. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송한다.(등록, 수정, 삭제 쿼리)

<br>

> 영속성 컨텍스트를 플러시하는 방법은 3가지다.

1. entityManager.flush()를 직접 호출한다.

2. 트랜잭션 커밋시, 플러시가 자동으로 호출된다.

3. JPQL 쿼리 실행시, 플러시가 자동으로 호출된다.

<br>

## entityManager.flush() 직접 호출

- flush() 메소드를 직접 호출해서 영속성 컨텍스트를 강제로 플러시한다.

- 테스트나 다른 프레임워크와 JPA를 함께 사용할 때를 제외하고 거의 사용하지 않는다.

<br>

## 트랜잭션 커밋시, 플러시가 자동으로 호출

- 트랜잭션 커밋하기 전에 자동으로 플러시가 호출되고, 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영해준다.

<Br>

## JPQL 쿼리 실행시, 플러시가 자동으로 호출

```java
entityManger.perist(memberA);
entityManger.perist(memberB);
entityManger.perist(memberC);

// 중간에 JPQL 실행
query = entityManager.createQuery("select m from Member m", Member.class);

List<Member> members = query.getResultList();
```

- 우선 memberA, memberB, memberC는 영속성 컨텍스트에 존재한다.

- 해당 엔티티들은 영속성 컨텍스트에는 있지만, 데이터베이스에는 반영되지 않았다.

- **이 때 JPQL을 실행하면, JPQL은 SQL로 변환되어 데이터베이스에서 엔티티를 조회하는데, 데이터가 없으므로 조회되지 않는다.**

- JPA는 JPQL 쿼리를 실행하기 직전에 자동으로 플러시를 호출해서 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다.

  - 참고) `entityManger.find(Member.class, id)`와 같은 식별자 기준으로 조회할 때는 플러시가 실행되지 않는다.

<br>

## 헷갈리지 말기!

플러시라는 이름으로 인해 영속성 컨텍스트에 보관된 엔티티를 지운다고 생각하면 안된다. 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 것이 플러시이다.
