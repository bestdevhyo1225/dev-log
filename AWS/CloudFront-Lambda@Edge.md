## AWS CloudFront & Lambda@Edge

<br>

### :book: CloudFront를 사용하는 이유?

예전에는 정적 콘텐츠(JS, CSS, HTML, 이미지, 비디오, 음악 파일)를 표준 서버에 저장하고 제공하는 형태였다. 이러한 구조는 서버에 많은 부담을 주는 문제가 있었다. 이러한 문제를 보완하기 위해서 사용자와 가까운 위치에서 정적 콘텐츠를 전송하기 위해서 CDN(Content Delivery Network)서비스인 `CloudFront`를 사용한다.

<br>

### :book: CloudFront는 어떤 방식으로 동작하나요?

`CloudFront`는 `EdgeLocation`이라는 전 세계 데이터 센터 네트워크를 통해 콘텐츠를 제공한다. 캐시 서버를 사용해서 콘텐츠를 캐시하고 사용자에게 제공한다. 만약에 CloudFront에 정적 콘텐츠가 캐시 되었다면 빠르게 사용자에게 제공하고, 캐시 되지 않았다면 오리진 서버(AWS S3)에 접속해서 콘텐츠를 가져와 캐시한다.

<br>

### :book: CloudFront의 주요 특징은 무엇인가요?

콘텐츠에 대한 엑세스를 제한할 수 있는 OAI(Origin Access Identity) 기능이 있다. 이 기능으로 인해 사용자가 직접 오리진 서버(S3)에 접근할 수 없고, `CloudFront` 및 해당 작업만으로 제한한다.

<br>

### :book: CloudFront의 Lambda@Edge란?

`Lambda@Edge`는 사용자와 더 가까운 위치에서 코드를 실행하여 성능을 개선하고 지연 시간을 단축할 수 있게 해주는 서비스이다. `Lambda@Edge`를 사용하면 전 세계에 있는 인프라를 프로비저닝하거나 관리하지 않아도 되는 장점이 있다.

<br>

### :book: Lambda@Edge는 어떤 방식으로 동작하나요?

`CloudFront`가 오리진 서버(S3)에 컨텐츠에 대한 요청을 한다. 그러면 오리진 서버(S3)는 요청에 대한 응답 이벤트를 `Lambda@Edge`에 보내고, `Lambda@Edge`는 코드를 실행하여 작업을 처리한다.