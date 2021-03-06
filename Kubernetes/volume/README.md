# Volume 실습

<br>

## emptyDir

![image](https://kubetm.github.io/img/practice/beginner/Volume%20with%20emptyDir%20for%20Kubernetes.jpg)

`emptyDir`은 `Pod`안에 컨테이너들이 서로 데이터를 공유가 필요할 때 사용하는 것이다. 최초 볼륨이 생성될 때는 볼륨 안에 내용이 비어있기 때문에 `emptyDir`라고 명칭이 된 것이다. 중요한 점은 `Pod`안에 생성되기 때문에 `Pod`에 문제가 있으면, 데이터가 모두 삭제된다. 따라서 일시적인 데이터만 보관하고 있어야 한다.

<br>

## emptyDir 실습

> `pod-volume-1.yml`에 정의한 내용을 통해 Pod를 생성한다.

```zsh
$ kubectl apply -f pod-volume-1.yml

$ kubectl get pods
```

> container1에 접속해서 mount1이 생성 되었는지 확인한다.

```zsh
$ kubectl exec pod-volume-1 -it -c container1 -- /bin/bash

[root@pod-volume-1 /]$ ls -al

[root@pod-volume-1 /]$ mount | grep mount1
```

> /mount1 디렉토리로 이동해서 file을 생성한다.

```zsh
[root@pod-volume-1 /]$ cd mount1

[root@pod-volume-1 /mount1]$ echo "file context" >> file.txt

[root@pod-volume-1 /mount1]$ cat file.txt
```

> container2로 이동해서 /mount2에 file.txt가 있는지 확인한다.

```zsh
$ kubectl exec pod-volume-1 -it -c container2 -- /bin/bash

# 컨테이너 접속 후
[root@pod-volume-1 /]$ ls -al

[root@pod-volume-1 /]$ cd mount2

[root@pod-volume-1 /mount2]$ cat file.txt
```

<br>

## hostPath

![image](https://kubetm.github.io/img/practice/beginner/Volume%20with%20hostPath%20for%20Kubernetes.jpg)

`Pod`에 있는 데이터를 저장하기 위한 용도가 아니라 `Node`에 있는 데이터를 `Pod`에서 쓰기위한 용도이다. 아까 `emptyDir`과는 다르게 `Node`에 볼륨이 올라가 있기 때문에 `Pod`가 죽어도 볼륨안에 있는 데이터는 사라지지 않는다. 그러나 `Pod` 입장에서 보면, 자기 자신이 죽고 다시 재생성 됐을때, 해당 `Node`에서 다시 생성된다는 보장이 없다. 즉, 다른 `Node2`에서 재생성 될 수 있으며, 이러한 경우에는 기존에 사용했던 `Node`의 볼륨을 사용할 수 없다.

> 언제 사용하나요?

- 각 `Node` 자신을 위해서 사용되는 파일들이 있다.(시스템 파일, 설정 파일) `Pod` 자신이 할당되어 있는 호스트의 파일을 읽거나 쓰는 경우에 사용하면 된다.

> 주의할 점?

- 사전에 해당 `Node`에 디렉토리가 존재해야 에러가 발생하지 않는다.

```yml
spec:
  volumes:
    - name: host-path
      hostPath:
        path: /node-v # 해당 디렉토리가 미리 만들어져 있어야 에러가 나지 않는다.
        type: DirectoryOrCreate
```

<br>

## hostPath 실습

> pod-volume-2.yml, pod-volume-3.yml에 정의된 Pod를 생성한다.

- `type: DirectoryOrCreate`로 설정하면, 디렉토리가 없을때 생성한다는 의미이다.

```zsh
$ kubectl apply -f pod-volume-2.yml

$ kubectl apply -f pod-volume-3.yml

$ kubectl get pods
```

> pod-2에 생성된 컨테이너에 접속해서 /mount1 디렉토리를 확인한다.

```zsh
$ kubectl exec pod-volume-2 -it -c container -- /bin/bash

[root@pod-volume-2 /]$ ls -al
```

> /mount1 디렉토리로 이동해서 file을 생성한다.

```zsh
[root@pod-volume-2 /]$ cd mount1

[root@pod-volume-2 /mount1]$ echo "file context" >> file.txt

[root@pod-volume-2 /mount1]$ cat file.txt
```

> pod-3의 container로 이동해서 /mount1에 file.txt가 있는지 확인한다.

```zsh
$ kubectl exec pod-volume-3 -it -c container -- /bin/bash

[root@pod-volume-3 /]$ ls -al

[root@pod-volume-3 /]$ cd mount1

[root@pod-volume-3 /mount1]$ cat file.txt
```

> Node로 접속해서 /node-v 디렉토리에 file.txt가 있는지 확인한다.

```zsh
$ minikube ssh

# minkube node 접속...
$ cd /

$ ls -al

$ cd node-v

$ cat file.txt
```

<br>

## PV(Persistent Volume) / PVC(Persistent Volume Claim)

![image](https://kubetm.github.io/img/practice/beginner/Volume%20with%20PersistentVolume%20PersistentVolumeClaim%20for%20Kubernetes.jpg)

`PV/PVC`는 `Pod`에 영속성이 있는 볼륨을 제공하기 위한 기능이다.

<br>

## PV(Persistent Volume) / PVC(Persistent Volume Claim) 실습

> persistent volume을 생성한다.

```zsh
# storage: 1G, accessModes: ReadWriteOnce
$ kubectl apply -f persistent-volume-1.yml

# storage: 1G, accessModes: ReadOnlyMany
$ kubectl apply -f persistent-volume-2.yml

# storage: 2G, accessModes: ReadWriteOnce
$ kubectl apply -f persistent-volume-3.yml
```

> persistent voluem claim을 생성하면, persistent volume에 연결되는 것을 확인할 수 있다.

- 만약 PVC의 storage가 PV의 storage보다 크게 설정하면, 바인딩이 되지 않는다.

- 반대로 PVC의 storage가 PV의 storage보다 작게 설정하면, 바인딩이 된다.

```zsh
# storage: 1G, accessModes: ReadWriteOnce인 PV에 연결된다.
$ kubectl apply -f persistent-volume-claim-1.yml

# storage: 1G, accessModes: ReadOnlyMany인 PV에 연결된다.
$ kubectl apply -f persistent-volume-claim-2.yml

# storage: 2G, accessModes: ReadWriteOnce인 PV에 연결된다.
$ kubectl apply -f persistent-volume-claim-3.yml
```

> Pod를 생성해서 PVC를 연결한다.

```zsh
$ kubectl apply -f pod-volume-4.yml
```

> 컨테이너에 접속해서 /mount3 디렉토리를 확인한다.

```zsh
$ kubectl exec pod-volume-4 -it -c container -- /bin/bash

[root@pod-volume-4 /]$ ls -al

[root@pod-volume-4 /mount3]$ ls -al

[root@pod-volume-4 /mount3]$ cat file.txt
```

<br>

## 참고

- [Volume 강의 자료](https://kubetm.github.io/practice/beginner/object-volume/)
