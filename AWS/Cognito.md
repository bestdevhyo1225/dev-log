## AWS Cognito

<br>

### :book: Cognito란 무엇인가요?

`Cognito` 서비스는 웹 또는 모바일 앱에 대한 인증 그리고 권한 부여하고, 사용자 관리를 제공하는 서비스이다. `Cognito`는 2가지 구성 요소를 가지고 있는데 **사용자 풀**과 **자격 증명 풀**이 있다.

<br>

### :book: 사용자 풀은 뭔가요?

사용자 풀은 가입이나 로그인 옵션을 제공하는 사용자 디렉토리이다. 사용자는 사용자 풀을 통해 직접 로그인 하거나 타사 자격 증명 공급자(IdP)를 통해 연동 로그인을 할 수 있다. 사용자 풀에서는 소셜 로그인에서 반환된 토큰과 OpenID Connect(OIDC) 및 SAML IdP에서 반환된 토큰의 처리 작업을 관리한다. 마지막으로 사용자 풀은 모든 멤버의 디렉토리 프로필을 보유하고 있고, SDK를 통해 이를 액세스 할 수 있다.

<br>

### :book: 자격 증명 풀은 뭔가요?

자격 증명 풀은 AWS 자격 증명을 제공하여 기타 AWS 서비스에 대한 사용자 접근 권한을 부여하는 구성요소이다.

<br>

### :book: 여러가지 Cognito의 시나리오가 있는데 말해주세요

**`1. 사용자 풀을 통한 인증`**

**`2. 사용자 풀을 통한 서버 측 자원 접근`**

**`3. 사용자 풀을 통해 API Gateway 및 Lambda에서 자원 접근`**

**`4. 사용자 풀 및 자격 증명 풀을 사용하여 AWS 서비스 접근`**

4번 시나리오 사용해보고 나서 정리할 것

**`5. 타사를 통한 인증 및 자격 증명 풀을 통한 AWS 서비스 접근`**

**`6. Amazon Cognito를 사용한 AWS AppSync 자원 접근`**

<br>

### :bookmark: 참고

* [일반적인 Amazon Cognito 시나리오](https://docs.aws.amazon.com/ko_kr/cognito/latest/developerguide/cognito-scenarios.html)