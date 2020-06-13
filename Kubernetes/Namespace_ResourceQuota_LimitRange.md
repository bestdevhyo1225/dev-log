# Namespace, ResourceQuota, LimitRange

<br>

## 사용해야 하는 이유?

`Kubernetes Cluster`에는 전체 사용할 수 있는 자원이 있고, `Cluster`안에는 각각의 `Namespace`가 있다. 그리고 `Namespace` 안에 있는 각각의 `Pod`들이 있다.`Pod`는 필요한 자원을 `Cluster`자원을 공유해서 사용한다. 만약 한 `Pod`가 `Cluster`에 남은 자원을 모두 사용해버리면 다른 `Pod`에 입장에서 쓸 자원이 없고, 자원이 필요할 때 문제가 발생한다.

#### ResouceQuota

이러한 문제를 해결하기 위해 `ResourceQuota`가 있고, `Namespace`에 설정하면 `Namespace`의 자원 한계를 설정할 수 있다.

#### LimitRange

한 `Pod`가 자원 사용량을 너무 크게 해버리면 다른 `Pod`들이 해당 `Namespace`에 들어올수 없게 된다. 이 부분을 관리하기 위해서 `LimitRange`를 적용해서 `Namespace`에 들어오는 `Pod`의 크기를 제한할 수 있다. 즉, 한 `Pod` 자원 사용량이 `LimitRange`보다 작아야 `Namespace`에 들어올 수 있고, 이보다 클 경우 들어올 수 없다.

#### 다양하게 사용되는 ResouceQuota, LimitRange

`ResourceQuota`와 `LimitRange`는 `Namespace`뿐만 아니라 `Cluster`에도 자원에 대한 제한을 걸 수 있다.

<br>

## 참고

- [인프런 - 대세는 쿠버네티스 ^o^](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88)
