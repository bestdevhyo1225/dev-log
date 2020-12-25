# @MapsId

@MapsId에 대해서 간단히 정리하려고 한다.

<br>

## @MapsId란?

외래 키와 매핑한 연관관계를 기본 키에도 매핑하겠다는 뜻이다. 주로 `@OneToOne` 관계에서 사용한다. 다음 예제를 보자

> Book

```java
@Entity
public class Book {
  @Id
  @GeneratedValue
  private Long bookId;

  @OneToOne(mappedBy = "book", fetch = FetchType.LAZY)
  private BookDescription bookDescription;
}
```

> BookDescription

```java
@Entity
public class BookDescription {
  // @MapsId로 인해, Book 엔티티의 Id가 매핑된다.
  @Id
  private Long bookId;

  @MapsId
  @OneToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "book_id")
  private Book book;
}
```

> 위와 같이 2개의 엔티티가 `@OneToOne` 양방향 연관관계로 설정되어 있다.
> `BookDescription` 엔티티에서 `@MapsId`를 지정해두면, `@Id`는 `Book` 엔티티의 PK인 `bookId`로 매핑된다.

> 풀어서 설명하자면, `Book`, `BookDescription`의 라이프 사이클이 동일한 경우가 있다. 즉, `Book`, `BookDescription`이 저장되고, 수정되는 사이클이 같다면, `Book`을 애그리거트로 두고, `cascade = CascadeType.ALL` 옵션을 통해 `Book`, `BookDescription`을 한 번에 저장되도록 구현할 수 있다. 이 때 `BookDescription` 엔티티의 `@Id`는 `Book` 엔티티의 `@Id`와 동일한 값이 매핑된다.
