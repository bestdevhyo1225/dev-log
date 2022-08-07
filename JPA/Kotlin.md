# Kotlin 기반의 JPA 사용하는 방법 정리 (개발시 참고)

## Entity 클래스의 private 생성자를 사용할 수 있음

- [distributed-transaction-demo repository pr#17 - 엔티티 생성자에 private 접근 제어자 추가](https://github.com/bestdevhyo1225/distributed-transaction-demo/pull/17)

위의 `distributed-transaction-demo` 저장소 PR에는 아래와 같은 내용이 정리되어 있다.

- JPA Entity에는 `private` 생성자를 적용하면 에러가 발생하는 줄 알았음.
- `kotlin plugin.jpa` 은 `noarg` 플러그인을 사용하여 자동으로 `@Entity`, `@Embeddable`, `@MappedSuperClass` 이 적용되어 있는 클래스에 `NoArg`
  생성자를
  만들어준다.
