# Spring Boot Auto Configuration

## 자동 구성 이해

- `resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 경로에 있는 파일을 읽어서 스프링 부트 자동
  구성으로 사용한다.
- `spring-boot-autoconfigure` 를 보면, 아래와 같이 자동 구성이 정의되어 있다.

```imports
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration
org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration
...
...
...
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
```

## 자동 구성 원리

- `@SpringBootApplication` -> `@EnableAutoConfiguration` -> `@Import(AutoConfigurationImportSelector.class)` 순서로 동작한다.
- `@EnableAutoConfiguration` 은 `@AutoConfiguration` 을 활성화한다.
- `AutoConfigurationImportSelector` 클래스는 `@Configuration` 이 아니며, 이 기능을 이해하기 위해서는 `ImportSelector` 에 대해 알아야 한다.

## ImportSelector

- `@Import` 에 설정 정보를 추가하는 방법은 2가지가 있다.
- `정적`
    - `@Import({AConfig.class, BConfig.class})`
    - 코드에 설정 대상이 박혀있다.
- `동적`
    - `@Import(ImportSelector)`
    - 코드로 프로그래밍해서 사용할 설정 대상을 동적으로 사용할 수 있다.
    - 특정 조건에 따라서 설정 정보를 선택해야 하는 경우에 사용된다.
- 참고
  - `boot-source-autoconfig` 프로젝트의 `src/test/selector` 패키지 참고

## AutoConfigurationImportSelector

- `resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 경로에 있는 파일을 열어서 설정 정보를 선택한다.
- 해당 파일의 설정 정보가 `스프링 컨테이너` 에 등록되고 사용된다.
