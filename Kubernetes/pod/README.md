# Pod 실습

`Pod`는 쿠버네티스에서 가장 기본적인 배포 단위이며, 컨테이너를 포함하는 단위이다. 쿠버네티스는 하나의 컨테이너를 개별적으로 배포하는 것이 아니라 `Pod` 단위로 배포하는데, `Pod`는 하나 이상의 컨테이너를 포함한다.

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

## Node Schedule

> 직접 선택

```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  nodeSelector: # 직접 선택한다.
    kubernetes.io/hostname: k8s-node1
  containers:
    - name: container
      image: kubetm/init
```

> 스케줄러 판단

```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4
spec:
  containers:
    - name: container
      image: kubetm/init
      resources: # 자원의 사용량에 따라서 스케줄이 판단한다.
        requests:
          memory: 2Gi
        limits:
          memory: 3Gi
```

<br>

## 참고

- [Pod 강의 자료](https://kubetm.github.io/practice/beginner/object-pod/)
