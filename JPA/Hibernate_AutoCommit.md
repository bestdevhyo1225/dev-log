# Hibernate 구현체에서 Auto Commit 최적화

## Auto Commit

### 자동 커밋

```mysql
set autocommit true; # 자동 커밋 모드 설정
insert into member(member_id, money)
values ('data1', 10000); -- 개별적으로 쿼리가 실행되고, 커밋 됨 (트랜잭션 1)
insert into member(member_id, money)
values ('data2', 10000); -- 개별적으로 쿼리가 실행되고, 커밋 됨 (트랜잭션 2)
```

- 자동 커밋으로 설정하면, `각각의 쿼리 실행 직후에 자동으로 커밋을 호출한다.` 커밋이나 롤백을 호출하지 않아도 되는 장점이 있지만, 자동으로 커밋이 되기 때문에 트랜잭션 기능을 제대로 사용할 수 없다.

### 수동 커밋

```mysql
set autocommit false; -- 수동 커밋 모드 설정
insert into member(member_id, money)
values ('data1', 10000); -- 트랜잭션 1에서 처리됨
insert into member(member_id, money)
values ('data2', 10000); -- 트랜잭션 1에서 처리됨
commit; -- 수동 커밋 설정인 상태에서 작업이 완료되면, 꼭 commit, rollback 처리 할 것
set autocommit true; -- 커넥션 풀을 사용하는 경우는 자동 커밋 모드로 다시 변경한다.
```

- 보통 자동 커밋 모드가 기본으로 설정된 경우가 많기 때문에 `수동 커밋 모드로 설정하는 것을 트랜잭션 시작 한다고 표현할 수 있다.` 수동 커밋 모드를 설정하면, 작업 이후에 `commit`
  또는 `rollback`
  을 꼭 호출해야 한다.

## Hibernate Auto Commit 최적화

```mysql
set autocommit false; -- 수동 커밋 모드 설정
insert into member(member_id, money values ('data1', 10000);
commit;
set autocommit true; -- 커넥션 풀을 사용하는 경우는 자동 커밋 모드로 다시 변경한다.
```

- `JPA` 의 구현체인 `Hibernate` 를 사용하는 경우, 트랜잭션을 시작할 때마다 매번 `set autocommit false` 쿼리를 실행하고, `commit` or `rollback`
  후에는 `set autocommit true` 를 실행한다.

```yaml
spring:
  datasource:
    hikari:
      auto-commit: false # auto commit 상태가 'false' 인 상태에서 트랜잭션을 시작하겠다.
```

- `spring.datasource.hikari.auto-commit` 를 `false` 로 두면, `수동 커밋 모드` 상태로 트랜잭션을 시작하는데, 이렇게만 설정한다고 해도 수동 커밋 모드로 실행되지 않고, 자동
  커밋 모드로 실행되기 때문에 `Hibernate` 옵션도 설정해줘야 한다.

```yaml
spring:
  jpa:
    properties:
      hibernate:
        provider_disables_autocommit: true
```

- `spring.jpa.properties.hibernate.provider_disables_autocommit` 를 `true` 로 설정하면, `Database Connection Pool` 의 커밋 모드 설정을
  신뢰한다는 의미로 해석할 수 있고, `Hibernate` 에서는 커밋 모드와 관련해서 쿼리를 실행하지 않는다.

## Hibernate Auto Commit 동작 원리?

### provider_disables_autocommit 값이 false 상태인 경우

- `Hibernate` 에서는 `Database Connection Pool` 의 자동 커밋 설정을 신뢰하지 않는다. 이에 따라 매번 동적으로 커넥션을 획득해서 매번 설정을 변경한다.
  `LogicalConnectionManagedImpl.java` 파일의 소스 코드를 분석해보면, 다음과 같다.

```java
public class LogicalConnectionManagedImpl extends AbstractLogicalConnectionImplementor {
    @Override
    public void begin() {
        initiallyAutoCommit = !doConnectionsFromProviderHaveAutoCommitDisabled()
            && determineInitialAutoCommitMode(getConnectionForTransactionManagement());
        super.begin();
    }

    @Override
    protected boolean doConnectionsFromProviderHaveAutoCommitDisabled() {
        return providerDisablesAutoCommit;
    }
}
```

- 가장 먼저 `begin()` 메소드를 실행한다. 이 때, `doConnectionsFromProviderHaveAutoCommitDisabled()` 메소드를
  호출한다. `doConnectionsFromProviderHaveAutoCommitDisabled()` 메소드에서는 우리가 `application.yml` 에서
  설정했던 `providerDisablesAutoCommit(provider_disables_autocommit)` 값을 반환한다.
- 즉, `doConnectionsFromProviderHaveAutoCommitDisabled()` 메소드 결과가 `false`
  이면, `!doConnectionsFromProviderHaveAutoCommitDisabled()` 조건에
  따라 `true` 값이 되며, `Database Connection Pool` 의 자동 커밋 옵션을 신뢰하지 않는 상태로 해석하기 때문에 `getConnectionForTransactionManagement()`
  메소드를 호출하여 동적으로 커넥션을 획득한다. 그리고 `determineInitialAutoCommitMode(Connection)` 메소드를 호출한다.

```java
public abstract class AbstractLogicalConnectionImplementor implements LogicalConnectionImplementor, PhysicalJdbcTransaction {
    protected static boolean determineInitialAutoCommitMode(Connection providedConnection) {
        try {
            return providedConnection.getAutoCommit();
        } catch (SQLException e) {
            log.debug("Unable to ascertain initial auto-commit state of provided connection; assuming auto-commit");
            return true;
        }
    }
}
```

- `determineInitialAutoCommitMode(Connection)` 메소드는 부모 클래스인 `AbstractLogicalConnectionImplementor.java` 에서 동작하며,
  현재 Spring 애플리케이션에서 현재 설정되어 있는 `autoCommit` 상태를 호출한다. 다시 `LogicalConnectionManagedImpl.java` 의 `begin()` 메소드로 돌아간다.

```java
public class LogicalConnectionManagedImpl extends AbstractLogicalConnectionImplementor {
    @Override
    public void begin() {
        initiallyAutoCommit = !doConnectionsFromProviderHaveAutoCommitDisabled()
            && determineInitialAutoCommitMode(getConnectionForTransactionManagement());
        super.begin();
    }
}
```

- `begin()` 메소드 내부에서는 `initiallyAutoCommit` 값이 `true` 로 되며, `super.begin()` 을
  통해 `AbstractLogicalConnectionImplementor.java` 부모 클래스의 `begin()` 을 호출한다.

```java
public abstract class AbstractLogicalConnectionImplementor implements LogicalConnectionImplementor, PhysicalJdbcTransaction {
    @Override
    public void begin() {
        try {
            if (!doConnectionsFromProviderHaveAutoCommitDisabled()) {
                log.trace("Preparing to begin transaction via JDBC Connection.setAutoCommit(false)");
                getConnectionForTransactionManagement().setAutoCommit(false);
                log.trace("Transaction begun via JDBC Connection.setAutoCommit(false)");
            }
            status = TransactionStatus.ACTIVE;
        } catch (SQLException e) {
            throw new TransactionException("JDBC begin transaction failed: ", e);
        }
    }
}
```

- `AbstractLogicalConnectionImplementor.java` 의 `begin()` 에서는 `doConnectionsFromProviderHaveAutoCommitDisabled()` 결과를 다시
  한 번 확인하며 `false` 인 경우, `!doConnectionsFromProviderHaveAutoCommitDisabled()` 결과에 따라 `true`
  이기에 `getConnectionForTransactionManagement().setAutoCommit(false);` 를 통해 `set autocommit false` 쿼리를 실행한다.

```java
public abstract class AbstractLogicalConnectionImplementor implements LogicalConnectionImplementor, PhysicalJdbcTransaction {
    protected void resetConnection(boolean initiallyAutoCommit) {
        try {
            if (initiallyAutoCommit) {
                log.trace("re-enabling auto-commit on JDBC Connection after completion of JDBC-based transaction");
                getConnectionForTransactionManagement().setAutoCommit(true);
                status = TransactionStatus.NOT_ACTIVE;
            }
        } catch (Exception e) {
            log.debug(
                "Could not re-enable auto-commit on JDBC Connection after completion of JDBC-based transaction : " + e
            );
        }
    }
}
```

- 그리고 `commit` or `rollback` 이 호출하면, `resetConnection(initiallyAutoCommit)` 메소드를 호출하게 되는데, `initiallyAutoCommit`
  값이 `true` 이기 때문에 `getConnectionForTransactionManagement().setAutoCommit(true);` 를 통해 `set autocommit true` 쿼리를 실행한다.

### provider_disables_autocommit 값이 true 상태인 경우

```java
public class LogicalConnectionManagedImpl extends AbstractLogicalConnectionImplementor {
    @Override
    public void begin() {
        initiallyAutoCommit = !doConnectionsFromProviderHaveAutoCommitDisabled()
            && determineInitialAutoCommitMode(getConnectionForTransactionManagement());
        super.begin();
    }

    @Override
    protected boolean doConnectionsFromProviderHaveAutoCommitDisabled() {
        return providerDisablesAutoCommit;
    }
}
```

- `doConnectionsFromProviderHaveAutoCommitDisabled()` 결과 값이 `true`
  이고, `!doConnectionsFromProviderHaveAutoCommitDisabled()` 결과에 따라 `false` 상태가 된다. 이에 따라 `Database Connection Pool` 의 자동
  커밋 상태를 신뢰하며 `initiallyAutoCommit` 값은 `false` 가 된다. 그리고 `super.begin()` 을 호출한다.

```java
public abstract class AbstractLogicalConnectionImplementor implements LogicalConnectionImplementor, PhysicalJdbcTransaction {
    @Override
    public void begin() {
        try {
            if (!doConnectionsFromProviderHaveAutoCommitDisabled()) {
                log.trace("Preparing to begin transaction via JDBC Connection.setAutoCommit(false)");
                getConnectionForTransactionManagement().setAutoCommit(false);
                log.trace("Transaction begun via JDBC Connection.setAutoCommit(false)");
            }
            status = TransactionStatus.ACTIVE;
        } catch (SQLException e) {
            throw new TransactionException("JDBC begin transaction failed: ", e);
        }
    }
}
```

- `AbstractLogicalConnectionImplementor.java` 의 `begin()` 메소드 내부에서는 하위 클래스에서 앞서 확인했던
  것처럼 `!doConnectionsFromProviderHaveAutoCommitDisabled()` 결과가 `false` 이기
  때문에 `getConnectionForTransactionManagement().setAutoCommit(false);` 메소드를 실행하지 않기 때문에 `set autocommit false` 쿼리를 실행하지
  않는다.

```java
public abstract class AbstractLogicalConnectionImplementor implements LogicalConnectionImplementor, PhysicalJdbcTransaction {
    protected void resetConnection(boolean initiallyAutoCommit) {
        try {
            if (initiallyAutoCommit) {
                log.trace("re-enabling auto-commit on JDBC Connection after completion of JDBC-based transaction");
                getConnectionForTransactionManagement().setAutoCommit(true);
                status = TransactionStatus.NOT_ACTIVE;
            }
        } catch (Exception e) {
            log.debug(
                "Could not re-enable auto-commit on JDBC Connection after completion of JDBC-based transaction : " + e
            );
        }
    }
}
```

- 마지막으로 `commit` or `rollback` 이 호출되는 경우에도 `initiallyAutoCommit` 값이 `false` 임에
  따라 `getConnectionForTransactionManagement().setAutoCommit(true);` 메소드를 호출하지 않기 때문에 `set autocommit true` 쿼리가 실행되지 않는다.

## 정리

- `Hibernate` 를 사용하고 있다면, `spring.datasource.hikari.auto-commit` 옵션 뿐만
  아니라 `spring.jpa.properties.hibernate.provider_disables_autocommit` 옵션 값도 변경해야 한다.
- `spring.jpa.properties.hibernate.provider_disables_autocommit` 값을 `true` 로 설정했다면, `Database Connection Pool` 의 자동 커밋
  설정을 신뢰하는 의미로 해석하기 때문에 `spring.datasource.hikari.auto-commit` 옵션 값의 상태를 반드시 확인해야 한다.
- `spring.datasource.hikari.auto-commit` 옵션 값이 `true` 인
  상태에서 `spring.jpa.properties.hibernate.provider_disables_autocommit` 옵션 값이 `true` 이면, 즉시 쿼리를 실행하기 때문에 트랜잭션 내부에서 실행되던 모든
  쿼리의 `rollback` 이
  정상적으로 실행되지 않는 점을 반드시 인지해야 한다.
- SpringBoot 2.x 부터는 커넥션 풀의 auto-commit 설정값에 따라 자동으로 `provider_disables_autocommit` 을 `true` 상태로 설정을 해준다.
