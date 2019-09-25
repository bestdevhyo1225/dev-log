## Promise

<br>

### :book: Promise가 뭔가요?

JavaScript나 TypeScript로 백엔드 서버를 개발하다 보면, 대부분 비동기 처리를 해야한다. 왜 비동기 처리를 해야할까? 비동기 처리를 하는 이유는 다음과 같다. 어떠한 특정 코드가 실행되고 나서 그 코드가 종료되기를 기다리지 않고, 다음 코드를 수행하기 때문에 전체적인 프로그램 수행시간을 단축할 수 있다. 그렇다면 Promise는 뭘까? **Promise는 JavaScript 비동기 처리에서 사용되는 객체이다.**

<br>

### :book: Promise는 왜 필요한가요?

Promise를 사용하기 이전에 비동기 처리는 대부분 Callback 함수를 사용해서 처리를 했다. 하지만 여러 함수를 비동기 처리해야 하는 과정에서 **Callback Hell**이라는 문제가 발생하게 되었다. Callback 함수를 여러 중첩으로 작성하면 코드의 가독성이 떨어지고 로직을 변경하는데 어려움이 있다.

* `Callback Hell` 예제

```javascript
function square(x, callback) {
    setTimeout(callback, 100, x * x);
}

square(2, function(x) {
    square(x, function(x2) {
        square(x2, function(x3) {
            console.log(x3);
        });
    });
});

// 결과 : 256
```

* `Promise`를 사용한 코드

```javascript
function square(x) {
    return new Promise((resolve, reject) => resolve(x * x))
}

square(2)
.then(x2 => { return x2 * x2; })
.then(x3 => console.log(x3 * x3));

// 결과 : 256
```

<br>

### :book: Promise는 어떤 방식으로 동작되나요?

