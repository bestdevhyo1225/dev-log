# Service 실습

<br>

## Cluster IP 타입으로 Service 적용하기

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

:pushpin: 쿠버네티스 클러스터 내에서는 Service IP에 접근할 수 있고, 외부에서는 접근할 수 없다.

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

```sh
$ kubectl apply -f service-node-port.yml
```

:pushpin: 쿠버네티스 클러스터의 연결되어 있는 모든 노드에 대해서 `동일한 포트`가 할당된다.

- Node 1 : 192.168.0.31:`30000` (Pod-1 있음)
- Node 2 : 192.168.0.32:`30000` (Pod-2 있음)
- Service IP : 10.101.147.131

:pushpin: IP와 Port를 통해 접속하면 어떤 노드인지와 상관없이 Service에 바로 연결된다.

:pushpin: 하지만 `externalTrafficPolicy: Local`을 설정하면 해당 노드의 Pod에만 접근한다.

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

:pushpin: 만약 pod-1이 없으면, `192.168.0.31:30000`로 요청을 날렸을 때 접근이 안된다.

```sh
$ curl 192.168.0.31:30000/hostname
# 응답 없음...
```

<br>

## LoadBalancer 타입으로 Service 적용하기

:pushpin: 로드 밸런서타입은 외부를 노출시켜줄 플러그인을 설치해서 사용하면 된다.

<br>

## 참고

- [Service 강의 자료](https://kubetm.github.io/practice/beginner/object-service/)
