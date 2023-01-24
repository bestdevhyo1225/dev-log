# String, StringBuilder, StringBuffer

## String

- 불변성을 갖는다.
- `리터럴` 값으로 할당 하는 경우에는 `Heap` 영역 안에 특별한 메모리 공간인 `String Constant Pool` 에 저장된다.
    - 새롭게 리터럴 값을 생성하는 것이 아니다.
    - 기존에 존재하는 값이 있으면, 그 값을 그대로 활용한다.
- `new` 키워드로 생성하는 경우에는 `Heap` 영역에 동적으로 메모리 공간이 할당된다.

## StringBuilder vs StringBuffer

- 같은 주소 공간을 활용한다는 점과 가변성을 지니는 공통 특징이 있다.
- 둘은 `동기화(Synchronization)` 라는 차이점을 가지고 있다.
- `StringBuilder` 는 `동기화(Synchronization)` 를 지원하지 않는다.
- `StringBuffer` 는 `동기화(Synchronization)` 를 지원하기 때문에 멀티 스레드 환경에서 안전하게 동작할 수 있다.
    - `StringBuffer` 에서는 `synchronized` 키워드를 사용한다.

> 멀티 스레드 환경에서 StringBuilder를 쓰면 안되는 예

```java
public class Service {

    public static StringBuilder sb = new StringBuilder();

    public void go() {
        sb.append("추가");
        sb.append("추가");
    }
}
```

- 이렇게 전역변수로 `StringBuilder` 를 선언하면 여러 쓰레드에서 동시에 접근하게 되고, `StringBuilder` 는 쓰레드에 안전하지 않기 때문에 자연스럽게 문제가 생긴다.

> 멀티 스레드 환경에서 StringBuilder를 써도 되는 예

```java
public class Service {

    public void go() {
        StringBuilder sb = new StringBuilder();
        sb.append("추가");
        sb.append("추가");
    }
}
```

- 지역변수로 `StringBuilder` 를 선언하면 쓰레드가 메소드를 수행할 때 객체가 생성되고 쓰레드마다 서로 다른 객체를 사용하게 된다. 그렇기 때문에 멀티쓰레드 환경에서 아래와 같이 코딩을 하여도 전혀
  문제가 없다.

### synchronized 키워드

- 여러 개의 스레드에서 하나의 자원에 접근하려고 할 때, 현재 사용하고 있는 스레드를 제외한 나머지 스레드들이 접근할 수 없도록 막는다.
- 예를 들어 멀티스레드 환경에서 A 스레드와 B스레드 모두 같은 StringBuffer 클래스 객체 sb의 append() 메서드를 사용하려고 하면, 다음과 같은 절차를 수행하게 된다.

    - A 스레드 : sb의 append() 동기화 블록에 접근 및 실행
    - B 스레드 : A 스레드 sb 의 append() 동기화 블록에 들어가지 못하고 `block` 상태가 됨.
    - A 스레드 : sb의 append() 동기화 블록에서 탈출
    - B 스레드 : block 에서 running 상태가 되며 sb 의 append() 동기화 블록에 접근 및 실행.

- StringBuilder 클래스 주석에서 동기화가 필요할 경우 StringBuffer을 추천한다는 문구를 확인할 수 있다.

## 예상 면접 질문

- `StringBuilder` 와 `StringBuffer` 의 차이는 무엇일까요?
    - 답변) `동기화(Synchronization)` 지원 유무
- `동기화(synchronized)` 가 걸려있으면 왜 느린걸까요?
    - 답변) 동기화 블록을 접근하고 실행하는 점에 있어서 추가 비용이 들며, 다른 스레드에서는 자원에 접근하려면 `block` 상태가 되기 때문에
- 싱글 스레드로 접근한다는 가정하에선 `StringBuilder` 와 `StringBuffer` 의 성능이 똑같을까요?
    - 답변) 싱글 스레드로 접근하는 경우에도 `StringBuilder` 가 `StringBuffer` 의 성능 보다 더 나을 것이라고 생각한다. 이유는 `StringBuffer` 에서는 동기화 기능을
      추가적으로 수행하기 때문에 더 느릴 수 밖에 없다. 
