# 영속성 컨텍스트

<br>

## 영속성 컨텍스트란?

![image](https://user-images.githubusercontent.com/23515771/90488896-895aab80-e177-11ea-8a3b-5d4a69a3ccb3.png)

- Entity를 영구 저장하는 환경이다.

- 눈에 보이지 않는 논리적인 개념이다.

- Entity Manager Factory 와 Entity Manager 를 활용해서 영속성 컨텍스트에 접근할 수 있다.

<br>

## Entity Manager Factory

- 사용자에게 요청이 올때마다 Entity Mananger를 생성한다.

<br>

## Entity Manager

- 내부적으로 Database Connection을 사용해서 Database를 사용하게 한다.

<br>

## Entity의 생명주기

**`비영속`** - 영속성 컨텍스트와 전혀 관계 없는 상태

```java
Member member = new Member();
member.setId("id")
member.setUsername("janghyoseok");
```

**`영속`** - 영속성 컨텍스트에 관리되는 상태 (DB에 저장되지 않는다.)

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory(persistenceUnitName);

EntityManager em = emf.createEntityManager();

em.getTransaction().begin();
em.persist(member);
```

**`준영속`** - 영속성 컨텍스트에서 분리한 상태

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory(persistenceUnitName);

EntityManager em = emf.createEntityManager();

em.detach(member);
```

**`삭제`** - Database에서 영구적으로 삭제하는 상태

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory(persistenceUnitName);

EntityManager em = emf.createEntityManager();

em.remove(member);
```

<br>

## 영속성 컨텍스트의 이점

- 1차 캐시

- 동일성 보장 (Identity)

- 트랜잭션을 지원하는 쓰기 지연 (Transactional Write-Behind)

- 변경 감지 (Dirty Checking)

- 지연 로딩 (Lazy Loading)

<br>

### 1. 엔티티 조회, 1차 캐시

```java
Member member = new Member();
member.setId("member1")
member.setUsername("janghyoseok");

EntityManagerFactory emf = Persistence.createEntityManagerFactory(persistenceUnitName);

EntityManager em = emf.createEntityManager();

// 영속 상태
em.persist(member);

// 1차 캐시에서 조회한다.
Member member = em.find(Member.class, "member1");
```

> 해당 코드를 아래의 이미지로 표현할 수 있고, 조회를 하는 경우에는 1차 캐시에 있는 데이터를 조회한다.

<br>

![image](https://user-images.githubusercontent.com/23515771/90495597-c165ec80-e17f-11ea-98e6-0602d3685f24.png)

> 만약에 1차 캐시에 데이터가 없다면, 아래 이미지 방식으로 동작한다.

<br>

![image](https://user-images.githubusercontent.com/23515771/90511140-02b5c680-e197-11ea-8660-da612a7df986.png)

<br>

> 하지만, 그다지 큰 이점은 없다. Entity Manager는 보통 트랜잭션 단위로 만들고, 트랜잭션이 끝날때 같이 종료하기 때문이다. 따라서 1차 캐시는 한 트랜잭션내에서만 확보하고 있다는 것 (비즈니스 로직이 복잡하면, 큰 이점을 볼 가능성도 있음)

<br>

### 2. 영속 Entity 동일성 보장

```java
Member findMember1 = em.find(Member.class, 1L);
Member findMember2 = em.find(Member.class, 1L);

system.out.println(a == b); // true
```

> 1차 캐시로 반복 가능한 읽기(Repeatable Read) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공한다.

<br>

### 3. 트랜잭션을 지원하는 쓰기 지연

```java
EntityManager em = emf.createEntityManager();
EntityTransactio transaction = em.getTransaction();

transaction.begin();

em.persist(memberA);
em.persist(memberB);
// 여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

// 커밋하는 순간 데이터베이스 INSERT SQL을 보낸다.
transaction.commit();
```

> 1차 캐시에 MemberA 엔티티를 저장한다.

![image](https://user-images.githubusercontent.com/23515771/90604725-3eed3380-e238-11ea-8916-2a5473f75fe5.png)

<br>

> 1차 캐시에 MemberB 엔티티를 저장한다.

![image](https://user-images.githubusercontent.com/23515771/90605125-e23e4880-e238-11ea-82a5-5ada430a371c.png)

<br>

> transaction 에서 commit 하는 순간 데이터베이스에 INSERT SQL을 보낸다.

![image](https://user-images.githubusercontent.com/23515771/90973403-b1be1d80-e55c-11ea-8f86-ce63be2bf5a6.png)

<br>

### 4. 엔티티 수정시, 변경 감지

> 엔티티 수정의 경우에는 값을 변경하고, `update` 또는 `save`와 같은 메소드를 호출해야할 것 같은데 그렇게 하지 않아도 JPA가 변경을 감지한다.

```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();

transaction.begin();

// 영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");

// 영속 엔티티 데이터 수정
memberA.setUsername("hi");
memberA.setAge(10);

// 변경을 하려면 아래와 같은 코드가 필요하지 않을까 생각하지만, 필요없다.
// em.update(memberA);

transaction.commit();
```

> 1차 캐시에는 스냅샷이 있는데 기존 스냅샷과 현재 엔티티를 비교하고, 값이 변경되었음을 감지하면 UPDATE SQL을 생성한다.

![image](https://user-images.githubusercontent.com/23515771/90973626-d3200900-e55e-11ea-816c-c0c955bca7de.png)

<br>

## 참고

- [인프런 - 자바 ORM 표준 JPA 프로그래밍 (기본편)](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
