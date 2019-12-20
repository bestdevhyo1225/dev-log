### Iterable / Iterator

<br>

### :book: Iterable / Iterator 프로토콜

* Iterable : Iterator를 리턴하는 `[Symbol.iterator]()` 를 가진 값

* Iterator : { value, done } 객체를 리턴하는 next() 를 가진 값

* Iterable / Iterator 프로토콜 : Iterable을 for...of, 전개 연산자등과 함께 동작하도록한 규약 

```javascript
// 사용자 정의 이터러블..
const iterable = {
    [Symbol.iterator]() {
        let i = 3;
        return {
            next() {
                return i === 0 ? { done: true } : { value: i--, done: false };
            },
            // iterator도 iterable로도 순회가 가능하게 하기 위함
            [Symbol.iterator]() { return this; }
        }
    }
}

let iterator = iterable[Symbol.iterator]();

console.log(iterator.next());   // { value: 3, done: false }
console.log(iterator.next());   // { value: 2, done: false }
console.log(iterator.next());   // { value: 1, done: false }
console.log(iterator.next());   // { done: true }

// for...of 로도 순회 가능 -> 이터레이블
for (const a of iterable) console.log(a);

// 이터레이터로도 순회가 가능
for (const a of iterator) console.log(a);
```

<br>

### :bookmark: 참고

* [인프런 - 함수형 프로그래밍과 JavaScript E6+](https://www.inflearn.com/course/functional-es6#)