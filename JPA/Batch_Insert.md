# JPA 에서 Batch Insert 사용하기

## JPA Batch Insert 지원하는 경우

- `@GeneratedValue(strategy = GenerationType.Sequence)`
- `@GeneratedValue(strategy = GenerationType.TABLE)`
- `@GeneratedValue` 를 설정하지 않는 경우

## JPA Batch Insert 지원하지 않는 경우

- `@GeneratedValue(strategy = GenerationType.IDENTITY)` 의 경우, 지원하지 않는다.

## 테스트를 위한 application.yml 환경 설정

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/batch_study?useSSL=false&serverTimezone=UTC&autoReconnect=true&rewriteBatchedStatements=true
    driver-class-name: com.mysql.cj.jdbc.Driver
  
  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        jdbc:
          batch_size: 50
        order_inserts: true
        order_updates: true

logging:
  level:
    org.hibernate.SQL: debug
```

- `spring.datasource.url` 에 `rewriteBatchedStatements=true` 추가
- `spring.jpa.properties.jdbc.batch_size` 에 `50` 설정
- `spring.jpa.properties.order_inserts` 에 `true` 설정
- `spring.jpa.properties.order_updates` 에 `true` 설정

## Hibernate 로그

- `multi values` 방식의 INSERT 쿼리 로그가 보이지 않으며, `개별 INSERT 쿼리 로그` 를 확인할 수 있다.

## MySQL 레벨에서 로그 확인하기

```mysql-sql
show variables like 'general_log%';
set global general_log = 'ON';
```

- `set global general_log = 'ON';` 쿼리로 로그를 활성화한다.
  - 주의할 점은 `개발 및 테스트 환경` 에서만 위의 옵션을 사용해야 한다.
  - `ON` 상태일 경우, 성능에 지장을 줄 수 있다.
- `multi values` 의 INSERT 쿼리 로그를 확인할 수 있다. 
