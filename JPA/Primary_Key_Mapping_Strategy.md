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

1. 먼저 엔티티를 데이터베이스에 저장한다.

2. 식별자를 조회해서 엔티티의 식별자에 할당한다.

<br>

## SEQUENCE 전략

- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트다.

- Oracle, PostgreSQL, DB2, H2 데이터베이스에서 사용

```kotlin
@Entity
@SequenceGenerator(
  name = "MEMBER_SEQ_GENERATOR",
  sequenceName = "MEMBER_SEQ", // 매핑할 데이터베이스 시퀀시 이름
  initialValue = 1, allocationSize = 1)
class Member {

  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE,
                  generator = "MEMBER_SEQ_GENERATOR")
  @Column(name = "member_id")
  val id: Long? = null

}
```

SEQUENCE 전략은 entityManager.persist()를 호출할 때, 다음과 같이 동작한다.

1. 데이터베이스 시퀀스를 사용해서 식별자를 조회한다.

2. 조회한 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 저장한다.

### @SequenceGenerator

|      속성       |                                    기능                                     |       기본값       |
| :-------------: | :-------------------------------------------------------------------------: | :----------------: |
|      name       |                             식별자 생성기 이름                              |        필수        |
|  sequenceName   |                  데이터베이스에 등록되어 있는 시퀀스 이름                   | hibernate_sequence |
|  initialValue   | DDL 생성시에만 사용됨, 시퀀스 DDL을 생성할 때, 처음 시작하는 수를 지정한다. |         1          |
| allocationSize  |           시퀀스 한 번 호출에 증가하는 수 (성능 최적화에 사용됨)            |         50         |
| catalog, schema |                      데이터베이스 catalog, schema 이름                      |                    |

### SEQUENCE 전략과 최적화

SEQUENCE 전략은 데이터베이스 시퀀스를 통해 식별자를 조회하는 추가 작업이 필요함. 따라서 데이터베이스와 2번 통신한다.

> 가장 먼저 식별자를 구하려고 데이터베이스 시퀀스를 조회한다.

```sql
SELECT MEMBER_SEQ.NEXTVAL FROM DUAL
```

> 조회한 시퀀스를 기본키 값으로 사용해 데이터베이스에 저장한다.

```sql
INSERT INTO MEMBER...
```

여기서 JPA는 시퀀스에 접근하는 횟수를 줄이기 위해 `@SequenceGenerator.allocationSize`를 사용한다. 즉, `allocationSize`만큼 한 번에 시퀀스 값을 증가시키고 나서 그만큼 메모리에 시퀀스 값을 할당한다. 즉, `allocationSize`가 50이면, 시퀀스를 한 번에 50 증가 시킨 다음에 1 ~ 50까지는 메모리에서 식별자를 할당한다. 그리고 51이 되면, 시퀀스 값을 100으로 증가시킨 다음 51 ~ 100까지 메모리에서 식별자를 할당한다.

<br>

이 최적화 방법은 시퀀스 값을 선점하므로 여러 JVM이 동시에 동작해도 기본키 값이 충돌하지 않는 장점이 있다. 반면에 데이터베이스에 직접 접근해서 데이터를 등록할 때, 시퀀스 값이 한 번에 많이 증가한다는 점을 염두해두어야 한다. 이런 상황이 부담스럽고 `INSERT` 성능이 중요하지 않다면, `allocationSize`의 값을 1로 설정하면 된다.

<br>

마지막으로 최적화를 하려면, application.yml 에서 `hibernate.id.new_generator_mappings` 속성을 `true`로 설정해야 한다. 이 속성을 적용하지 않으면, 하이버네이트가 과거에 사용하던 방법으로 키 생성을 최적화한다고 한다.

```yml
spring:
  jpa:
    properties:
      hibernate:
        id:
          new_generator_mappings: true
```

> 하이버네이트는 과거에 시퀀스 값을 하나씩 할당받고, 애플리케이션에서 allocationSize만큼 사용했다.

```
allocationSize가 50으로 설정했다고 가정한 상황에서

시퀀스 값이 1이면? -> 애플리케이션에서 1 ~ 50 까지 사용

시퀀스 값이 2이면? -> 애플리케이션에서 51 ~ 100 까지 사용
```

<br>

## 참고

- 자바 ORM 표준 - JPA 프로그래밍
