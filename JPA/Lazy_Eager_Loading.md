# 지연 로딩과 즉시 로딩

<br>

## 지연 로딩 (Lazy Loading)

간단하게 요약하자면, 객체가 실제 사용될 때 로딩하는 방식이다.

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

```java
Member member = memberRepository.find(memberId); // SELECT M.*, T.* FROM MEMBER JOIN TEAM ....

Team team = member.getTeam();

String teamName = team.getName();
```

- **즉시 로딩의 장점**

  - 지연 로딩과 반대되는 개념으로, 이미 로딩되어 있는 데이터를 활용하기 때문에 네트워크 비용이 줄어든다.

- **즉시 로딩의 단점**

  - 필요하지 않는 데이터가 포함 되어있는 경우, 사용하면 안되고 성능적으로 좋지 않다.

<br>

## Tip

강의를 해주시는 김영한님께서는 애플리케이션 개발 시점에 모두 부분을 `지연 로딩`으로 개발한 다음에 뭔가 최적화가 필요한 곳에 최적화를 하신다고 한다.

<br>

## 참고

- [인프런 - 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
