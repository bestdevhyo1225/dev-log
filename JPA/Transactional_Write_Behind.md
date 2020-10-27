# 트랜잭션을 지원하는 쓰기 지연을 사용할 시, 주의할 점

트랜잭션을 지원하는 쓰기 지연을 테스트 해보려다가 꼭 기록을 해야할 것 같아서 남김 (공부했던 부분인데 까먹는 것을 방지하기 위해서)

<br>

## 쓰기 지연을 사용하려면?

- 우선 엔티티가 영속 상태가 되려면, 식별자가 필요하다.

- 식별자 생성 전략을 `IDENTITY`로 하면, 실제 데이터베이스에 저장을 해야 식별자를 구할 수 있다. 따라서 INSERT 쿼리가 즉시 데이터베이스에 전달된다.

- 위의 경우에는 쓰기 지연을 활용한 성능 최적화를 할 수 없다.

<br>

## 쓰기 지연 예제

`Entity`

```kotlin
@Entity
class Order(price: Int) {

    @Id
    @GeneratedValue // 식별자 생성 전략을 IDENTITY 로 하면, 트랜잭션을 지원하는 쓰기 지연을 사용할 수 없다.
    @Column(name = "order_id")
    val id: Long? = null

    @Column(nullable = false)
    var price: Int = price

}
```

`Test`

```kotlin
@DataJpaTest
@DisplayName("Order 엔티티 테스트")
internal class OrderTest(@Autowired private val entityManager: EntityManager) {

    @Test
    @DisplayName("트랜잭션을 지원하는 쓰기 지연")
    fun transactional_write_behind_test() {
        // Order 엔티티의 식별자 생성 전략은 디폴트 값인 Auto
        val orderA = Order(20000)
        val orderB = Order(30000)

        entityManager.persist(orderA)
        entityManager.persist(orderB)

        println("----- 커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다. -----")

        entityManager.flush() // 데이터베이스 동기화
        entityManager.clear() // 영속성 컨텍스트 초기화
    }

}
```

![스크린샷 2020-10-27 오후 4 54 57](https://user-images.githubusercontent.com/23515771/97272395-30914880-1875-11eb-9b05-3e697ce71f32.png)
