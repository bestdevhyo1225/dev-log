# Service 실습

<br>

## Cluster IP 타입으로 Service 적용하기

> ClusterIP는 클러스터 내에서만 사용하는 IP이며, 주로 클러스터 내부를 접근할 수 있는 인가된 사용자(운영자)를 위해 사용한다.

- 내부 대쉬보드

- Pod의 서비스 상태 디버깅

```sh
# Pod 생성
$ kubectl apply -f pod-1.yml

# Pod 확인
$ kubectl get pods

# Cluster IP 타입의 Service 생성
$ kubectl apply -f service-cluster-ip.yml

# Service 확인
$ kubectl get service
```

> 쿠버네티스 클러스터 내에서는 Service IP에 접근할 수 있고, 외부에서는 접근할 수 없다.

```sh
# << minikube 클러스터 환경에서 테스트 >>

# minikube 클러스터 접속
$ minikube ssh

# curl 명령어로 호출해보면...
$ curl ${service ip}:${service port}/hostname

# 결과 출력
Hostname : pod-1
```

<br>

## Node Port 타입으로 Service 적용하기

> 내부망을 연결하는데 사용하거나 데모 및 임시 연결을 위해 사용한다.

```sh
# Node Port 타입의 서비스 시작
$ kubectl apply -f service-node-port.yml
```

> 쿠버네티스 클러스터의 연결되어 있는 모든 노드에 대해서 `동일한 포트`가 할당된다.

- Node 1 : 192.168.0.31:`30000` (Pod-1 있음)
- Node 2 : 192.168.0.32:`30000` (Pod-2 있음)
- Service IP : 10.101.147.131

> IP와 Port를 통해 접속하면 어떤 노드인지와 상관없이 Service에 바로 연결되지만, `externalTrafficPolicy: Local`을 설정하면 해당 노드의 Pod에만 접근할 수 있다.

- externalTrafficPolicy 설정을 하지 않음

```sh
# << 로컬 환경에서 테스트 >>

# 트래픽을 분산해서 각각의 Pod에 전달한다. (동일 IP, Port에 요청을 날렸을 때)
$ curl 192.168.0.31:30000
Hostname : pod-1

$ curl 192.168.0.31:30000
Hostname : pod-2

$ curl 192.168.0.31:30000
Hostname : pod-2

$ curl 192.168.0.31:30000
Hostname : pod-1

$ curl 192.168.0.31:30000
Hostname : pod-1

$ curl 192.168.0.31:30000
Hostname : pod-2

$ curl 192.168.0.31:30000
Hostname : pod-1
```

- externalTrafficPolicy: Local로 설정함

```sh
# << 로컬 환경에서 테스트 >>

# externalTrafficPolicy: Local인 상태에서는 해당 노드의 Pod에만 접근한다.
$ curl 192.168.0.31:30000
Hostname : pod-1

$ curl 192.168.0.31:30000
Hostname : pod-1

$ curl 192.168.0.32:30000
Hostname : pod-2

$ curl 192.168.0.32:30000
Hostname : pod-2
```

> 만약 pod-1이 없으면, `192.168.0.31:30000`로 요청을 날렸을 때 접근이 안된다.

```sh
$ curl 192.168.0.31:30000/hostname
# 응답 없음...
```

<br>

## Load Balancer 타입으로 Service 적용하기

> Load Balancer 타입은 외부 시스템 노출용이며, 외부를 노출시켜줄 플러그인을 설치해서 사용하면 된다. GCP, AWS 등등 클라우드 서비스에서 제공하는 쿠버네티스 플랫폼을 사용할 때는 자체적으로 플러그인 설치가 되어 있다.

<br>

## 참고

- [Service 강의 자료](https://kubetm.github.io/practice/beginner/object-service/)
