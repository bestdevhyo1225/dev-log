## 함수형 컴포넌트 - useEffect()로 간단한 Life Cycle 사용하기

<br>

### :book: 클래스 컴포넌트의 componentDidMount 처럼 동작하는 코드

```typescript
import React, { useEffect } from 'react';

interface FuncComponentProps {}

function FuncComponent(props: FuncComponentProps) {
  useEffect(() => {
    console.log('componentDidMount 처럼 동작한다.');
  }, []); // 두 번째 인자에 '[]' 넣어주면???

  return (
    <>
    </>
  );
}
```

useEffect 함수 두 번째 인자에 **`[]`** 를 넣어주면 클래스 컴포넌트의 Life Cycle인 `componentDidMount` 처럼 동작한다.

<br>

### :book: 클래스 컴포넌트의 componentDidUpdate 처럼 동작하는 코드

```typescript
import React, { useEffect, useState } from 'react';

interface FuncComponentProps {}

function FuncComponent(props: FuncComponentProps) {
  const [ name, setName ] = useState('');

  useEffect(() => {
    console.log('componentDidMount 처럼 동작한다.');
    console.log(name);
  }, [ name ]); // 특정 값이 업데이트 될 때만 실행하고 싶다면 [] 에 값을 넣어 주면 된다.

  return (
    <>
    </>
  );
}
```

<br>

### :book: 클래스 컴포넌트의 componentWillUnmount 처럼 동작하는 코드

```typescript
import React, { useEffect } from 'react';

interface FuncComponentProps {}

function FuncComponent(props: FuncComponentProps) {
  useEffect(() => {
    // ....
    return () => {
      console.log('componentWillUnmount 처럼 동작한다.');
    }
  }, []);

  return (
    <>
    </>
  );
}
```

### :book: 정리

함수형 컴포넌트 hooks인 useEffect 함수를 잘 활용하면 클래스 컴포넌트의 `componentDidMount`, `componentDidUpdate`, `componentWillUnmount` Life Cycle를 구현할 수 있다.