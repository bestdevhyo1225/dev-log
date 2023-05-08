# Spring 환경 변수

## 우선순위 이해방법

- 더 `유연한 것` 이 우선권을 가진다.
  - 변경하기 어려운 파일보다 실행시, 원하는 값을 줄 수 있는 `커맨드 라인 옵션 인수` 이 더 우선권을 가진다.
- `범위` 가 넓은 것보다 `좁은 것` 이 우선권을 가진다.
  - `OS 환경 변수` 보다 `자바 시스템 속성` 이 더 높은 우선권을 가진다.
  - `자바 시스템 속성` 보다 `커맨드 라인 인수 옵션` 이 더 높은 우선권을 가진다.

## 자주 사용하는 우선순위

> 우선순위는 1번이 제일 높고, 5번이 제일 낮다.

1. `@TestPropertySource` (테스트에서 사용)
2. 커맨드 라인 옵션 인수
3. 자바 시스템 속성
4. OS 환경 변수
5. 설정 데이터 (`application.yml` or `application.properties`)

## 설정 데이터 우선순위

> 우선순위는 1번이 제일 높고, 4번이 제일 낮다.

1. jar `외부 프로필 적용` 파일 `application-{profile}.properties`
2. jar `외부` 파일 `application.properties`
3. jar `내부 프로필 적용` 파일 `application-{profile}.properties` 
4. jar `내부` 파일 `application.properties`

## 참고

- [Spring Externalized Configuration - 우선순위](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)
