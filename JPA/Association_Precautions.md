# 연관 관계에서 주의 사항

<br>

## 양방향 매핑시, 많이 하는 실수

### 예시

연관 관계 주인에 값을 입력하지 않는 경우가 있다.

```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "team")
    private Team team;

    @Column(name = "username")
    private String name;
}
```

```java
@Entity
public class Team {
    @Id @GeneratedValue
    @Column(name = "team_id")
    private Long id;

    @OneToMany(mappedBy = "team")
    private List<Member> members = ArrayList<>();

    @Column(name = "team_name")
    private String name;
};
```

```java
public class Main {
    public static void main(String[] args) {
        Team team = new Team();
        team.setName('team');
        em.persist(Team);

        Member member = new Member();
        member.setName('member');

        // 역방향(주인이 아닌 방향)만 연관 관계 설정
        team.getMembers().add(member);

        em.persist(member);
    }
}
```

| ID  | USERNAME | TEAM_ID |
| :-: | :------: | :-----: |
|  1  |  member  |  null   |

위의 코드를 실행해보면, member의 team_id가 데이터에 저장되지 않는다. 왜냐하면, JPA에서는 연관 관계 주인이 아닌곳에서 값을 설정할 때, 그 내용을 반영하지 않는다.
( `연관 관계 주인이 아닌 곳 = mappedBy로 설정된 부분` )

```java
public class Main {
    public static void main(String[] args) {
        Team team = new Team();
        team.setName('team');
        em.persist(Team);

        Member member = new Member();
        member.setName('member');

        // 순수한 객체 관계를 고려하면, 항상 양쪽 모두 값을 입력해야 한다.
        team.getMembers().add(member);
        // 연관 관계의 주인인 member에서 team의 값을 입력해야 한다.
        member.setTeam(team);

        em.persist(member);
    }
}
```

| ID  | USERNAME | TEAM_ID |
| :-: | :------: | :-----: |
|  1  |  member  |    1    |

위의 코드와 같이 `연관 관계의 주인인 member에서 team의 값을 입력해야 한다.` 그리고 `순수한 객체 관계를 고려하면, 항상 양쪽 모두 값을 입력해야 한다.`

### 정리

- 순수 객체 상태를 고려하여 항상 양쪽에 값을 설정하자

- 양방향 매핑시 참조 순환을 조심하자

- 연관 관계 편의 메소드를 생성하자

- 연관 관계 주인은 정해져 있지만 연관 관계 편의 메소드를 작성하는 것은 기준에 따라 다르다. (한 곳으로 정해야한다. Member에 작성할지? Team에 작성할지?)

### 1. Member에 연관 관계 메소드를 정하는 경우

```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "team")
    private Team team;

    @Column(name = "username")
    private String name;

    // 연관 관계 편의 메소드 - Member를 기준으로 했다면
    public void changeTeam(final Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}

public class Main {
    public static void main(String[] args) {
        Team team = new Team();
        team.setName('team');
        em.persist(Team);

        Member member = new Member();
        member.setName('member');
        member.changeTeam(team);

        em.persist(member);
    }
}
```

### Team에 연관 관계 메소드를 정하는 경우

```java
@Entity
public class Team {
    @Id @GeneratedValue
    @Column(name = "team_id")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members;

    public addMembers(final Member member) {
        this.members.add(member);
        member.setTeam(this);
    }
}

public class Main {
    public static void main(String[] args) {
        Member member = new Member();
        member.setName('member');
        em.persist(member);

        Team team = new Team();
        team.setName('team');
        team.addMembers(member);
        em.persist(Team);
    }
}
```

<br>

## 참고

- [인프런 - 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
