# Replication Controller, ReplicaSet

서비스를 관리하고, 운영하는데 도움을 준다. 4가지의 특징을 가지고 있는데 다음과 같다.

- Auto Healing

- Auto Scaling

- Software Update

- Job

<br>

## Auto Healing

만약 `Node`가 죽어서 `Pod`에 장애가 온 상황이면, 즉각적으로 인지하고 `Pod`를 새로운 `Node`에 만들어준다.

<br>

## Auto Scaling

`Pod`의 리소스가 Limit 상태가 되었을때, 컨트롤러는 이 상태를 파악하고, `Pod`를 하나 더 만들어줌으로써, 부하를 분산시키고 `Pod`가 죽지 않도록 해준다. 그러면 이 서비스는 성능에 대한 장애없이 안정적인 서비스를 운영할 수 있다.

<br>

## Software Update

여러 `Pod`에 대한 버전을 업데이트 하려면, 컨트롤러를 통해서 쉽게 할 수 있다. 업데이트 도중에 문제가 있으면, 롤백할 수 있는 기능을 제공한다.

<br>

## Job

일시적인 작업을 할 수 있도록 도와준다. 컨트롤러가 필요한 순간에 `Pod`를 만들어서 해당 작업을 이행하고, 삭제한다. 그 순간에만 자원이 활용되고, 작업후에 다시 반환되기 때문에 효율적인 자원 활용이 가능해진다.

<br>

## Replication Controller, ReplicaSet 실습

> Pod를 생성하고, 상태를 확인하고, 세부 사항을 확인한다.

```sh
# Pod 생성
$ kubectl apply -f pod1.yml

# Pod 상태 확인
$ kubectl get pods

# Pod 세부 사항 보기
$ kubectl describe pod/pod1
```

> Pod1을 위한 ReplicaSet를 생성한다.

```sh
# ReplicaSet 생성
$ kubectl apply -f replica1.yml

# ReplicaSet 상태 확인
$ kubectl get pods
```

> ReplicaSet을 삭제하면, Pod들도 모두 삭제하게 된다. 따라서 ReplicaSet만 삭제하고 싶으면 다음 명령을 입력해야 한다.

```sh
$ kubectl delete replicationcontrollers replication1 --casecade=false
```

<br>

## Selector 실습

> `matchExpressions`는 자주 사용하지 않고, 사전에 어떤 오브젝트가 미리 만들어져 있는 상황에서 그 오브젝트들에 여러 라벨들이 붙어 있고, 내가 원하는 오브젝트만 세밀하게 사용하고 싶을때 사용한다.

```yml
spec:
  selector:
    matchExpressions:
      - {key: type, operator: In, values:[web]}
      - {key: ver, operator: Exists}
```

> `selector`에서 정의한 `matchLabels`는 `template.metadata.labels`에 반드시 있어야 에러가 나지 않는다. (`matchExpressions`도 마찬가지)

```yml
spec:
  replicas: 1
  selector: # selector에 있는 내용이 template.metadata.labels에 붙어있어야한다.
    matchLabels:
      type: web
      ver: v1
    matchExpressions:
      - {key: type, operator: In, values:[web]}
      - {key: ver, operator: Exists}
  template:
    metadata:
      name: pod1
      labels: # selector에 있는 내용이 labels에 붙어있어야한다.
        type: web
        ver: v1
        ver: v2
```

## 참고

- [Replication Controller, ReplicaSet 강의 자료](https://kubetm.github.io/practice/beginner/controller-replicationcontroller_replicaset/)
