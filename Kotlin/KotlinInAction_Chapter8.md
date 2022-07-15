# [Kotlin In Action - Chapter 8] 고차 함수: 파라미터와 반환 값으로 람다 사용

## 8.1 고차 함수 정의

- 고차 함수는 `다른 함수를 인자로 받거나 함수를 반환하는 함수` 이다.

### 8.1.1 함수 타입

- 함수 타입은 `(파라미터 타입) -> 반환 타입` 으로 구성되어 있다.
    - `(Int, String) -> Unit`

```kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
val action: () -> Unit = { println(42) }
```

- 반환 타입이 `null` 이 될 수 있는 타입으로 지정할 수 있다.

```kotlin
val canReturnNull: (Int, Int) -> Int? = { x, y -> null }
```

- `null` 이 될 수 있는 함수 타입 변수를 정의할 수도 있다.

```kotlin
val funOrNull: ((Int, Int) -> Int)? = null
```

### 8.1.2 인자로 받은 함수 호출

```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
    val result = operation(2, 3)
    println("The result is $result")
}
```

### 8.1.3 자바에서 코틀린 함수 타입 사용

- 컴파일된 코드 안에서 함수 타입은 일반 인터페이스로 바뀐다. 함수 타입의 변수는 `FunctionN` 인터페이스를 구현하는 객체를 저장한다.
- 코틀린 표준 라이브러리는 함수 인자의 개수에 따라 `Function0<R>`, `Function1<P1, R>` 등의 인터페이스를 제공한다.
- 각 인터페이스에는 `invoke` 메서드 정의가 하나 들어있다.

### 8.1.4 디폴트 값을 지정한 함수 타입 파라미터나 null이 될 수 있는 함수 타입 파라미터

- 함수 타입의 파라미터에 대한 `디폴트 값` 을 지정할 수 있다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = { it.toString() } // 함수 타입 파라미터를 선언하면서, 람다를 디폴트 값으로 지정한다.
)
```

### 8.1.5 함수를 함수에서 반환

- 함수의 반환 타입으로 함수 타입을 지정하면, 함수를 반환할 수 있다.

```kotlin
enum class Delivery { STANDARD, EXPEDITED }

class Order(val itemCount: Int)

fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double {
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }

    return { order -> 1.2 * order.itemCount }
}

/////// 사용 ///////
val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
val costs = calculator(Order(itemCount = 3))

println("Shipping costs $costs")
```

### 8.1.6 람다를 활용한 중복 제거

- 아래의 코드는 웹 사이트 방문 기록을 분석하는 예제이다.

```kotlin
enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS,
)

val log = listOf(
    SiteVisit(path = "/", duration = 34.0, os = OS.WINDOWS),
    SiteVisit(path = "/", duration = 22.0, os = OS.MAC),
    SiteVisit(path = "/login", duration = 12.0, os = OS.WINDOWS),
    SiteVisit(path = "/signup", duration = 8.0, os = OS.IOS),
    SiteVisit(path = "/", duration = 16.3, os = OS.ANDROID)
)
```

- 위의 코드를 통해 `WINDOWS` 사용자의 평균 방문 시간을 출력하려면 아래와 같이 코드를 작성해야 한다.

```kotlin
val averageWindowsDuration = log
    .filter { it.os == OS.WINDOWS }
    .map(SiteVisit::duration)
    .average()
```

- 만약, `MAC` 사용자의 평균 방문 시간을 출력하려면, 중복 코드가 존재하게 되는데 OS를 파라미터로 뽑아서 중복을 제거할 수 있다.

```kotlin
fun List<SiteVisit>.averageDurationFor(os: OS) =
    filter { it.os == os }
        .map(SiteVisit::duration)
        .average()
```

## 8.2 인라인 함수: 람다의 부가 비용 없애기

- 람다를 무명 클래스로 컴파일 하지만 `람다식을 사용할 때마다 새로운 클래스가 만들어지지 않는다.`
- 람다가 `변수를 포획` 하면, `람다가 생성되는 시점마다 새로운 무명 클래스 객체가 생긴다.`
- `inline` 변경자를 어떤 함수에 붙이면, 컴파일러는 그 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 바꿔치기 해준다.

### 8.2.1 인라이닝이 작동하는 방식

- 함수를 호출하는 코드를 함수를 호출하는 바이트코드 대신에 `함수 본문을 번역한 바이트코드로 컴파일한다.`
