# ConfigMap, Secret 실습

<br>

## Env (Literal)

> ConfigMap을 생성합니다.

```zsh
$ kubectl apply -f env-literal-configmap.yml
```

> Secret을 생성합니다.

```zsh
$ kubectl apply -f env-literal-secret.yml
```

> Pod를 생성합니다.

```zsh
$ kubectl apply -f env-literal-pod.yml
```

> 컨테이너에 접속해서 `env`를 확인합니다.

- base64 인코딩 값인 secret 값은 디코딩 되서 `env`에 들어옵니다.

```zsh
# 컨테이너 접속
$ kubectl exec pod-1 -it -c container -- /bin/bash

# env 확인
[root@pod-1 /]$ env
```

<br>

## Env (File)

> 쿠버네티스 Dashboard에서 File로 ConfigMap 만드는 것을 지원하지 않기 때문에 클러스터에 접속해서 File을 만듭니다.

```zsh
# minkube 클러스터 접속
$ minikube ssh
```

> 파일을 생성하고, ConfigMap을 생성합니다.

- minkube 환경에서는 kubectl이 없기 때문에 설치해야 합니다.

```zsh
# minikube 클러스터 환경...
$ echo "Content" >> file-c.txt

$ kubectl create configmap cm-file --from-file=./file-c.txt
```

> 파일을 생성하고, Secret을 생성합니다.

- 자동으로 base64로 인코딩 됩니다.

- minkube 환경에서는 kubectl이 없기 때문에 설치해야 합니다.

```zsh
# minikube 클러스터 환경...
$ echo "Content" >> file-s.txt

$ kubectl create secret generic sec-file --from-file=./file-s.txt
```

> Pod를 생성합니다.

```zsh
$ kubectl apply -f env-file-pod.yml
```

> 컨테이너에 접속해서 `env`을 확인합니다.

```zsh
# 컨테이너 접속
$ kubectl exec pod-file -it -c container -- /bin/bash

# 환경 변수 확인
[root@pod-file /]$ env
```

<br>

## Volume Mount (File)

> Pod를 생성합니다.

```zsh
$ kubectl apply -f volume-mount-file-pod.yml
```

> 컨테이너에 접속해서 `/mount` 경로에 `file-c.txt`가 있는지 확인합니다.

```zsh
# 컨테이너 접속
$ kubectl exec pod-mount -it -c container -- /bin/bash

# file-c.txt 확인
[root@pod-file /]$ cd mount
[root@pod-file /mount]$ ls -al
```

<br>

## Env(File)과 VolumeMount(File)의 큰 차이

> Pod 생성 후, 각각의 ConfigMap 내용이 변경 된다면...?

- `Env` 방식은 한 번 주입하면 끝이기 때문에 Pod가 죽어서 재 생성되어야 변경된 값을 반영한다.

- `Volume Mount` 방식은 내용이 변하면 마운팅된 파일 내용이 바로 변한다.

<br>

## 참고

- [ConfigMap, Secret 강의자료](https://kubetm.github.io/practice/beginner/object-configmap_secret/)
