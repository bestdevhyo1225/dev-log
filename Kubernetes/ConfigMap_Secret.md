# ConfigMap, Secret

<br>

## :question: ConfigMap과 Secret이 사용되는 경우

dev, stage, production 환경마다 달라져야 하는 값들이 있다. 예를 들면, `dev`에서 `dev-db-url`을 설정하고 `stage`에서는 `stage-db-url`을 설정하고, `production`에서는 `production-db-url`을 설정해야 한다고 했을때 컨테이너를 3개 만들어야 한다. 이렇게 환경에 따라 변하는 값들이 있을때 사용한다.

<br>

## :book: ConfigMap

- 일반적인 상수들을 모아서 만든 Object이다.

- Key, Value 리스트를 무한히 넣을 수 있다.

- `ConfigMap`은 Pod 생성시, 컨테이너 Env에 들어간다.

<br>

## :book: Secret

- 보안적인 관리가 필요한 값들을 모아서 만든 Object이다.

- 1MB 까지만 넣을 수 있기 때문에 Secret값을 너무 많이 만들면 시스템 자원의 영향을 끼친다.

- `Secret`은 Pod 생성시, 컨테이너 Env에 들어간다.
