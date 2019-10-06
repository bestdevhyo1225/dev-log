## CORS(Cross-Origin Resource Sharing)

<br>

### :book: CORS란??

Cross-Site Http Request를 가능하게 하는 표준 규약이다. HTTP 요청은 기본적으로 Cross-Site Http Request가 가능하지만, JavaScript로 다른 웹 페이지에 접근할 때는 `Same Origin Policy`로 인해 요청이 불가능하다.

<br>

### :book: Same Origin Policy는 뭔가요?

Same Origin Policy는 `프로토콜`, `호스트`, `포트`가 동일할 때 해당 웹 사이트에 접근할 수 있는 정책을 말한다.

<br>

### :book: 그렇다면 CORS를 사용하는 이유를 말해준다면?

최근에 REST API등의 외부 호출이 많아지고, 여러 도메인에 대규모로 구성되는 웹 프로젝트가 많아지면서 불편한 정책이 되었다. 그래서 웹 브라우저에서 외부 도메인 서버와 통신하기 위한 방식을 표준화 한 것이 `CORS(Cross-Origin Resource Sharing)`이다.

<br>

### :book: 클라이언트가 서버에 어떻게 요청하면 되나요?

클라이언트가 CORS 요청을 위해 HTTP 헤더에 CORS를 허용하는 정보를 추가하고, 서버는 클라이언트에서 전송한 헤더를 확인해서 허용할지 말지를 결정한다.

* `AWS S3(서버)에서 CORS를 설정하면 웹 브라우저에서 접근할 수 있도록 설정하는 예`

```html
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
    <AllowedOrigin>*</AllowedOrigin>
    <AllowedMethod>POST</AllowedMethod>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedMethod>DELETE</AllowedMethod>
    <AllowedMethod>HEAD</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```

* `Web Application Server에서 CORS를 허용하는 예`

```typescript
// 간단한 예제
import { Request, Response } from 'express';

export const show = async (req: Request, res: Response): Promise<void> => {
    res.header("Access-Control-Allow-Origin", "*");
}
```

<br>

### :bookmark: 참고

* [CORS란?](https://juicyjusung.github.io/2019/08/21/http/cors/)