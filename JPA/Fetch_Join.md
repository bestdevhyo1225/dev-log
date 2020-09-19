# JPQL & QueryDsl Fetch Join

<br>

## Fetch Join 이란?

- SQL 조인 종류가 아님

- JPQL에서 `성능 최적화`를 위해 제공하는 기능

- 연관된 엔티티나 컬렉션을 SQL 한 번에 조회하는 기능

<br>

## 엔티티 Fetch Join

Join 한 번에 `Member`, `Team`을 함께 조회한다.

`JPQL`

```java
SELECT m FROM Member m JOIN FETCH m.team
```

`SQL`

```sql
SELECT m.*, t.* FROM member m
INNER JOIN team t ON m.team_id = t.id
```

```java
// 쿼리 실행...
String jpql = "SELECT m FROM Member m JOIN FETCH m.team";
List<Member> members = em.createQuery(jpql, Member.class)
                          .getResultList();

for (Member member : members) {
  // Fetch Join으로 Member와 Team을 함께 조회 -> LAZY 로딩이 발생하지 않음
  String userName = member.getUsername();
  String teamName = member.getTeam().getName();
}
```

<br>

## 컬렉션 Fetch Join

`@OneToMany` 와 같은 `1 : N` 관계의 Fetch Join

`JPQL`

```java
SELECT t FROM Team t JOIN FETCH t.members WHERE t.name = '팀A'
```

`SQL`

```sql
SELECT t.*, m.* FROM team t
INNER JOIN member m ON t.id = m.team_id
WHERE t.name = '팀A'
```

```java
// 쿼리 실행...
String jpql = "SELECT t FROM Team t JOIN FETCH t.members WHERE t.name = '팀A'";
List<Team> teams = em.createQuery(jpql, Team.class)
                          .getResultList();

for (Team team : teams) {
  String teamName = team.getName();
  for (Member member : team.getMembers()) {
    // Fetch Join으로 Team과 Member를 함께 조회 -> LAZY 로딩이 발생하지 않음
    String userName = member.getUserName();
  }
}
```

하지만, 문제가 존재함. Database에서 `1 : N` 관계에서 조인을 사용하면, Data가 부풀려지는 현상이 발생함 이를 해결하기 위해서는 `DISTINCT` 키워드를 사용해야 함

- SQL에서 `DISTINCT`는 중복된 결과를 제거하는 명령어

- JPQL에서 `DISTINCT`는 2가지 기능을 제공

  1. SQL에 `DISTINCT`를 추가

  2. Application에서 엔티티 중복을 제거

즉, 아래와 같은 JPQL을 작성해서 `1 : N` Fetch Join을 사용하면 됨

```java
SELECT DISTINCT t FROM Team t JOIN FETCH t.members WHERE t.name = '팀A'
```

<br>

## 참고

- [인프런 - 자바 ORM 표준 JPA 프로그래밍 (기본편)](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
