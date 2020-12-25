# @OneToOne 양방향 연관관계시 주의할 점

프로젝트를 진행하던 중, 문제가 없을거라고 생각하고 그냥 진행했던 `@OneToOne` 양방향 매핑에 대해서 정리하고자 한다.

<br>

## Lazy Loading 이슈

프로젝트를 진행하면서, 양방향 관계 모두 Fetch 전략을 `Lazy`로 설정했지만, `Eager`로 동작하는 문제가 발생했다.

> Book Entity

```java
@Entity
@Getter
@DynamicUpdate
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Book {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column(name = "book_id")
  private Long id;

  @OneToOne(mappedBy = "book", fetch = FetchType.LAZY, cascade = CascadeType.ALL)
  private BookDescription bookDescription;
}
```

> BookDescription Entity

```java
@Entity
@Getter
@DynamicUpdate
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class BookDescription {
  @Id
  private Long bookId;

  @OneToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "book_id")
  private Book book;
}
```

> Book을 가져오기 위한 테스트 코드

```java
@SpringBootTest
@Transactional
@DisplayName("BookRepository 테스트")
class BookRepositoryTest {
  @PersistenceContext
  private EntityManager entityManager;

  @Autowired
  private BookRepository repository;

  @BeforeEach
  void init() {
    String title = "JPA";
    String author = "author1";
    int price = 25000;
    String contents = "JPA 책이고, 반드시 공부 해야합니다.";

    BookDescription bookDescription = BookDescription.builder().contents(contents).build();
    Book book = Book.create(title, author, price, bookDescription, new ArrayList<>());

    entityManager.persist(book);
    entityManager.flush();
    entityManager.clear();
  }

  @Test
  void Book_Entity를_조회한다() {
    Long bookId = 1L;
    repository.findById(bookId)
            .orElseThrow(() -> new NoSuchElementException("존재하지 않음"));
  }
}
```

> Book의 정보만 가져와야 하고, BookDescription은 Lazy로 설정되어 있기 때문에 조회가 되면 안된다. 그런데 Eager로 동작하게 된다.

![image](https://user-images.githubusercontent.com/23515771/103120341-a588cf00-46ba-11eb-8db7-6ce9a43ce263.png)

<br>

## JPA에서 @OneToOne 연관관계 제약 사항

이러한 이슈를 해결하고자 여러 검색을 해보니 다음과 같은 제약 사항이 있었다.

- JPA에서는 `@OneToOne`관계에서 무조건적인 `Lazy Loading`을 지원하지 않는다.

- `특정 조건`을 모두 만족해야만 `Lazy Loading`이 작동한다. 그 외에는 객체를 참조하기 위한 Fetch 전략을 `Lazy`로 지정하더라도 `Eager`로 동작한다.

<br>

## @OneToOne은 무조건적인 Lazy Loading을 지원하지 않을까?

JPA에서는 객체 참조가 `프록시` 기반으로 동작하는데, 연관관계가 있는 객체는 참조를 할 때, 기본적으로 `Null`이 아닌 `프록시` 객체를 반환한다.

- `@OneToOne` 관계에서 `Null`이 허용되는 경우, `프록시` 형태로 `Null` 객체를 반환할 수 없기 때문이다. (그래서 `Eager`로 동작해서 객체를 채우는 듯...)

- `@OneToMany` 관계에서는 이미 배열의 형태로 참조할 `프록시` 객체를 랩핑하고 있기 때문에, 그 객체가 `Null`이라도 참조할때는 문제가 되지 않는다.

<br>

## JPA에서 @OneToOne 관계일 때, Lazy Loading 발동 조건은 무엇일까?

- nullable이 허용되지 않는 `@OneToOne` 관계여야 한다. 즉, 참조 객체가 `optional = false`로 지정할 수 있는 관계여야 한다.

- 양방향이 아닌 단방향 `@OneToOne` 관계여야 한다.

- `@PrimaryKeyJoin`은 허용되지 않는다. 부모와 자식 엔티티 간의 조인 컬럼이 모두 `Primary Key`인 경우를 의미한다.

<br>

## 위의 내용을 토대로 여러가지 테스트를 해보자 (양방향 관계)

> 연관관계 주인이 호출하게 되면, Lazy Loading이 제대로 동작한다.

- 연관관계 주인은 `외래 키를 가지고 있는 쪽`을 말한다.

```java
@Test
void BookDescriptio_Entity를_조회한다() {
  bookDescriptionRepository.findAll();
}
```

> Book을 조회하는 쿼리가 발생되지 않음. 즉, Eager로 동작하지 않고, Lazy가 제대로 동작하고 있다.

<img width="909" alt="스크린샷 2020-12-25 오후 2 38 50" src="https://user-images.githubusercontent.com/23515771/103121354-1e8a2580-46bf-11eb-85ea-0a62e1184992.png">

> 연관관계 주인이 아닌쪽에서 호출하게 되면, Lazy Loading이 동작하지 않는다. Lazy Loading으로 동작하게 하려면, 다음과 같이 코드를 수정해야 한다.

> Book Entity

- `@OneToOne`설정에서 `optional = false`로 설정을 해두면, Null이 아닌 관계가 항상 존재해야 한다는 의미로 해석된다.

```java
@Entity
@Getter
@DynamicUpdate
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Book {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column(name = "book_id")
  private Long id;

  // optional = false 추가
  @OneToOne(mappedBy = "book", fetch = FetchType.LAZY, optional = false, cascade = CascadeType.ALL)
  private BookDescription bookDescription;
}
```

> BookDescription Entity

- `@OneToOne` 관계로 설정되어 있는 부분에 `@MapsId`를 추가하게 되면, 외래 키와 매핑한 연관관계를 기본 키에도 매핑하겠다는 뜻이다.

```java
@Entity
@Getter
@DynamicUpdate
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class BookDescription {
  @Id
  private Long bookId;

  @MapsId // 추가
  @OneToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "book_id")
  private Book book;
}
```

> 기존에 작성했던 테스트 코드를 다시 실행하면, Lazy Loading이 정상적으로 작동되는 것을 확인할 수 있다.

<img width="871" alt="스크린샷 2020-12-25 오후 2 49 45" src="https://user-images.githubusercontent.com/23515771/103121692-a3297380-46c0-11eb-9e0c-e22292e7f612.png"><br><br>

## 정리

내가 참고한 블로그에는 다음과 같이 2가지 측면으로 문제를 해결하도록 권유하고 있다.

> 신규 서비스에 JPA를 적용하는 경우

- 필요한 경우가 아니면, `@OneToOne` 관계를 맺지 않도록 설계하자

- `@OneToOne` 관계가 꼭 필요하다면? 부모가 외래 키를 가지는 방식으로 설계하자 (여기서는 `Book` 엔티티가 외래 키를 가지는 방식으로 설계할 것)

- 조금이라도 `@OneToMany` 관계로 변경될 가능성이 있다면, 처음부터 `@OneToMany` 관계로 설계하자

> 레거시 시스템에 JPA를 적용하는 경우

- 부모가 외래 키를 가지는 방식으로 설계되어 있다면? 엔티티간 연관 관계를 `@OneToOne 단방향`으로 설정하면 되고, 이슈가 발생하지 않는다.

- 자식이 외래 키를 가지는 방식으로 설계되어 있다면? 부모쪽 `@OneToOne`에는 `optional = false` 처리를 반드시 해주고, 자식쪽 `@OneToOne`에는 `@MapsId`를 추가해서 문제를 해결해야 한다. (양방향 연관관계를 기준으로 해결 방법을 제시한 것임)

<br>

## 참고

- [JPA 도입 — OneToOne 관계에서의 LazyLoading 이슈 #1](https://medium.com/@yongkyu.jang/jpa-%EB%8F%84%EC%9E%85-onetoone-%EA%B4%80%EA%B3%84%EC%97%90%EC%84%9C%EC%9D%98-lazyloading-%EC%9D%B4%EC%8A%88-1-6d19edf5f4d3)
