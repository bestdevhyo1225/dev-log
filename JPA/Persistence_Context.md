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

`비영속` : 영속성 컨텍스트와 전혀 관계 없는 상태

```java
Member member = new Member();
member.setId("id")
member.setUsername("janghyoseok");
```

`영속` : 영속성 컨텍스트에 관리되는 상태 (DB에 저장되지 않는다.)

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory(persistenceUnitName);

EntityManager em = emf.createEntityManager();

em.getTransaction().begin();
em.persist(member);
```

`준영속` : 영속성 컨텍스트에서 분리한 상태

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory(persistenceUnitName);

EntityManager em = emf.createEntityManager();

em.detach(member);
```

`삭제` : Database에서 영구적으로 삭제하는 상태

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

## 엔티티 조회, 1차 캐시

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

해당 코드를 아래의 이미지로 표현할 수 있고, 조회를 하는 경우에는 1차 캐시에 있는 데이터를 조회한다.

![image](https://user-images.githubusercontent.com/23515771/90495597-c165ec80-e17f-11ea-98e6-0602d3685f24.png)

만약에 1차 캐시에 데이터가 없다면, 아래 이미지 방식으로 동작한다.

![image](https://user-images.githubusercontent.com/23515771/90511140-02b5c680-e197-11ea-8660-da612a7df986.png)

하지만, 그다지 큰 이점은 없다. Entity Manager는 보통 트랜잭션 단위로 만들고, 트랜잭션이 끝날때 같이 종료하기 때문이다. 따라서 1차 캐시는 한 트랜잭션내에서만 확보하고 있다는 것 (비즈니스 로직이 복잡하면, 큰 이점을 볼 가능성도 있음)

<br>

## 참고

- [인프런 - 자바 ORM 표준 JPA 프로그래밍 (기본편)](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
