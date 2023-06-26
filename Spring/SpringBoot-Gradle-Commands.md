# Spring Boot 사용시, 자주 사용하는 Gradle 명령어 모음

## 특정 모듈만 Task

```shell
# 특정 모듈만 clean, test는 제외 (-x, --exclude-task)
./gradlew :domain:rdbms:jpa:clean -x test

# 특정 모듈만 build, test는 제외 (-x, --exclude-task)
./gradlew :domain:rdbms:jpa:build -x test

# 특정 모듈만 test, build는 제외 (-x, --exclude-task)
./gradlew :domain:rdbms:jpa:test -x build
```
