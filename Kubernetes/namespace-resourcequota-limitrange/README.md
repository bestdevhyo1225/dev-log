# Namespace, ResourceQuota, LimitRange

[시작하기 전에 Namespace, ResourceQuota, LimitRange에 대해서 간단하게 알아보기](https://github.com/bestdevhyo1225/dev-log/blob/master/Kubernetes/Namespace_ResourceQuota_LimitRange.md)

<br>

## Namespace 실습

오브젝트의 연결은 같은 `Namespace` 안에서만 되는지 확인

```zsh
# namespace 생성
$ kubectl apply -f namespace.1-1.yml

# pod 생성
$ kubectl apply -f pod.1-2.yml

# service 생성
$ kubectl apply -f service.1-3.yml
```

생성한 `Pod`와 `Service`는 같은 `Namespace`에 있다. 직접 확인해 보려면 클러스터에 접속해서 호출을 해보면 된다.

```zsh
$ minikube ssh

# minkube 클러스터 환경...
# service ip로 요청
$ curl 10.102.25.224:9000/hostname
# 응답
Hostname : pod-1
```

<br>

## ResourceQuota 실습

`Namespace`를 생성하고, `ResourceQuota`를 생성합니다.

```zsh
$ kubectl apply -f namespace.2-1.yml

$ kubectl apply -f resouce-quota.2-2.yml
```

`ResouceQuota`의 정보는 아래의 명령어를 통해 확인합니다.

```zsh
$ kubectl describe resoucequotas --namespace=nm-3

Name:            rq-1
Namespace:       nm-3
Resource         Used  Hard
--------         ----  ----
limits.memory    0     1Gi
requests.memory  0     1Gi

```

`ResourceQuota`가 적용된 `Namespace`에 `Pod`를 생성합니다.

- 주의할 점, yml에 resouces 부분을 반드시 명시해야 합니다.

생성된 pod-3으로 인해 현재 `Namespace`에 request.memory와 limits.memory의 남은 자원은 0.5Gi입니다. 만약에 이보다 큰 값을 설정한다면, 배포를 실패하게 됩니다. 또한, hard 옵션에 pod의 갯수를 제한할 수도 있습니다.

```zsh
$ kubectl -f apply pod.2-3.yml
```

또 다른 주의할 점이 있습니다. 하나의 `Namespace`를 생성하고, `Pod`를 여러 개 생성합니다.

```zsh
# nm-4의 이름을 가진 Namespace
$ kubectl -f apply namespace-4.yml

# Namspace에 Pod 1 생성
$ kubectl -f apply pod-1.yml

# Namspace에 Pod 2 생성
$ kubectl -f apply pod-2.yml
```

이렇게 생성한 다음, `ResouceQuota`를 생성하고, 다시 `Pod`를 생성하면, 현재 `Namespace`는 `ResouceQuota`에 명시한 memory보다 더 많은 자원을 사용하고 있게 됩니다.

```zsh
# ResouceQuota를 생성...
$ kubectl -f apply resouce-quota-4.yml

# 이 상황에서 Pod를 만든다면, 현재 Namespace에는 지정한 메모리보다 더 많이 사용하게 된다.
$ kubectl -f apply new-pod.yml
```

따라서 `Namspace`를 생성하고 바로 `ResourceQuota`를 생성해야 자원 사용에 문제가 없습니다.

<br>

## LimitRange 실습

`Namespace`를 생성하고, `LimitRange`를 생성합니다.

```zsh
$ kubectl apply -f namespace.3-1.yml

$ kubectl apply -f limit-range3-2.yml
```

`LimitRange`도 대시보드에서 확인할 수 없기 때문에 명령어로 확인해야 합니다.

```zsh
$ kubectl describe limitranges --namespace=nm-3
```

`Pod`를 생성합니다. (requests, limits를 정의하지 않으면, default값을 가진채로 생성됩니다.)

```zsh
# requests.memory: 0.2Gi, limits.memory: 0.4Gi, maxLimitRequestRatio가 2라서 생성이 된다.
$ kubectl apply -f pod.3-3.yml
```

#### 주의 사항

`Namespace`에는 여러 개의 `LimitRange`를 만들 수 있습니다. 그러나 여러 개의 `LimitRange`를 지정하게 되면, 예상치 못한 동작에 의해서 `Pod`가 생성되지 않을 수 있다.

```zsh
$ kubectl apply -f namespace.4-1.yml

$ kubectl apply -f limit-range-exception-1.yml

$ kubectl apply -f limit-range-exception-2.yml
```

<br>

## 참고

- [Namespace, ResourceQuota, LimitRange 강의 자료](https://kubetm.github.io/practice/beginner/object-namespace_resourcequota_limitrange/)
