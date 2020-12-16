# Pod 실습

> [시작하기 전에 Pod에 대해서 간단하게 알아보기](https://github.com/bestdevhyo1225/dev-log/blob/master/Kubernetes/Pod.md)

<br>

## Pod

:pushpin: Pod를 생성합니다.

```sh
# Pod 생성
$ kubectl apply -f pod-1.yml

# Pod 상태 확인
$ kubectl get pods

# Pod 세부 사항 보기
$ kubectl describe pod/pod-1
```

<br>

## ReplicationController

:pushpin: Pod를 만들어주고 Pod가 죽었을 때 다시 생성하는 관리적인 역할을 하는 ReplicationController 생성합니다.

```sh
# ReplicationController 생성
$ kubectl apply -f replication-1.yml
```

<br>

## Label 그리고 Service

:pushpin: 6개의 Pod를 생성합니다.

```sh
# pod-1
$ kubectl apply -f pod-dev-web.yml

# pod-2
$ kubectl apply -f pod-dev-db.yml

# pod-3
$ kubectl apply -f pod-dev-server.yml

# pod-4
$ kubectl apply -f pod-production-web.yml

# pod-5
$ kubectl apply -f pod-production-db.yml

# pod-6
$ kubectl apply -f pod-production-server.yml
```

:pushpin: type: web인 Pod을 위한 Service를 생성합니다.

```sh
# type: web인 Pod들이 여기에 묶인다. ex) pod-1, pod-4
$ kubectl apply -f service-for-web.yml
```

:pushpin: lo: production인 Pod을 위한 Service를 생성합니다.

```sh
# lo: production인 Pod들이 여기에 묶인다. ex) pod-4, pod-5, pod-6
$ kubectl apply -f service-for-production.yml
```

<br>

## 참고

- [Pod 강의 자료](https://kubetm.github.io/practice/beginner/object-pod/)
