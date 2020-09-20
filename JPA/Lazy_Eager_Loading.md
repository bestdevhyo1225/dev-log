# 지연 로딩과 즉시 로딩

<br>

## 지연 로딩 (Lazy Loading)

간단하게 요약하자면, 객체가 실제 사용될 때 로딩하는 방식이다.

- JPA에서 `@OneToMany`, `@ManyToMany`는 기본적으로 `Lazy Loading`으로 설정되어 있다.

```java
Member member = memberRepository.find(memberId); // SELECT * FROM MEMBER 쿼리가 날라감

Team team = member.getTeam();

String teamName = team.getName(); // SELECT * FROM TEAM 쿼리가 날라감
```

- **지연 로딩의 장점**

  - Member의 값만 주로 사용하고, Team값을 주로 사용하지 않는 경우라면, 지연 로딩을 사용하는 것이 성능적으로 유리하다.

- **지연 로딩의 단점**

  - 쿼리가 많이 나뉘고, 네트워크 비용이 자주 든다는 것

<br>

## 즉시 로딩 (Eager Loading)

실제로 서비스를 운영하다가 Member를 가져올 때, 90% 이상으로 Team도 같이 가져온다면, 즉시 로딩을 고려해보는 것이 좋다.

- JPA에서 `@ManyToOne`, `@OneToOne`은 기본적으로 `Eager Loading`으로 설정되어 있기 때문에 각별히 주의해야 한다. (`Lazy Loading`으로 반드시 설정할 것)

```java
Member member = memberRepository.find(memberId); // SELECT M.*, T.* FROM MEMBER JOIN TEAM ....

Team team = member.getTeam();

String teamName = team.getName();
```

- **즉시 로딩의 장점**

  - 지연 로딩과 반대되는 개념으로, 이미 로딩되어 있는 데이터를 활용하기 때문에 네트워크 비용이 줄어든다.

- **즉시 로딩의 단점**

  - 필요하지 않는 데이터가 포함 되어있는 경우, 사용하면 안되고 성능적으로 좋지 않다.

  - 즉시 로딩으로 설정된 테이블이 10개라면, 10개 테이블을 조인하기 때문에 엄청난 양의 쿼리가 실행된다. 즉, 비효율적이라고 할 수 있다.

  - 즉시 로딩 JPQL에서는 `N+1` 문제가 있다.

<br>

## JPQL에서 즉시 로딩의 N+1 문제점

```java
// Member 와 Team 은 1 : N 관계
Team team1 = new Team();
team1.setName("A");
em.persist(team1);

Team team2 = new Team();
team2.setName("B");
em.persist(team2);

Member member1 = new Member();
member1.setName("hyoseok1");
member1.setTeam(team1)
em.persist(member1);

Member member2 = new Member();
member2.setName("hyoseok2");
member2.setTeam(team2);
em.persist(member2);

List<Member> members = em.createQuery("select m from Member m", Member.class)
        .getResultList();

// 결과는?
```

- Member 리스트를 조회하는 1번의 쿼리와 2개의 Team을 조회하는 쿼리가 개별적으로 나간다. 팀이 N개 있다면, 1번의 쿼리와 N개의 쿼리가 나간다.

- 추가로, JPQL에서는 `LAZY`, `EAGER` 옵션과 상관없이 `N + 1` 문제를 발생시킨다. (이러한 문제 때문에 `Fetch Join`을 사용해서 해결함)

<br>

## Tip

강의를 해주시는 김영한님께서는 애플리케이션 개발 시점에 모두 부분을 `지연 로딩`으로 개발한 다음에 뭔가 최적화가 필요한 곳에 최적화를 하신다고 한다.

<br>

## 참고

- [인프런 - 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
