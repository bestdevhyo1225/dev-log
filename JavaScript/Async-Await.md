## Async / Await

<br>

### :book: async / await는 뭔가요?

JavaScript에는 기존에 비동기 처리를 위한 패턴이 있다. Callback 함수를 사용하거나 Promise 객체를 사용해서 비동기 처리를 할 수 있다. 하지만, 두 패턴의 문제점을 보완하고자 최근에 나온 비동기 처리 패턴이며 개발자가 읽기 좋은 코드를 작성할 수 있게 도와준다.

<br>

### :book: 개발자가 읽기 좋은 코드??

사실 개인적인 생각으로 async / await 비동기 처리 패턴이 무조건 좋다고 얘기할 수는 없다. Callback 패턴이 익숙한 사람이거나 Promise 패턴에 익숙한 사람이라면 해당 패턴에 코드를 잘 읽을 수 있다. 그러나 혼자 코드를 잘 읽을 수 있는 것도 좋지만, 팀 작업과 누군가가 내 코드를 리뷰하는 상황이라면 남을 배려해야 한다고 생각한다. 나는 프로그래밍을 처음 배울 때 절차 지향적인 방법으로 시작했다. 그래서 순서대로 읽는 것에 익숙하다. 대부분이 그러하리라 생각한다. async / await 패턴이 절차 지향적인 방법으로 비동기를 처리하기 때문에 코드를 읽기가 쉬운 편이다.

<br>

### :book: async / await 기본적인 문법

```javascript
async function 함수명() {
    await 비동기_처리_함수명();
}
```

<br>

### :book: async / await 기본적인 예제 1

* `callback 패턴 처리 예시`

```typescript
interface Article {
    id: string;
}

function logArticle(): void {
    fetchArticle('domain.com/api/article/1', (article: Article): void => {
        if (article.id === 1) {
            console.log(article.title);
        }
    });
}
```

* `async / await 패턴 처리 예시`

```typescript
interface Article {
    id: string;
}

async function logArticle(): Promise<void> {
    const article: Article = await fetchArticle('domain.com/api/article/1');
    if (article.id === 1) {
        console.log(article.title);
    }
}
```

<br>

### :book: async / await 기본적인 예제 2

```typescript
function fetchItems(): Promise<Array<number>> {
    return new Promise((resolve: Function, reject: Function) => {
        const items = [1, 2, 3, 4, 5];
        resolve(items);
    });
}

async function logItems(): Promise<void> {
    const items: Array<number> = await fetchItems();
    if (items) {
        console.log(items);
    }
}
```

<br>

### :book: async / await 실용 예제

```typescript
interface User {
    id: number;
}

interface Todo {
    title: string;
}

// fetch 예시 1
function fetchUser(): User {
    const url: string = 'domain.com/api/user/1';
    return fetch(url).then((res: Response) => {
        return res.json();
    });
}

// fetch 예시 2
function fetchTodo(): Todo {
    const url: string = 'domain.com/api/todo/1';
    return fetch(url).then((res: Response) => {
        return res.json();
    });
}

async function logTodoTitle(): Promise<void> {
    const user: User = await fetchUser();
    if (user && user.id === 1}) {
        const todo: Todo = await fetchTodo();
        console.log(todo.title);
    }
}
```

<br>

### :book: async / await 예외 처리 방법은??

Promise 패턴에서는 then, catch를 사용하여 예외 처리 했지만, `async / await`에서는 `try-catch` 문법을 사용해서 비동기 예외 처리를 한다.

```typescript
interface User {
    id: number;
}

interface Todo {
    title: string;
}

// fetch 예시 1
function fetchUser(): User {
    const url: string = 'domain.com/api/user/1';
    return fetch(url).then((res: Response) => {
        return res.json();
    });
}

// fetch 예시 2
function fetchTodo(): Todo {
    const url: string = 'domain.com/api/todo/1';
    return fetch(url).then((res: Response) => {
        return res.json();
    });
}

// try - catch 문법을 사용하여 비동기 예외 처리..
async function logTodoTitle(): Promise<void> {
    try {
        const user: User = await fetchUser();
        if (user && user.id === 1}) {
            const todo: Todo = await fetchTodo();
            console.log(todo.title);
        }
    } catch (error) {
        console.error(error);
    }
}
```

<br>

### :bookmark: 참고

* [자바스크립트 async와 await](https://joshua1988.github.io/web-development/javascript/js-async-await/)