# 도메인(Domain), 도메인 모델(Domain Model) 정리

<br>

## :book: 개발자 입장에서 도메인(Domain)이란 무엇일까?

온라인 서점으로 예시를 들어보면 `온라인 서점`은 개발자가 구현해야 할 소프트웨어의 대상이 된다. 필요한 기능으로는 `상품 조회`, `구매`, `결제`, `배송`, `추적`등의 기능이 있다. 여기서 `온라인 서점`은 소프트웨어로 해결하고자 하는 문제 영역 즉, `도메인(Domain)`에 해당한다.

<br>

## :book: 도메인 모델(Domain Model)이란?

도메인 모델은 기본적으로 도메인 자체를 이해하기 위한 개념 모델이다.

> ex) 클래스 다이어그램(객체 기반 모델), 상태 다이어그램, 그래프, 수학 공식을 활용한 도메인 모델 등등...

개념 모델과 구현 모델은 서로 다른 것이지만 구현 모델이 개념 모델을 최대한 따르드록 할 수 있다.

> ex) 객체 기반 모델을 이용했다면, 객체 지향 언어를 이용해서 개념 모델에 가깝게 표현할 수 있다.

<br>

## :book: 도메인 모델 패턴

|                  계층(Layer)                  |                                                      설명                                                      |
| :-------------------------------------------: | :------------------------------------------------------------------------------------------------------------: |
| 사용자 인터페이스(UI) 또는 표현(Presentation) |                              사용자의 요청을 처리하고 사용자에게 정보를 보여준다.                              |
|               응용(Application)               | 사용자가 요청한 기능을 실행한다. 비즈니스 로직을 직접 구현하지 않으며, 도메인 계층을 조합해서 기능을 실행한다. |
|                도메인(Domain)                 |                                   시스템이 제공할 도메인의 규칙을 구현한다.                                    |
|        인프라스트럭쳐(Infrastructure)         |       외부 시스템과의 연동을 처리한다. (ex. Sequelize, TypeORM, Mali, Koa, Express, gRPC-Caller 등등...)       |

도메인 계층은 핵심 규칙을 구현한다.

- `출고 전에 배송지를 변경할 수 있다.`

- `주문 취소는 배송 전에만 할 수 있다.`

```java
public enum OrderState {
  PAYMENT_WAITING, PREPARING, SHIPPED, DELIVERING, DELIVERY_COMPLETED;
}

public class Order {
  private OrderState state;
  private ShippingInfo shippingInfo;

  private boolean isShippingChangeable() {
    return this.state == OrderState.PAYMENT_WAITING || this.state == OrderState.PREPARING;
  }

  public void changeShippingInfo(ShippingInfo newShippingInfo) {
    if (!this.isShippingChangeable()) {
      throw new IllegalStateException("can't change shipping in " + this.state);
    }
    this.shippingInfo = newShippingInfo
  }
}
```

중요한 점은 주문과 관련된 중요 업무 규칙을 도메인 모델인 Order나 OrderState에서 구현한다는 점이다. 핵심 규칙을 구현한 코드는 도메인 모델에만 위치하기 때문에 **`규칙이 바뀌거나 규칙을 확장해야 할 때 다른 코드에 영향을 덜 주고 변경 내역을 모델에 반영할 수 있게 된다.`**

<br>

## :book: 도메인 모델 도출

```
- 최소 한 종류 이상의 상품을 주문해야 한다.
- 한 상품을 한 개 이상 주문할 수 있다.
- 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다.
- 각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다.
- 주문할 때 배송지 정보를 반드시 지정해야 한다.
- 배송지 정보는 받는 사람, 이름, 전화번호, 주소로 구성된다.
- 출고를 하면, 배송지 정보를 변경할 수 없다.
- 출고 전에 주문을 취소할 수 있다.
- 고객이 결제 완료하기 전에는 상품을 준비하지 않는다.
```

위와 같은 요구사항에서 알 수 있는 것은 주문은 아래와 같이 4가지의 기능을 제공한다.

- 출고 상태로 변경하기

- 배송지 정보 변경하기

- 주문 취소하기

- 결제 완료로 변경하기

```java
// 주문 항목을 표현하는 OrderLine
public class OrderLine {
  private Product product;
  private int price;
  private int quantity;
  private int amounts;

  public OrderLine(Product product, int price, int quantity) {
    this.product = product;
    this.price = price;
    this.quantity = quantity;
    this.amounts = this.calculateAmounts();
  }

  private int calculateAmounts() {
    return this.price * this.quantity;
  }

  public int getAmounts() {
    return this.amounts;
  }
}
```

한 종류 이상의 상품을 주문할 수 있으므로 Order는 최소 한 개 이상의 OrderLine을 포함해야 한다. 또한, OrderLine으로부터 총 주문 금액을 구할 수 있다.

```java
public class Order {
  private List<OrderLine> orderLines;
  private int totalAmounts;

  public Order(List<OrderLine> orderLines) {
    this.setOrderLines(orderLines);
  }

  private void setOrderLines(List<OrderLine> orderLines) {
    this.verifyAtLeastOneOrMoreOrderLines(orderLines);
    this.orderLines = orderLines;
    this.calculateTotalAmounts();
  }

  private void verifyAtLeastOneOrMoreOrderLines(List<OrderLine> orderLines) {
    if (orderLines == null || orderLines.isEmpty()) {
      throw new IllegalArgumentException("no OrderLines");
    }
  }

  private void calculateTotalAmounts() {
    this.totalAmounts = new Money(this.orderLines.stream().mapToInt(x -> x.getAmounts().getValue()).sum());
  }
}
```

OrderLine이 최소 한개 이상인지 검사하고, 그 다음 작업들을 처리한다. 뿐만 아니라 요구 사항중에서 `'주문할 때 배송지 정보를 반드시 지정해야 한다.'`라는 내용도 담겨있어야 한다.

```java
public class ShippingInfo {
  // 생략...
}

public class Order {
  private List<OrderLine> orderLines;
  private int totalAmounts;
  private ShippingInfo shippingInfo;

  public Order(List<OrderLine> orderLines, ShippingInfo shippingInfo) {
    this.setOrderLines(orderLines);
    this.setShippingInfo(shippingInfo);
  }

  // orderLine 관련 생략...(위의 Order 코드 참고)

  private void setShippingInfo(ShippingInfo shippingInfo) {
    if (shippingInfo == null) {
      throw new IllegalArgumentException("no ShippingInfo");
    }
    this.shippingInfo = shippingInfo;
  }
}
```

위의 요구사항에 따르면, `출고 상태가 되기 전과 후의 제약 사항`, `결제 완료 전을 의미하는 상태와 결제 완료`에 따른 여러가지 상태를 처리할 필요가 있다.

```java
public enum OrderState {
  PAYMENT_WAITING, PREPARING, SHIPPED, DELIVERING, DELIVERY_COMPLETED, CANCELED;
}

public class Order {
  private OrderState state;
  // 나머지 변수 생략...

  public void changeShippingInfo(ShippingInfo newShippingInfo) {
    this.verifyNotYetShipped();
    this.setShippingInfo(newShippingInfo);
  }

  public void cancel() {
    this.verifyNotYetShipped();
    this.state = OrderState.CANCELED;
  }

  public void verifyNotYetShipped() {
    if (this.state != OrderState.PAYMENT_WAITING && this.state != OrderState.PREPARING) {
            throw new IllegalStateException("already shipped");
    }
  }

  // 생략...
}
```

<br>

## :bookmark: 참고

- DDD START! - 도메인 주도 설계 구현과 핵심 개념 익히기
