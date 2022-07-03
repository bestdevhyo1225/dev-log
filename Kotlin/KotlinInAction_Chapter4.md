# [Kotlin In Action - Chapter 4] 클래스, 객체, 인터페이스

## 4.1 클래스 계층 정의

- 코틀린 가시성 / 접근 변경자는 자바와 비슷하지만 `아무것도 지정하지 않은 경우, 기본 가시성은 다르다.`
- 새로 도입한 `sealed` 변경자는 `클래스 상속을 제한한다.`

### 4.1.1 코틀린 인터페이스

- 추상 메서드를 정의할 수 있다.
- 구현이 있는 메서드도 정의할 수 있다. (Java 8의 디폴트 메서드와 비슷하다.)

```kotlin
interface Clickable {
    fun click()

    // 본문이 있는 메서드 정의
    fun showOff() = println("I'm clickable!")
}
```

> 동일한 메서드를 구현하는 다른 인터페이스 정의

```kotlin
interface Focusable {
    fun setFocus(b: Boolean) = println("....")
    fun showOff() = println("I'm focusable!")
}
```

> 위의 두 인터페이스를 구현하고, `showOff()` 메서드를 호출할 경우 명확하게 지정해야 한다.

```kotlin
class Button : Clickable, Focusable {
    override fun click() = println("....")
    override fun showOff() {
        // <> 사이에 "super" 를 지정한다.
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

### 4.1.2 open, final, abstract 변경자 : 기본적으로 final

- 기반 클래스를 변경하는 경우, 하위 클래스의 동작이 예기치 않게 바뀔 수도 있다는 면에서 기반 클래스는 `취약` 하다.
- 코틀린의 클래스와 메서드는 기본적으로 `final` 이다.

> 열린 메서드를 폼하는 열린 클래스 정의하기

```kotlin
open class RichButton : Clickable {
    fun disable() {} // 해당 메서드는 'final' 이다.
    open fun animate() {}
    override fun click() {}
}
```

> 오버라이드 금지하기

```kotlin
open class RichButton : Clickable {
    final override fun click() {} // final 이 없는 override 메서드나 프로퍼티는 기본적으로 열려있다.
}
```

- `abstract` 로 선언한 추상 클래스는 인스턴스화 할 수 없다.
- 추상 멤버는 항상 열려 있기 때문에 `open` 변경자를 `명시할 필요가 없다.`

| 변경자 | 이 변경자가 붙은 멤버는... | 설명 |
| :-: | :-: | :-: |
| final | 오버라이드 할 수 없음 | 클래스 멤버의 기본 변경자이다. |
| open | 오버라이드 할 수 있음 | 반드시 open을 명시해야 오버라이드 할 수 있다. |
| abstract | 반드시 오버라이드 해야 함 | 추상 클래스의 멤버에만 이 변경자를 붙일 수 있으며, 추상 멤버에는 구현이 있으면 안된다. |
| override | 상위 클래스나 상위 인스턴스의 멤버를 오버라이드 하는 중 | 오버라이드하는 멤버는 기본적으로 열려있다. 하위 클래스의 오버라이드를 금지하려면, final을 명시해야 한다. |

### 4.1.3 가시성 변경자: 기본적으로 공개

- 코틀린의 가시성 변경자는 자바와 비슷하다.
- 코틀린의 `기본 가시성만 자바와 다르다.`
- 자바의 기본 가시성인 `패키지 전용(package-private)` 은 코틀린에 없다.
- 패키지 전용 가시성에 대한 대안으로 `internal` 이라는 새로운 가시성 변경자가 있다.
- `internal` 은 모듈 내부에서만 볼 수 있다는 의미이다.

```kotlin
internal open class TalkativeButton : Focusable {
    private fun yell() = println("Hey!")
    protected fun whisper() = println("Let's talk!")
}

// 아래 코드는 오류가 발생함
fun TalktiveButton.giveSpeech() {
    yell()
    whisper()
}
```

- `public` 함수인 `giveSpeech()` 안에서 그보다 가시성이 낮은 타입인 `TalktiveButton` 을 참조하지 못하게 한다.
- 코틀린에서는 같은 패키지에 있는 `protected` 멤버에 접근할 수 없다. (자바와 다름.)

### 4.1.4 내부 클래스와 중첩 클래스 : 기본적으로 중첩 클래스

- 코틀린의 중첩 클래스는 명시적으로 요청하지 않는 한 `바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다.`
- 코틀린 중첩 클래스에 아무런 변경자가 붙지 않으면, `자바 static 중첩 클래스` 와 같다.

| 클래스 A 안에 정의된 클래스 B | 자바 | 코틀린 |
| :-: | :-: | :-: |
| 중첩 클래스 (바깥쪽 클래스에 대한 참조를 저장하지 않음) | static class A | class A |
| 내부 클래스 (바깥쪽 클래스에 대한 참조를 저장함) | class A | inner class A |

> 중첩 클래스

- 중첩 클래스 안에는 바깥쪽 클래스에 대한 내부 참조가 없다.

```kotlin
class Outer {
    class Nested {
    }
}
```

> 내부 클래스

- 내부 클래스 안에는 바깥쪽 클래스에 대한 내부 참조가 있다.

```kotlin
class Outer {
    inner class Inner {
        // 내부 클래스 안에서 바깥쪽 클래스를 참조할 수 있다.
        fun getOuterReference(): Outer = this@Outer
    }
}
```

### 4.1.5 봉인된 클래스 : 클래스 계층 정의시 계층 확장 제한 (sealed 변경자)

- 아래의 코드는 항상 else 분기가 꼭 있어야 한다.

```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int {
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.rigth) + eval(e.left)
        // else 분기가 꼭 있어야 한다.
        else -> throw IllegalArgumentException("Unknown expression")
    }
}
```

- 상위 클래스에 `sealed` 변경자를 붙이면, 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다.
- `sealed` 클래스의 하위 클래스를 정의할 때는 반드시 상위 클래스 안에 중첩 시켜야한다.
- `sealed` 변경자로 표시된 클래스는 자동으로 `open` 이다.

```kotlin
sealed class Expr {
    class Num(val value: Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}

fun eval(e: Expr): Int {
    when (e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.rigth) + eval(e.left)
    }
}
```

## 4.2 뻔하지 않은 생성자와 프로피터를 갖는 클래스 선언

- 주 생성자, 부 생성자, 초기화 블록이 존재한다.

### 4.2.1 클래스 초기화: 주 생성자와 초기화 블록

> 주 생성자, 초기화 블록이 있는 코드

```kotlin
// 파라미터가 하나 있는 '주 생성자'
class User constructor(_nickname: String) {

    private val nickname: String

    // 초기화 블록
    init {
        nickname = _nickname
    }
}
```

> 초기화 블록이 없는 코드

```kotlin
class User constructor(_nickname: String) {

    private val nickname = _nickname
}
```

> 주 생성자 비공개

```kotlin
class Secretive private constructor() {}
```

### 4.2.2 부 생성자 : 상위 클래스를 다른 방식으로 초기화

> 다양한 생성자를 지원해야 하는 경우, 부 생성자 방식 사용

```kotlin
open class View {

    constructor(ctx: Context) {}
    constructor(ctx: Context, attr: AttributeSet) {}
}
```

### 4.2.3 인터페이스에 선언된 프로퍼티 구현

- 인터페이스에 있는 프로퍼티 선언에는 뒷받침하는 필드나 게터등의 정보가 들어있지 않다.
- 인터페이스는 아무 상태도 포함할 수 없으므로 상태를 저장할 필요가 있다면, 인터페이스를 구현한 하위 클래스에서 상태 저장을 위한 프로퍼티를 만들어야한다.

> 인터페이스 프로퍼티 및 클래스 구현

```kotlin
interface User {
    val nickname: String
}

class PrivateUser(override val nickname: String) : User

class SubscribingUser(private val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@')
}

class FacebookUser(private val accountId: Int) : User {
    override val nickname: String = getFacebookName(accountId)
}
```

## 4.3 컴파일러가 생성한 메서드 : 데이터 클래스와 클래스 위임

### 4.3.1 모든 클래스가 정의해야 하는 메서드

- 코틀린 클래스도 `toString`, `equals`, `hashCode` 등을 오버라이드 할 수 있다.
- `toString` : 객체의 프로퍼티들을 문자열로 표현할 수 있다.
- `equals` : 코틀린에서는 `==` 연산자를 사용한다. (참조 비교는 `===` 연산자를 사용한다.)
- `hashCode` : `equals` 가 `true` 를 반환하는 두 객체는 반드시 같은 `hashCode` 를 반환해야 한다는 제약이 있다.

```kotlin
class Client(val name: String, val postalCode: Int) {
    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Client) return false
        return name == other.name && postalCode == other.postalCode
    }
    override fun toString(): String = "Client(name=$name, postalCode=$postalCode)"
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
```

### 4.3.2 데이터 클래스 : 모든 클래스가 정의해야 하는 메서드 자동 생성

- `data` 클래스는 `equals`, `toString`, `hashCode` 메서드를 모두 포함하고 있다.

```kotlin
data class Client(val name: String, val postalCode: Int)
```

### 4.3.3 클래스 위임 : by 키워드 사용

- 인터페이스를 구현할 때, `by` 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시할 수 있다.

```kotlin
class DelegatingCollection<T>(
    innerList: Collection<T> = ArrayList()
) : Collection<T> by innerList {
    // 구현...
}
```

## 4.4 object 키워드 : 클래스 선언과 인스턴스 생성

- `object` 키워드는 클래스를 정의하면서 동시에 인스턴스를 생성한다는 공통점이 있다.
- `객체 선언(object declaration)` 은 싱글턴을 정의하는 방법 중 하나이다.
- `동반 객체(companion object)` 는 인스턴스 메서드는 아니지만 어떤 클래스와 관련 있는 메서드와 팩토리 메서드들 담을 때 쓰인다.
- `객체 식` 은 자바의 `무명 내부 클래스(anonymous inner class)` 대신 쓰인다.

### 4.4.1 객체 선언 : 싱글턴 쉽게 만들기

- 코틀린은 `객체 선언` 기능을 통해 싱글턴을 언어에서 기본적으로 지원한다.
- `객체 선언` : 클래스 선언 + 단일 인스턴스의 선언
- 주 생성자, 부 생성자는 `객체 선언` 에 쓸 수 없다.
- 싱글턴 객체는 객체 선언문이 있는 위치에서 생성자 호출 없이 즉시 만들어진다.

```kotlin
object Payroll {
    private val allEmployees = arrayListOf<Person>()

    fun calculateSalary() {
        for (person in allEmployees) {
            TODO("Not yet implemented")
        }
    }
}
```

### 4.4.2 동반 객체 : 팩토리 메서드와 정적 멤버가 들어갈 장소

- 코틀린 언어는 자바 static 키워드를 지원하지 않는다.
- `companion` 키워드를 사용해서 자바의 정적 메서드 호출이나 정적 필드 사용을 할 수 있도록 도와준다.

```kotlin
class A {
    companion object {
        fun bar() {
            println("Companion object called")
        }
    }
}

A.bar()
```

- `동반 객체` 는 private 생성자를 호출하기 좋은 위치다.
- `동반 객체` 는 자신을 둘러싼 클래스의 모든 private 멤버에 접근할 수 있다.
- `동반 객체` 는 팩토리 패턴을 구현하기 가장 적합한 위치다.

```kotlin
class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
        fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
    }
}
```

- `동반 객체` 에도 이름을 붙일 수 있다.

```kotlin
class Person(val name: String) {
    companion object Loader {
        fun fromJSON(jsonText: String): Person {
            TODO("Not yet implemented")
        }
    }
}

val person1 = Person.Loader.fromJSON("{name: 'Jang'}")
val person2 = Person.fromJSON("{name: 'Kim'}")
```

- `동반 객체` 에서 인터페이스를 구현할 수 있다.

```kotlin
interface JSONFactory<T> {
    fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
    companion object : JSONFactory<Person> {
        override fun fromJSON(jsonText: String): Person {
            TODO("Not yet implemented")
        }
    }
}
```

### 4.4.4 객체 식 : 무명 내부 클래스를 다른 방식으로 작성

- 무명 객체(anonymous object)를 정의할 때도 `object` 키워드를 사용한다.
- 객체 식은 클래스를 정의하고 그 클래스에 속한 인스턴스를 생성하지만, 그 `클래스나 인스턴스에 이름을 붙이지 않는다.` (자바의 무명 내부 클래스를 대신한다.)
- 객체 선언과 달리 **`무명 객체는 싱글턴이 아니다.`** 객체 식이 쓰일 때마다 새로운 인스턴스가 생성된다.

```kotlin
val listener = object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {}
    override fun mouseEntered(e: MouseEvent) {}
}
```
