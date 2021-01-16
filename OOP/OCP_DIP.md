# 객체 지향 설계 원칙 - OCP, DIP

## OCP (Open Closed Principle)

`개방 폐쇄 원칙`이라 하며, **`소프트웨어 요소는 확장에는 열려 있으나, 변경에는 닫혀 있어야 한다.`** 라는 이론이다.

<br>

## DIP (Dependency Inversion Principle)

`의존 관계 역전 원칙`이며, **`추상화에 의존해야지 구체화에 의존하면 안된다.`** 라는 이론이다.

<br>

## OCP와 DIP를 위반하는 사례

> DiscountPolicy

```java
// 할인 정책과 관련된 인터페이스
public interface DiscountPolicy {
    int discount(Member member, int price);
}
```

> FixDiscountPolicy

```java
@Component
public class FixDiscountPolicy implements DiscountPolicy {

  // 등급에 따른 Fix 할인율을 책정한다.
  @Override
  pubilc int discount(Member member, int price) {
    if (member.getGrade() == Grade.VIP) {
      return 1000;
    }
    return 0;
  }
}
```

> RateDiscountPolicy

```java
@Component
public class RateDiscountPolicy implements DiscountPolicy {

  // 등급에 따른 Rate 할인율을 책정한다.
  @Override
  pubilc int discount(Member member, int price) {
    if (member.getGrade() == Grade.VIP) {
      int discountPercent = 10;
      return price * discountPercent / 100;
    }
    return 0;
  }
}
```

> OrderSerivceImpl

```java
@Service
public class OrderServiceImpl implements OrderService {
  // 기존 정책
  //private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

  // 새로운 정책으로 변경하려 한다...
  private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```

위와 같이 할인 정책과 관련된 `DiscountPolicy` 인터페이스가 있고, 이를 구현한 `FixDiscountPolicy`와 `RateDiscountPolicy` 클래스가 있다. 또한 할인 정책을 `OrderServicImpl` 클래스에서 사용한다. 기존 `FixDiscountPolicy` 에서 `RateDiscountPolicy` 정책으로 변경하게 되면, `OCP`와 `DIP`를 위반하는 사례를 확인할 수 있다.

<br>

## 어떻게 OCP와 DIP를 위반한 사례일까?

> DIP 위반

위의 예시를 통해 의존 관계를 분석해보면, `DiscountPolicy` 인터페이스와 같은 추상화에 의존하고 있지만, `FixDiscountPolicy`와 `RateDiscountPolicy`와 같은 구체화 클래스에도 의존하고 있기 때문에 `DIP`를 위반하고 있다.

> OCP 위반

클라이언트인 `OrderServiceImpl`에서 할인 정책을 `FixDiscountPolicy`에서 `RateDiscountPolicy`으로 변경해야 하기 때문에 클라이언트 코드에 영향을 준다. 따라서 `OCP`를 위반하고 있다.

![image](https://user-images.githubusercontent.com/23515771/104808949-e644b580-582c-11eb-8c4a-c2e26ad6e7d7.png)

<br>

## 해결 방법

우선 여러가지 방법이 있는데 나는 아래와 같은 코드 수정을 통해 문제를 해결해 보려고 한다.

> FixDiscountPolicy

```java
// @Component 제거, @Component를 제거하지 않고 해결하는 방법도 있다!
public class FixDiscountPolicy implements DiscountPolicy {}
```

> RateDiscountPolicy

```java
// @Component 제거, @Component를 제거하지 않고 해결하는 방법도 있다!
public class RateDiscountPolicy implements DiscountPolicy {
```

> AppConfig

- `DiscountPolicy`를 구현한 객체를 생성하고, 연결하는 책임을 가진 별도의 `설정 클래스`이다.

```java
@Configuration
public class AppConfig {

  @Bean
  public DiscountPolicy discountPolicy() {
    // 구체화 클래스를 Bean으로 등록하는 방식...
    return new RateDiscountPolicy();
  }
}
```

> OrderServiceImpl

```java
@Service
public class OrderServiceImpl implements OrderService {

  private final DiscountPolicy discountPolicy;

  // 생성자를 통한 의존 관계 주입 방식(DI)으로 DIP 문제를 해결할 수 있다!
  public OrderServiceImpl(DiscountPolicy discountPolicy) {
      // Bean으로 등록된 구체화 클래스가 주입될 것이다.
      this.discountPolicy = discountPolicy;
  }

}
```

> 참고) 위의 문제를 해결하는 방법은 많으며, 그 중에 하나의 예시일 뿐이다.

위와 같이 `AppConfing`와 같은 설정 클래스를 통해 구체화 클래스를 생성하는 방식으로 `OCP` 설계 원칙을 지킬 수 있으며, 이를 사용하는 `OrderServiceImpl` 클래스에서는 `생성자를 통한 의존 관계 주입 방식(DI)`으로 `DIP` 설계 원칙을 지킬 수 있다.

<br>

## 참고

- [[인프런] 스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/dashboard)
