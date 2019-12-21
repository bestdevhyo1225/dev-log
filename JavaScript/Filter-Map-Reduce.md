## Filter, Map, Reduce

### :book: filter(callback)

* 배열에 조건을 만들고, **조건에 만족하지 못하는 요소**들을 걸러낸다. 

* 즉 **조건에 만족하는 return 이 true 인 요소**만 모아서 새로운 배열을 만든다.

```javascript
const tenUpFilter = [11, 3, 9, 130, 44].filter( (element) => { return element > 10; } );
console.log(tenUpFilter);   // 11, 130, 44

const arr1 = [4, 15, 377, 395, 400, 1024, 3000];
const arr2 = arr1.filter( (element) => { return element % 5 === 0; } );
console.log(arr2); // [ 15, 395, 4000, 3000 ]
```

<br>

### :book: map(callback)

* 어떠한 배열에 **특정 규칙을 적용시켜 새로운 배열**을 만든다.

* **요소를 일괄적으로 변경하는데 효과적**

```javascript
// ex1
const arr1 = ['foo', 'hello', 'diamond', 'A'];
const arr2 = arr1.map( (str) => { return str.length; } );
console.log(arr2); // [3, 5, 7, 1];

// map 은 콜백 함수의 리턴을 모아서 새로운 배열을 만드는것이 목적이다.
// arr1 배열의 문자열들을 arr2 배열에서 문자열의 길이로 리턴 된 새로운 배열이 생성 된다.

// ex2
const arr3 = [1.1, 2.1, 3.1, 4.1, 5.1, 6.1, 7.1, 8.1, 9.1, 10.1];
const arr4 = arr3.map( (element) => { return element * 10; } );
console.log(arr4); // [ 11, 21, 31, 41, 51, 61, 71, 81, 91, 101 ]

const arr1 = [4, 15, 377, 395, 400, 1024, 3000];
// arr1 배열에 있는 요소 중에서
// 1. filter 를 통해서 5의 배수의 조건을 만족하는 요소들을 찾아서 배열을 만들고,
// 2. map 을 이용해서 걸러진 요소들에 2씩 곱하여 새로운 배열을 만든다.
const arr2 = arr1.filter( (element) => { return element % 5 === 0; } )
                    .map( (element) => { return element * 2; } );

console.log(arr2);
```

<br>

### :book: reduce(callback)

* reduce 메소드는 **filter, find, map 메소드를 모두 대체할 수 있는 아주 유연한 메소드**이다.
* filter, find, map 으로 구현할 수 있는 문제라면 reduce 로도 구현할 수 있다.
* 상황에 따라 적절한 메소드를 사용하는 것이 가독성 측면에서 유리하기 때문에 적절하게 사용하는 것을 권장 한다.

* **reduce 의 return 이 중요하다.** why ? 배열이 될 수 도..요소 하나의 값이 될 수 도.. 사용자가 원하는 값 뭐든지 될 수 있다.

* [ e1, e2, e3, … ].reduce(callback[, initialValue])

  `callback`

  1. **previousValue** : 이전 마지막 콜백 호출에서 반환된 값 또는 공급된 경우 initialValue

  2. **currentValue**: 배열 내에서 현재 처리되고 있는 요소 (element)

  3. **currentIndex** : 배열 내 현재 처리되고 있는 요소의 인덱스

  4. **[ e1, e2, e3, … ]** : reduce 에 호출 되는 배열

  `intialValue`

  1. **선택 사항**이며, **callback 의 첫 호출에 첫 번째 인수**로 사용하는 값

```javascript
const arr1 = [9, 2, 8, 5, 7];
const sum = arr1.reduce( (preValue, curValue) => { return preValue + curValue; } );
console.log(sum);   // 31
```

| 호출 순서 | preValue | curValue |    return     |
| :-------: | :------: | :------: | :-----------: |
|     1     |    9     |    2     |  11 ( 9 + 2)  |
|     2     |    11    |    8     | 19 ( 11 + 8 ) |
|     3     |    19    |    5     | 24 ( 19 + 5 ) |
|     4     |    24    |    7     | 31 ( 24 + 7 ) |

* 호출이 4번 된 이유 ??? -> initialValue 를 지정하지 않았기 때문에 preValue 는 9 로 지정 된다.

```javascript
const arr1 = [9, 2, 8, 5, 7];
let count = 0;
const sum = arr1.reduce( (preValue, curValue) => {
    ++count; 
    return preValue + curValue; 
}, 0 );
console.log(sum);   // 31
console.log(count); // 5
```

* 같은 결과지만 호출이 5번 된 이유는 intialValue 를 0 으로 지정해주었기 때문에 preValue 가 0 으로 설정된다.

> ### map, filter 로 구현한 코드 reduce 로 구현하기

```javascript
// map - 문자열 길이 구하기
// reduce 사용
const arr1 = ['foo', 'hello', 'diamond', 'A'];
const arr2 = arr1.reduce( (preValue, curValue) => {
    preValue.push(curValue.length);
    return preValue;
}, [] );
console.log(arr2);

// filter - 5의 배수인 정수만 모으기
const arr1 = [4, 15, 377, 395, 400, 1024, 3000];
const arr2 = arr1.reduce( (preValue, curValue) => {
    if (curValue % 5 === 0) preValue.push(curValue);
    return preValue;
}, [] );
console.log(arr2);
```

> ### Object.keys

* Object.keys 메소드는 Object 의 property 를 배열로 만들어 줍니다.

```javascript
const obj = {
  apple: 500,
  grape: 2000,
  berry: 30,
}

// Object.keys 는
// ['apple', 'grape', 'berry']
// 다음과 같은 배열의 형태로 만들어 준다.
```

* 한 가지 문제점이 있는데, 사이트 이펙트를 제거하지 못하는 문제점이 있다. 아래의 코드를 보자

```javascript
const obj = {
  apple: 500,
  grape: 2000,
  berry: 30,
}

const sum = Object.keys(obj).reduce( (preValue, curValue) => {
	return preValue + obj[curValue]; // 콜백 함수 내에서 obj 에 접근해야 한다.
}, 0 );
```

<br>

### :bookmark: 참고 사이트

* [ 자바 스크립트의 유용한 배열 메소드 사용하기 .. map(), filter(), find(), reduce() ](