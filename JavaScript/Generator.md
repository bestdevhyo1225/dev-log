## Generator란 무엇인가?

<br>

### :book: Generator란?

제너레이터란 사용자의 요구에 따라 시간을 순서대로 작업하여 여러 결과를 반환할 수 있게 도와주는 기능이다.

* 제너레이터는 이터레이터자 이터레이블을 생성하는 함수 (즉, 이터레이터를 리턴하는 함수이다.)

* 제너레이터를 실행한 결과는 이터레이터이다.

<br>

### :book: Generator Function이란?

Generator Function을 통해 작업을 중간에 멈췄다가 필요한 시점에 다시 재개할 수 있도록 도와주는 함수이다.

**`yield`**

* 제너레이터 함수의 실행을 일시적으로 정지 시킨다.

* 기억해둔 위치로 제어권을 넘겨주는 역할을 한다. (즉, 호출된 부분으로 값을 반환한다.)

**`next`**

* 함수가 호출될 때마다 호출되는 곳의 위치를 기억해둔 채로 실행된다.

```javascript
function *gen() {
    yield 1;
    yield 2;
    yield 3;
    return 100;
}

let iter = gen();

iter.next();    // { value: 1, done: false }
iter.next();    // { value: 2, done: false }
iter.next();    // { value: 3, done: false }
iter.next();    // { value: 100, done: true }

// return값은 없이 순회가 이루어짐
for (const a of gen()) console.log(a); // 1, 2, 3
```

* 사용자의 요구에 따라 조건을 지정하고, 이에 따라 순회하는 예제

```javascript
function *gen() {
    yield 1;
    if (false) yield 2; // 2는 무시
    yield 3;
}

let iter = gen();

iter.next(); // { value: 1, done: false }
iter.next(); // { value: 3, done: false }
iter.next(); // { value: undefined, done: true }

for (const a of gen()) console.log(a); // 1, 3
```

* 홀수만 발생시키는 이터레이터를 순회하는 예제

```javascript
function *infinity(i = 0) {
    while (true) yield i++;
}

function *limit(l, iter) {
    for (const a of iter) {
        yield a;
        if (a == l) return;
    }
}

function *odd(l) {
    for (const a of limit(l, infinity(1))) {
        if (a % 2) yield a;
    }
}

let iter = odd(10);

iter.next(); // { value: 1, done: false }
iter.next(); // { value: 3, done: false } 
iter.next(); // { value: 5, done: false }
iter.next(); // { value: 7, done: false }
iter.next(); // { value: 9, done: false }
iter.next(); // { value: undefined, done: true }

for (const a of odd(40)) console.log(a); // 40까지 홀수만 출력

```

<br>

### :bookmark: 참고

* [인프런 - 함수형 프로그래밍과 JavaScript E6+](https://www.inflearn.com/course/functional-es6#)