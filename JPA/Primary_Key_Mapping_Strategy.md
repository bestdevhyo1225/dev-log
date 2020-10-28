# 기본키 매핑 전략 - 자동 생성

- IDENTITY

- SEQUENCE

- TABLE

- AUTO

<br>

## IDENTITY 전략

- 기본키 생성을 DB에 위임

- MySQL, PostgreSQL, SQL Server, DB2에서 사용 (예시 -> AUTO_INCREMENT)

- JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL을 실행

- AUTO_INCREMENT는 DB에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있다.

- **IDENTITY 전략은 entityManager.persist() 시점에 즉시 INSERT SQL을 실행하고, DB에서 식별자를 조회한다.**

```kotlin
@Entity
class Member {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column(name = "member_id")
  val id: Long? = null

}
```

ID가 NULL로 넘어오면, DB에서 그때 값을 세팅한다. 따라서 ID값을 알 수 있는 시점이 DB에 INSERT 되고 나서 알 수 있다. 그런데 영속성 컨텍스트에서 엔티티를 관리하려면, PK와 같은 식별자가 있어야 한다. **IDENTITY 전략에서는 예외적으로 entityManager.persist를 호출하면, 커밋전에 즉시 DB에 INSERT 쿼리를 보낸다.**

<br>

## SEQUENCE 전략

- Sequence는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트이다.

- Oracle, PostgreSQL, DB2, H2 데이터베이스에서 사용

```kotlin

```
