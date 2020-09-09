# JPA 프록시

<br>

## find() vs getReference()

- find()는 데이터베이스를 통해서 실제 엔티티 객체를 조회한다.

- getReference()는 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체를 조회한다.

<br>

## 프록시의 기본적인 특징

- 실제 클래스를 상속 받아서 만들어진다.

- 실제 클래스와 겉모양이 같다.

<br>

## 실제 동작 방식

```java
Member member = em.getReference(Member.class, 1L);
member.getName();
```

위와 같은 코드가 있을때 프록시는 다음과 같이 동작한다.

![image](https://user-images.githubusercontent.com/23515771/92609099-2335f400-f2f1-11ea-9d30-608e3b6e8d67.png)

1. `getName()`을 처음 호출하는 경우

2. 프록시 객체는 영속성 컨텍스트에 초기화 요청을한다.

3. 영속성 컨텍스트에서는 실제 데이터베이스를 조회한다.

4. 데이터베이스 조회 후, 실제 엔티티를 생성한다.

5. 프록시 객체의 target의 참조는 실제 엔티티를 가리키며, `getName()`을 가져온다.

<br>

## 프록시 핵심 (간단 요약)

- 프록시 객체는 처음 사용할 때 한 번만 초기화 된다.

- 프록시 객체를 초기화 할 때, **프록시 객체가 실제 엔티티로 바뀌는 것이 아니라 초기화 되면, 프록시 객체를 통해 실제 엔티티에 접근 가능한 것이다.**

<br>

## 참고

- [인프런 - 자바 ORM 표준 JPA 프로그래밍 (기본편)](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
