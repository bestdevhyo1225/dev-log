# 9. 스코프 함수

## 스코프 함수 선택표

| 함수명 | 수신자 객체 참조 방법 | 반환 값 | 확장 함수 여부 |
| :-: | :-: | :-: | :-: |
| let | it | 함수의 결과 | O |
| run | this | 함수의 결과 | O |
| with | this | 함수의 결과 | X |
| apply | this | 컨텍스트 객체 | O |
| also | it | 컨텍스트 객체 | O |

## let 함수

```kotlin
fun main() {
    val str: String? = "안녕"

    val result: Int? = str?.let {
        println(it)

        1234
    }

    println(result)
}
```

## run 함수

```kotlin
class DatabaseClient {

    var url: String? = null
    var username: String? = null
    var password: String? = null

    fun connect(): Boolean {
        println("DB 접속 중...")
        Thread.sleep(1_000)
        println("DB 접속 완료")
        return true
    }
}

fun main() {
    val result: Boolean = DatabaseClient().run {
        url = "localhost:3306"
        username = "root"
        password = "example"
        connect()
    }
    println(result)
}
```

## with 함수

```kotlin
fun main() {
    val result: Boolean = with(DatabaseClient()) {
        url = "localhost:3306"
        username = "root"
        password = "example"
        connect()
    }
    println(result)
}
```

## apply 함수

객체 생성 시점에 내부 프로퍼티를 초기화 해야하는 경우에 사용된다.

```kotlin
fun main() {
    // apply 함수는 함수의 결과를 반환하는게 아니라 해당 객체를 반환한다.
    val databaseClient: DatabaseClient = DatabaseClient().apply {
        url = "localhost:3306"
        username = "root"
        password = "example"
    }

    val result = databaseClient.connect()

    println(result)
}
```

## also 함수

```kotlin
class User(
    val name: String,
    val password: String,
) {

    fun validate() {
        if (name.isNotBlank() && password.isNotBlank()) {
            println("검증 성공!")
        } else {
            println("검증 실패!")
        }
    }

    fun printName() = print(name)
}

fun main() {

    val user: User = User(name = "Hyo", password = "1234").also {
        it.validate()
        it.printName()
    }
}
```
