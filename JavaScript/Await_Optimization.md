## Await 및 Promise.all()을 적절하게 사용하여 수행시간을 단축하자

<br>

- async / await에 관한 좋은 글을 읽게 되어 나름대로 정리하려고 한다.
- 최대한 참고 문서를 보면서 다시 공부하고 정리할 것

<br>

### :book: 함수간의 의존관계를 파악해서 await 사용하기

- 아래의 코드는 비동기 함수들을 간단히 구현했다.

```javascript
const work = (resolve, data, sec) => setTimeout(() => resolve(data), sec);

const readProducts = () => {
  return new Promise((resolve) => work(resolve, "products", 1000));
};

const readCounpons = () => {
  return new Promise((resolve) => work(resolve, "coupons", 500));
};

const readKey = () => {
  return new Promise((resolve) => work(resolve, "key", 300));
};

const checkValidKey = (key) => {
  return new Promise((resolve) => work(resolve, key ? true : false, 900));
};
```

<br>

- main1 함수는 위의 코드를 호출하여 사용하는 부분이다.
- readProducts(), readCounpons(), readKey() 함수는 독립적이다. 즉, 어떤것을 먼저 호출해도 각자에게 영향을 주지 않는다.
- readKey(), checkValidKey(key) 함수의 관계를 따져보면, checkValidKey(key) 함수는 readKey()가 반드시 호출되어야 사용할 수 있는 함수이므로 의존적인 관계를 가지고 있다.
- 총 `2,700ms` 수행 시간이 걸린다.

```javascript
const main1 = async () => {
  const startTime = new Date().getTime();

  const products = await readProducts(); // 1,000ms
  const coupons = await readCounpons(); // 500ms
  const key = await readKey(); // 300ms
  const result = await checkValidKey(key); // 900ms

  const endTime = new Date().getTime();

  console.log("main1 수행 시간(ms) : ", endTime - startTime);

  if (result) console.log("end");
};
```

<br>

- 위의 코드를 독립적인 관계, 의존적인 관계로 나눠서 수행 시간을 줄일 수 있다.
- 독립적인 관계를 가지는 readProducts(), readCounpons(), readKey() 함수들은 Promise.all 함수를 사용하여 병렬적으로 처리한다.
- 그리고 나서 checkValidKey(key) 함수를 호출한다.
- 총 수행시간은 `1,900ms`로 위의 코드보다 `800ms`의 수행속도를 줄였다.

```javascript
const main2 = async () => {
  const startTime = new Date().getTime();

  const [products, coupons, key] = await Promise.all([readProducts(), readCounpons(), readKey()]); // 1,000ms -> readProducts() 함수 수행시간이 1,000ms 로 3개 함수 중 가장 오래 걸린다.
  const result = await checkValidKey(key); // 900ms

  const endTime = new Date().getTime();

  console.log("main2 수행 시간(ms) : ", endTime - startTime);

  if (result) console.log("end");
};
```

<br>

- Promise.all 함수가 비동기 함수들을 병렬적으로 처리해주는 특성을 살려 수행시간을 더 단축할 수 있다.
- 총 수행시간은 `1,200ms`이 된다.

```javascript
const main3 = async () => {
  const startTime = new Date().getTime();

  const [products, coupons, result] = await Promise.all([
    readProducts(), // 1,000ms
    readCounpons(), // 500ms
    checkValidKey(await readKey()), // 1,200ms (900ms + 300ms) -> readKey()를 수행한 후, 바로 checkValidKey(key) 작업을 진행함
  ]); // checkValidKey(await readKey()) 함수 수행시간이 1,200ms로 3개 함수 중 가장 오래 걸린다.

  const endTime = new Date().getTime();

  console.log("main3 수행 시간(ms) : ", endTime - startTime);

  if (result) console.log("end");
};
```

<br>

### :book: 정리

- await를 여러번 사용할 때, 함수간의 관계를 잘 파악해야 한다. (독립적? 의존적?)
- await는 동기식으로 코드를 만들어주기 때문에 주의하자
- Promise.all을 사용하거나 await 코드를 적절하게 배치하면, 수행시간을 단축할 수 있다.
- 좀 더 자세한 내용은 참고된 문서를 보면서 공부하자

<br>

### :bookmark: 참고

- [await의 함정, 숨은 병목을 찾자](https://jaeheon.kr/161)
