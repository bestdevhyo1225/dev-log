# Java 직렬화

`Spring Data Redis`를 사용했을때, 직렬화를 제대로 알고 있어야 한다는 것을 느끼게 되었고, 기술 블로그를 통해 간단히 테스트 해본 것을 정리했다.

<br>

## Java 진영에서 직렬화란 무엇인가?

- `직렬화`란 Java 시스템 내부에서 사용하는 `객체 또는 데이터`를 외부 Java 시스템에서도 사용할 수 있도록 `바이트 형태의 데이터`로 변환하는 기술이며, `역직렬화`란 `바이트로 변환된 데이터`를 다시 `객체`로 변환하는 기술이다.

- 시스템적으로 얘기하지면, JVM 메모리에 상주 되어 있는 `객체` 데이터를 `바이트`형태로 변환하는 기술과 `직렬화`된 `바이트`형태의 데이터를 다시 `객체`로 변환해서 JVM 메모리에 상주시키는 형태를 얘기한다.

<br>

## 직렬화 방법

> java.io.ObjectOutputStream 객체를 이용해서 직렬화를 할 수 있다.

```java
BookSerializeResult bookSerializeResult = BookSerializeResult.builder()
				.bookId(1L)
				.title("title")
				.author("author")
				.price(20000)
				.build();

byte[] serializedBook = null;
try (ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream()) {
	try (ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream)) {
		objectOutputStream.writeObject(bookSerializeResult);
		serializedBook = byteArrayOutputStream.toByteArray();
	} catch (IOException e) {
		e.printStackTrace();
	}
}

String encodedSerializedBook = Base64.getEncoder().encodeToString(serializedBook);
```

<br>

## 역직렬화 방법

> java.io.ObjectInputStream 객체를 통해 역직렬화를 할 수 있다.

```java
// 직렬화 된 문자열
String encodedSerializedBook = Base64.getEncoder().encodeToString(serializedBook);

// 역 직렬화
byte[] deserializedBook = Base64.getDecoder().decode(encodedSerializedBook);

try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(deserializedBook)) {
	try (ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream)) {
		Object book = objectInputStream.readObject();
		BookSerializeResult tempBookSerializeResult = (BookSerializeResult) book;
	} catch (IOException | ClassNotFoundException e) {
		e.printStackTrace();
	}
}
```

<br>

## 용량 문제

- `직렬화`할 경우에, 기본적으로 타입에 대한 정보, 클래스의 메타 정보를 가지고 있기 때문에 상대적으로 다른 포맷에 비해서 용량이 큰 문제가 있다.

- 특히 클래스의 구조가 거대해지면, 용량이 비대해진다.

- JSON과 같은 최소의 메타 정보만 가지고 있으면, 그냥 포맷된 데이터 보다 훨씬 적은 용량의 크기를 가질 수 있다.

> 그냥 포맷된 객체의 크기와 JSON 포맷의 크기를 비교해보자.

```java
String encodedSerializedBook = Base64.getEncoder().encodeToString(serializedBook);

// 역 직렬화
byte[] deserializedBook = Base64.getDecoder().decode(encodedSerializedBook);

// 그냥 포맷된 데이터의 크기
System.out.println("Deserialized Book Byte Size = " + deserializedBook.length);

ObjectMapper objectMapper = new ObjectMapper();

try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(deserializedBook)) {
	try (ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream)) {
		Object book = objectInputStream.readObject();
		BookSerializeResult tempBookSerializeResult = (BookSerializeResult) book;
		String strBook = objectMapper.writeValueAsString(tempBookSerializeResult);
		// JSON 형태로 변환했을때의 크기
    System.out.println("JSON Book Byte Size = " + strBook.getBytes(StandardCharsets.UTF_8).length);
	} catch (IOException | ClassNotFoundException e) {
		e.printStackTrace();
	}
}
```

> 아래의 결과를 확인해보면, 크기의 상당한 차이를 확인할 수 있다.

![image](https://user-images.githubusercontent.com/23515771/104421801-2ef73700-55bf-11eb-918b-3de13880ce89.png)

<br>

## 용량 문제 결론

- 일반 사용자를 대상으로 하는 `B2C`와 같은 시스템에서 Java 직렬화 정보를 캐시 서버에 저장하게 되면, 비 효율적인 문제를 가지고 있다.

  - 용량 크기에 따른 네트워크 비용과 캐시 서버 비용

- 새롭게 시작하는 서비스에서는 생산성을 위해서 Java 직렬화를 이용한다고 해도 괜찮다. 하지만 트래픽이 지속적으로 증가하는 상황이라면, JSON 형태나 다른 형태의 직렬화로 바꿔주는 것을 고려해야 한다.

<br>

## 참고

- [[우아한 형제들 기술 블로그]자바 직렬화, 그것이 알고싶다. 훑어보기편](https://woowabros.github.io/experience/2017/10/17/java-serialize.html)

- [[우아한 형제들 기술 블로그]자바 직렬화, 그것이 알고싶다. 실무편](https://woowabros.github.io/experience/2017/10/17/java-serialize2.html)
