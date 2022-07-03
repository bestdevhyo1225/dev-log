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
