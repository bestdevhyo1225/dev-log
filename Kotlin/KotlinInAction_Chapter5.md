# [Kotlin In Action - Chapter 5] 람다로 프로그래밍

## 5.4 자바 함수형 인터페이스 활용

- 함수형 인터페이스 == SAM 인터페이스 (`SAM` 은 단일 추상 메서드라는 뜻이다.)

### 5.4.1 자바 메서드에 람다를 인자로 전달

- 코틀린에서는 람다를 함수에 넘길 수 있다.

```java
void postponeComputation(int delay, Runnable computation);
```

```kotlin
postponeComputation(1_000) { println(42) }
```

- `Runnable` 을 구현하는 무명 객체를 명시적으로 만들어서 사용할 수도 있다.
- 중요한 점은 `객체를 명시적으로 선언` 하는 경우, 메서드를 호출할 때마다 `새로운 객체가 생성된다.`

```kotlin
postponeComputation(1_000, object : Runnable {
    override fun run() {
        println(42)
    }
})
```

- `람다` 의 경우에는 하나의 인스턴스만 사용하고, 호출할 때마다 반복 사용한다.

```kotlin
postponeComputation(1_000) { println(42) }
```

- **람다가 주변 영역의 변수를 포획한다면, 매 호출마다 같은 인스턴스를 사용할 수 없다.**

```kotlin
fun handleComputation(id: String) {
    // 람다 안에서 'id' 변수를 포획하기 때문에 호출할 때마다 새로운 Runnable 인스턴스를 만든다.
    postponeComputation(1_000) { println(id) }
}
```

### 람다의 자세한 구현

- 인라인 되지 않은 모든 람다 식은 무명 클래스로 컴파일 된다.
- 람다가 변수를 포획하면, 무명 클래스 안에 포획한 변수를 저장하는 필드가 생기며, **매 호출마다 무명 클래스의 인스턴스를 새로 만든다.**
- 코틀린에서 **`inline`** 으로 표시된 코틀린 함수에게 람다를 넘기면, **아무런 무명 클래스도 만들어 지지 않는다.**
- 대부분의 코틀린 확장 함수들은 **`inline`** 표시가 붙어있다.
