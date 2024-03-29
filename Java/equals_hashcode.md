# equals, hashcode 정리

## equals

- `동등성` 비교를 한다. 즉, `객체의 내부 값` 을 비교한다.
- `동일성` 비교는 `==` 연산을 통해 `객체 인스턴스의 주소 값` 을 비교한다.
    - primitive data 타입의 경우 `==` 을 통해 값 비교가 가능하다는 점에 있어서 예외적인 측면이 있다.

## hashcode

- 객체를 식별하기 위한 하나의 정수 값을 말한다.
- 객체의 `메모리 번지(주소)를 이용해서 해시 코드를 만들어 반환 한다.` 객체마다 다른 해시 코드 값을 가지고 있다.

## equals, hashcode 처리 과정

![img.png](img.png)

가장 먼저 `hashcode` 값이 다르면 `다른 객체` 로 판단하고, `hashcode` 값이 같으면 `equals` 로 다시 비교하는데, 객체의 내부 값이 같으면 `동등 객체` 로 판단하고, 객체 내부 값이
다르면 `다른 객체` 로 판단한다.

- 중요한 점은 `hashcode` 값이 다른 객체의 경우 동등성 비교 조차도 하지 않는다.

## Java HashTable

### 동작 원리

해시 함수를 이용해서 `Key` 를 기준으로 고유 식별값인 해시 값을 만들고, 생성된 해시 값을 `Bucket` 에 저장한다. (`hashcode` 가 해시 값 만드는 역할을 한다.)

- `HashTable` 의 경우 크기가 한정적이기 때문에 `서로 다른 객체라 하더라도 같은 해시 값을 갖게 될 수도 있다.` 이것을 **`해시 충돌(Hash Collisions)`** 라고 한다.
- **`해시 충돌(Hash Collisions)`** 이 일어나는 경우, 해당 `Bucket` 에 `LinkedList` 형태로 객체를 추가한다.
    - 참고) Java 8 또는 9버전 부터는 `LinkedList` 아이템의 갯수가 8개 이상으로 넘어가면, `TreeMap` 자료구조로 저장된다고 한다.

위의 `HashTable` 원리 처럼 같은 해시 값 안에 `다른 객체` 가 있는 경우 `equals` 가 사용되는 것이다. 따라서 `HashTable` 에 `put` 으로 객체를 추가하는 경우에 아래와 같은 기준으로
처리된다.

- 값이 `같은 객체` 가 이미 있는 경우(`equals 값 true`), 기존 객체를 덮어쓴다.
- 값이 `같은 객체` 가 없는 경우(`equals 값 false`), 해당 객체를 `LinkedList` 에 추가한다.

반대로 `get` 을 통해 조회하는 경우 아래와 같은 기준으로 처리된다.

- 값이 `같은 객체` 가 이미 있는 경우(`equals 값 true`), 해당 객체를 반환한다.
- 값이 `같은 객체` 가 없는 경우(`equals 값 false`), null 값을 반환한다.

## 예상 면접 질문

- 자바의 모든 클래스는 Object 클래스를 상속받습니다. 그리고 Object 클래스에는 `equals()` 와 `hashCode()` 라는 메소드가 선언되어 있습니다. 이 메소드들은 각각 어떤 역할일까요? 이
  둘의 차이점은 무엇일까요?
- `hashCode` 를 잘못 오버라이딩하면 `HashMap` 등 hash 콜렉션의 성능이 떨어질 수가 있습니다. 어떤 케이스일 때 그럴 수 있을까요?
    - 답변) 해시 충돌에 의해서 하나의 `Bucket` 에 객체들이 `LinkedList` 로 추가 되면, 객체를 추가 및 조회하는데 있어서 성능이 느리다. 최악으로는 하나의 `Bucket` 에서 끝까지
      탐색한 후, 객체를 추가하거나 조회하는 케이스가 발생할 수 있다.

## JPA Buddy (참고)

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || Hibernate.getClass(this) != Hibernate.getClass(o))
        return false;
    Item item = (Item) o;
    return Objects.equals(id, item.id);
}

@Override
public int hashCode() {
    return id.intValue();
}
```

- Java 코드에서 JPA Entity 클래스의 equals, hashcode 재정의시, 위의 형식대로 진행하면 될 듯
