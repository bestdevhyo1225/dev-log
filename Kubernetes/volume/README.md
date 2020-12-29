# Volume 실습

<br>

## emptyDir

:pushpin: `pod-volume-1.yml`에 정의한 내용을 통해 Pod를 생성한다.

```zsh
$ kubectl apply -f pod-volume-1.yml

$ kubectl get pods
```

:pushpin: container1에 접속해서 mount1이 생성 되었는지 확인한다.

```zsh
$ kubectl exec pod-volume-1 -it -c container1 -- /bin/bash

[root@pod-volume-1 /]$ ls -al

[root@pod-volume-1 /]$ mount | grep mount1
```

:pushpin: /mount1 디렉토리로 이동해서 file을 생성한다.

```zsh
[root@pod-volume-1 /]$ cd mount1

[root@pod-volume-1 /mount1]$ echo "file context" >> file.txt

[root@pod-volume-1 /mount1]$ cat file.txt
```

:pushpin: container2로 이동해서 /mount2에 file.txt가 있는지 확인한다.

```zsh
$ kubectl exec pod-volume-1 -it -c container2 -- /bin/bash

# 컨테이너 접속 후
[root@pod-volume-1 /]$ ls -al

[root@pod-volume-1 /]$ cd mount2

[root@pod-volume-1 /mount2]$ cat file.txt
```

<br>

## hostPath

:pushpin: pod-volume-2.yml, pod-volume-3.yml에 정의된 Pod를 생성한다.

- `type: DirectoryOrCreate`로 설정하면, 디렉토리가 없을때 생성한다는 의미이다.

```zsh
$ kubectl apply -f pod-volume-2.yml

$ kubectl apply -f pod-volume-3.yml

$ kubectl get pods
```

:pushpin: pod-2에 생성된 컨테이너에 접속해서 /mount1 디렉토리를 확인한다.

```zsh
$ kubectl exec pod-volume-2 -it -c container -- /bin/bash

[root@pod-volume-2 /]$ ls -al
```

:pushpin: /mount1 디렉토리로 이동해서 file을 생성한다.

```zsh
[root@pod-volume-2 /]$ cd mount1

[root@pod-volume-2 /mount1]$ echo "file context" >> file.txt

[root@pod-volume-2 /mount1]$ cat file.txt
```

:pushpin: pod-3의 container로 이동해서 /mount1에 file.txt가 있는지 확인한다.

```zsh
$ kubectl exec pod-volume-3 -it -c container -- /bin/bash

[root@pod-volume-3 /]$ ls -al

[root@pod-volume-3 /]$ cd mount1

[root@pod-volume-3 /mount1]$ cat file.txt
```

:pushpin: Node로 접속해서 /node-v 디렉토리에 file.txt가 있는지 확인한다.

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

:pushpin: persistent volume을 생성한다.

```zsh
# storage: 1G, accessModes: ReadWriteOnce
$ kubectl apply -f persistent-volume-1.yml

# storage: 1G, accessModes: ReadOnlyMany
$ kubectl apply -f persistent-volume-2.yml

# storage: 2G, accessModes: ReadWriteOnce
$ kubectl apply -f persistent-volume-3.yml
```

:pushpin: persistent voluem claim을 생성하면, persistent volume에 연결되는 것을 확인할 수 있다.

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

:pushpin: Pod를 생성해서 PVC를 연결한다.

```zsh
$ kubectl apply -f pod-volume-4.yml
```

:pushpin: 컨테이너에 접속해서 /mount3 디렉토리를 확인한다.

```zsh
$ kubectl exec pod-volume-4 -it -c container -- /bin/bash

[root@pod-volume-4 /]$ ls -al

[root@pod-volume-4 /mount3]$ ls -al

[root@pod-volume-4 /mount3]$ cat file.txt
```

<br>

## 참고

- [Volume 강의 자료](https://kubetm.github.io/practice/beginner/object-volume/)
