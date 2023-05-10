# Spring - Actuator (Health, Info, Logger, HTTP, Security)

## :pushpin: 자주 사용하는 EndPoint 목록

- `beans` : 스프링 컨테이너에 등록된 스프링 `Bean` 을 보여준다.
- `conditions` : `condition` 을 통해서 `Bean` 을 등록할 때, 평가 조건과 일치하거나 일치하지 않는 이유를 표시한다.
- `configprops` : `@ConfigurationProperties` 를 보여준다.
- `env` : `Environment` 를 보여준다.
- `health` : 애플리케이션 헬스 정보를 보여준다.
- `httpexchanges` : HTTP 호출 응답 정보를 보여준다. (`HttpExchangeRepository` 를 구현한 별도의 `Bean` 을 등록해야 한다.)
- `info` : 애플리케이션 정보를 보여준다.
- `loggers` : 애플리케이션 Logger 설정을 보여주고, 변경도 할 수 있다.
- `metrics` : 애플리케이션 메트릭 정보를 보여준다.
- `mappings` : `@RequestMapping` 정보를 보여준다.
- `threaddump` : 스레드 덤프를 실행해서 보여준다.
- `shutdown` : 애플리케이션을 종료한다. 이 기능은 `기본적으로 비활성화` 되어 있다.

## :pushpin: Health 정보

- `management.endpoint.health.show-details=always` 옵션을 통해 자세한 헬스 정보를 볼 수 있다.
- `db`, `diskSpace`, `ping`, `mongo`, `redis` 및 기타 정보들을 확인할 수 있다.

```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "H2",
        "validationQuery": "isValid()"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 250685575168,
        "free": 38241878016,
        "threshold": 10485760,
        "path": "/Users/janghyoseog/Documents/boot-source-20230228/start/actuator/.",
        "exists": true
      }
    },
    "ping": {
      "status": "UP"
    }
  }
}
```

- 너무 자세한 것 같으면, `show-components=always` 를 사용하면 된다.

```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP"
    },
    "diskSpace": {
      "status": "UP"
    },
    "ping": {
      "status": "UP"
    }
  }
}
```

- 하나라도 `Health` 에 문제가 생기면, `DOWN` 상태가 된다.

```json
{
  "status": "DOWN",
  "components": {
    "db": {
      "status": "DOWN"
    },
    "diskSpace": {
      "status": "UP"
    },
    "ping": {
      "status": "UP"
    }
  }
}
```

## :pushpin: Info 정보

- `java` : 자바 런타임 정보
- `os` : OS 정보
- `env` : `Environment` 에서 `info.` 로 시작하는 정보
- `build` : 빌드 정보 (`META-INF/build-info.properties` 파일이 필요하다.)
- `git` : `git` 정보 (`git.properties` 파일이 필요하다.)
- `java`, `os`, `env` 는 기본적으로 비활성화 되어 있다.

### Build 설정하기

- `build` 정보는 `build.gradle` 파일에 아래 내용을 입력하면 된다.

```gradle
springBoot {
    buildInfo()
}
```

- `Gradle` 로 애플리케이션을 실행하면, `build/resources/main/META-INF/build-info.properties` 파일이 생성된다.

```properties
build.artifact=actuator
build.group=hello
build.name=actuator
build.time=2023-05-10T08\:00\:21.627259Z
build.version=0.0.1-SNAPSHOT
```

### Git 설정하기

- `git` 으로 관리되는 프로젝트여야 한다.
- `build.gradle` 에 아래 플러그인을 넣는다.

```gradle
plugins {
    id 'com.gorylenko.gradle-git-properties' version '2.4.1'
}
```

- `Gradle` 로 애플리케이션을 실행하면, `build/resources/main/git.properties` 파일이 생성된다.

```properties
git.branch=?
git.build.host=?
git.build.user.email=?
git.build.user.name=?
git.build.version=?
git.closest.tag.commit.count=
git.closest.tag.name=
git.commit.id=?
git.commit.id.abbrev=?
git.commit.id.describe=
git.commit.message.full=?
git.commit.message.short=?
git.commit.time=?
git.commit.user.email=?
git.commit.user.name=?
git.dirty=true
git.remote.origin.url=?
git.tags=
git.total.commit.count=?
```

### Build, Git 설정 후, Info 호출

- `http://localhost:8080/actuator/info` 하면, 아래의 정보를 확인할 수 있다.

```json
{
  "app": {
    "name": "hello-actuator",
    "company": "hs"
  },
  "git": {
    "branch": "main",
    "commit": {
      "id": "a5436c4",
      "time": "2023-05-10T08:04:57Z"
    }
  },
  "build": {
    "artifact": "actuator",
    "name": "actuator",
    "time": "2023-05-10T08:10:48.196Z",
    "version": "0.0.1-SNAPSHOT",
    "group": "hello"
  },
  "java": {
    "version": "17.0.5",
    "vendor": {
      "name": "Oracle Corporation"
    },
    "runtime": {
      "name": "Java(TM) SE Runtime Environment",
      "version": "17.0.5+9-LTS-191"
    },
    "jvm": {
      "name": "Java HotSpot(TM) 64-Bit Server VM",
      "vendor": "Oracle Corporation",
      "version": "17.0.5+9-LTS-191"
    }
  },
  "os": {
    "name": "Mac OS X",
    "version": "13.3.1",
    "arch": "x86_64"
  }
}
```

## :pushpin: Logger 정보

- `Logger` 는 `trace > debug > info > warn > error` 순서대로 출력된다. (즉, 우선순위가 있다.)
- 만약에 아래의 설정을 `debug` 로 해두고

```yaml
logging:
  level:
    hello.controller: debug
```

- `/actuator/loggers` 를 호출하면, `DEBUG` 단위로 설정되어 있는 것을 확인할 수 있다.

```json
{
  "levels": [
    "OFF",
    "ERROR",
    "WARN",
    "INFO",
    "DEBUG",
    "TRACE"
  ],
  "loggers": {
    "hello.controller": {
      "configuredLevel": "DEBUG",
      "effectiveLevel": "DEBUG"
    },
    "hello.controller.LogController": {
      "effectiveLevel": "DEBUG"
    }
  }
}
```

- `/actuator/loggers/hello.controller` 호출도 가능하다.

```json
{
  "configuredLevel": "DEBUG",
  "effectiveLevel": "DEBUG"
}
```

### 실시간 로그 레벨 변경

- `loggers` 를 사용하면 애플리케이션을 다시 시작하지 않고, 실시간으로 로그 레벨을 변경할 수 있다.
- `POST http://localhost:8080/actuator/loggers/hello.controller` 를 요청하면
    - `Content-Type` 도 `application/json` 으로 전달해야 한다.

```json
{
  "configuredLevel": "TRACE"
}
```

## :pushpin: HTTP 요청 및 응답 기록

- `httpexchanges` 엔드포인트를 사용하면 된다.
- `HttpExchangeRepository` 인터페이스의 구현체를 `Bean` 으로 등록해야한다.
- 스프링 부트는 기본적으로 `InMemoryHttpExchangeRepository` 구현체를 제공한다.

```java

@SpringBootApplication
public class ActuatorApplication {

    public static void main(String[] args) {
        SpringApplication.run(ActuatorApplication.class, args);
    }

    @Bean
    public InMemoryHttpExchangeRepository httpExchangeRepository() {
        return new InMemoryHttpExchangeRepository();
    }
}
```

- 참고로 위의 기능은 너무 단순하기 때문에 `개발 단계` 에서만 `사용` 하고, `실제 운영 환경` 에서는 `모니터링 툴` 을 사용해야 한다.

## :pushpin: 보안

- `Actuator EndPoint` 들은 외부 인터넷에서 접근이 불가능하고, 내부에서만 접근 가능한 내부망을 사용하는 것이 안전하다.

### 다른 포트에서 실행

```yaml
management:
  server:
    port: 9292
```

## :pushpin: 참고

[인프런 - 스프링부트-핵심원리-활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%ED%95%B5%EC%8B%AC%EC%9B%90%EB%A6%AC-%ED%99%9C%EC%9A%A9)
