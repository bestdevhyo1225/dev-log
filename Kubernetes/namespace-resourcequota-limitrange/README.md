# Namespace, ResourceQuota, LimitRange

<br>

## Namespace

![image](https://kubetm.github.io/img/practice/beginner/How%20to%20use%20Namespace%20for%20Kubernetes.jpg)

하나의 `Namespace` 안에서는 `Pod`의 이름을 중복해서 만들 수 없으며, `Namespace`를 삭제하면, 그 안에 있는 자원들이 모두 삭제 된다. 만약 `Namespace1`에 `Serivce`가 있고, `Namespace2`에 `Pod`가 있는 상황에서, 같은 `Label(app:pod)`을 명시해도 `Namespace1`에 있는 `Service`는 `Namespace2`에 있는 `Pod`에 접근할 수 없다.

<br>

## Namespace 실습

> 오브젝트의 연결은 같은 `Namespace` 안에서만 되는지 확인

```zsh
# namespace 생성
$ kubectl apply -f namespace.1-1.yml

# pod 생성
$ kubectl apply -f pod.1-2.yml

# service 생성
$ kubectl apply -f service.1-3.yml
```

> 생성한 `Pod`와 `Service`는 같은 `Namespace`에 있다. 직접 확인해 보려면 클러스터에 접속해서 호출을 해보면 된다.

```zsh
$ minikube ssh

# minkube 클러스터 환경...
# service ip로 요청
$ curl 10.102.25.224:9000/hostname
# 응답
Hostname : pod-1
```

<br>

## ResourceQuota

![image](https://kubetm.github.io/img/practice/beginner/How%20to%20use%20ResourceQuota%20for%20Kubernetes.jpg)

`ResourceQuota`는 `Namespace`의 자원 한계를 설정하는 오브젝트이다. 또한 `Pod`를 생성할 때 스펙을 꼭 명시해야 한다. 만약 `Namespace`에 `requests.memory`를 `1Gi`로 설정하고, `Pod1`에서 `requests.memory`를 `0.5Gi`를 사용하면, 남은 자원은 `0.5Gi`이다. 이때 `Pod2`를 추가하고, `requests.memory`를 `0.7Gi`를 사용한다고 명시하면, `Pod2`는 생성이 되지 않는다. 생성이 되지 않는 이유는 자원이 `0.5Gi` 남아 있기 때문이다.

- Compute Resource : cpu, memory, storage

- Objects count : Pod, Service, ConfigMap, PVC, ...

```yml
# Pod
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-3 # 할당할 Namespace 설정
spec:
  containers:
    - name: container
      image: kubetm/app
      resources:
        requests:
          memory: 0.5Gi
        limits:
          memory: 0.5Gi
```

```yml
# ResourceQuota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-1
  namespace: nm-3
spec:
  hard:
    requests.memory: 1Gi
    limits.memory: 1Gi
```

<br>

## ResourceQuota 실습

> `Namespace`를 생성하고, `ResourceQuota`를 생성한다.

```zsh
$ kubectl apply -f namespace.2-1.yml

$ kubectl apply -f resouce-quota.2-2.yml
```

> `ResouceQuota`의 정보는 아래의 명령어를 통해 확인한다.

```zsh
$ kubectl describe resoucequotas --namespace=nm-3

Name:            rq-1
Namespace:       nm-3
Resource         Used  Hard
--------         ----  ----
limits.memory    0     1Gi
requests.memory  0     1Gi

```

> `ResourceQuota`가 적용된 `Namespace`에 `Pod`를 생성한다.

- 주의할 점, yml에 resouces 부분을 반드시 명시해야 한다.

> 생성된 pod-3으로 인해 현재 `Namespace`에 request.memory와 limits.memory의 남은 자원은 0.5Gi이다. 만약에 이보다 큰 값을 설정한다면, 배포를 실패하게 된다. 또한, hard 옵션에 pod의 갯수를 제한할 수도 있다.

```zsh
$ kubectl -f apply pod.2-3.yml
```

> 하나의 `Namespace`를 생성하고, `Pod`를 여러 개 생성한다.

```zsh
# nm-4의 이름을 가진 Namespace
$ kubectl -f apply namespace-4.yml

# Namspace에 Pod 1 생성
$ kubectl -f apply pod-1.yml

# Namspace에 Pod 2 생성
$ kubectl -f apply pod-2.yml
```

> 이렇게 생성한 다음, `ResouceQuota`를 생성하고, 다시 `Pod`를 생성하면, 현재 `Namespace`는 `ResouceQuota`에 명시한 memory보다 더 많은 자원을 사용하고 있게 된다.

```zsh
# ResouceQuota를 생성...
$ kubectl -f apply resouce-quota-4.yml

# 이 상황에서 Pod를 만든다면, 현재 Namespace에는 지정한 메모리보다 더 많이 사용하게 된다.
$ kubectl -f apply new-pod.yml
```

> 따라서 `Namspace`를 생성하고 바로 `ResourceQuota`를 생성해야 자원 사용에 문제가 없다.

<br>

## LimitRange

![image](https://kubetm.github.io/img/practice/beginner/How%20to%20use%20LimitRange1%20for%20Kubernetes.jpg)

`LimitRange`에서 `maxLimitRequestRatio`는 `requests`와 `limit`의 비율이 최대 3배를 넘으면 안된다는 뜻이다.

<br>

## LimitRange 실습

> `Namespace`를 생성하고, `LimitRange`를 생성한다.

```zsh
$ kubectl apply -f namespace.3-1.yml

$ kubectl apply -f limit-range3-2.yml
```

> `LimitRange`도 대시보드에서 확인할 수 없기 때문에 명령어로 확인해야 한다.

```zsh
$ kubectl describe limitranges --namespace=nm-3
```

> `Pod`를 생성한다. (requests, limits를 정의하지 않으면, default값을 가진채로 생성이 된다.)

```zsh
# requests.memory: 0.2Gi, limits.memory: 0.4Gi, maxLimitRequestRatio가 2라서 생성이 된다.
$ kubectl apply -f pod.3-3.yml
```

> `Namespace`에는 여러 개의 `LimitRange`를 만들 수 있다. 그러나 여러 개의 `LimitRange`를 지정하게 되면, 예상치 못한 동작에 의해서 `Pod`가 생성되지 않을 수 있다.

```zsh
$ kubectl apply -f namespace.4-1.yml

$ kubectl apply -f limit-range-exception-1.yml

$ kubectl apply -f limit-range-exception-2.yml
```

<br>

## 참고

- [Namespace, ResourceQuota, LimitRange 강의 자료](https://kubetm.github.io/practice/beginner/object-namespace_resourcequota_limitrange/)
