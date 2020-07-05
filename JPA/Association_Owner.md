# JPA 연관 관계 주인

<br>

엔티티간의 연관 관계가 존재할 때, 양방향인 경우 연관 관계의 주인을 정해야 한다.

## 연관 관계 주인은 누가?

외래키를 가지고 있는 엔티티가 연관 관계의 주인이다.

- 회원과 주문간의 관계는 1(회원) : N(주문) 이다.

```java
@Entity
public class Order {

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

}
```

- 연관 관계의 주인이 아닌 값은 `mappedBy`를 추가해서 단순 조회용으로 사용한다.

```java
@Entity
public class Member {

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();

}
```
