# Kubernetes 실습 예제

[인프런 - 대세는 쿠버네티스 ^o^](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88) 강의를 들으면서 직접 실습해보고 공부하는 공간입니다.

<br>

## 환경 구성

#### :pushpin: HyperVisor중에서 `virtualbox`를 설치합니다.

```sh
$ brew cask install virtualbox virtualbox-extension-pack
```

#### :pushpin: 로컬에서 쿠버네티스를 쉽게 실행시켜주는 `minikube`를 설치합니다.

```sh
$ brew install minikube
```

#### :pushpin: 쿠버네티스를 컨트롤 할 수 있는 `kubectl`을 설치합니다.

```sh
$ curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl"

$ chmod +x ./kubectl
$ mv ./kubectl /usr/local/bin/kubectl
```

#### :pushpin: `minkube`를 `virtualbox` 드라이버로 실행합니다.

```sh
$ minikube start --vm-driver=virtualbox
```

#### :pushpin: 쿠버네티스 클러스터의 대시보드를 확인하고 싶으면 아래와 같이 명령어를 입력합니다.

```sh
$ minikube dashboard
```

<br>

## 기본 오브젝트 실습

- [Pod 실습](https://github.com/bestdevhyo1225/kubernetes-study/tree/master/pod)

- [Service 실습](https://github.com/bestdevhyo1225/kubernetes-study/tree/master/service)

- [Volume 실습](https://github.com/bestdevhyo1225/kubernetes-study/tree/master/volume)

- [ConfigMap, Secret 실습](https://github.com/bestdevhyo1225/kubernetes-study/tree/master/configmap-secret)

- [Namespace, ResourceQuota, LimitRange 실습](https://github.com/bestdevhyo1225/kubernetes-study/tree/master/namespace-resourcequota-limitrange)

<br>

## 참고

- [인프런 - 대세는 쿠버네티스 ^o^](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88)

- [KubeTM Blog](https://kubetm.github.io/)
