# Axon Framework를 사용하면서 자주사용하는 어노테이션 정리

## Aggregate에서 사용하는 어노테이션

> @Aggregate

- Aggregate임을 알려주는 어노테이션이다.

```java
@Aggregate
public class Account {
}
```

> @AggregateIdentifer

- Aggregate별로 식별자가 반드시 존재해야 되기 때문에 유일성을 갖는 대표키를 지정하는 어노테이션이다.

```java
@Aggregate
public class Account {
  @AggregateIdentifier
  public String accountId;
}
```

<br>

## Command에서 사용하는 어노테이션

> @TargeAggregateIdentifier

- 어떤 Aggregate를 대상으로 명령을 수행할 것인지? 알아야하기 때문에 대상 Aggregate를 지정할 수 있도록 돕는 어노테이션이다.

```java
public class DepositMoneyCommand {
  @TargetAggregateIdentifier
  private final String accountId;
}
```

> @CommandHandler

- Aggregate에 대한 명령이 발생되었을때, 호출되는 메소드임을 알려주는 어노테이션

```java
@Aggregate
public class Account {
  @AggregateIdentifier
  public String accountId;

  // Aggregate를 대상으로 명령이 발생되면, 해당 메소드가 호출된다.
  @CommandHandler
  proteted void on(Command command) {

  }
}
```

> @EventSourcingHandler

- `@CommandHandler`에서 발생한 이벤트를 적용하는 메소드임을 알려주는 어노테이션

- 해당 이벤트가 있으면, Aggregate가 변경되는 방식을 주로 지정한다.

```java
@Aggregate
public class Account {
  @AggregateIdentifier
  public String accountId;

  @CommandHandler
  proteted void on(Command command) {}

  // Event 값이 넘어온다.
  @EventSourcingHandler
  proteted void on(Event event) {

  }
}
```

<br>

## Query에서 사용하는 어노테이션

> @EventHandler

- Command에서 이벤트가 발생했을때, Query쪽에서는 `@EventHandler` 어노테이션이 해당 이벤트를 적용하는 메소드임을 알려준다.

- EventStore에서 이벤트를 살펴보고, `@EventSourcingHandler`를 통해 업데이트 된 Aggregate를 가질 수 있다. (상태 머신이라고 생각할 수 있음)

```java
@Component
public class HolderAccountProjection {
  @EventHandler
  protected void on(Event event) {
  }
}
```
