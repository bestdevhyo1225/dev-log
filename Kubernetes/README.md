# Kubernetes 실습 예제

[인프런 - 대세는 쿠버네티스 ^o^](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88) 강의를 들으면서 직접 실습해보고 공부하는 공간

<br>

## 환경 구성

> 만약 MacOS 버전을 업데이트 했다면, 시스템 환경 설정 -> 보안 및 개인 정보 보호로 이동해서 `개발자 'Oracle America, Inc.'의 시스템 소프트웨어가 업데이트되었습니다.`를 허용해준다.

<img width="651" alt="스크린샷 2021-01-02 오후 6 18 04" src="https://user-images.githubusercontent.com/23515771/103454418-136e6f80-4d27-11eb-9058-8288335d55db.png">

> HyperVisor중에서 `virtualbox`를 설치한다.

```sh
$ brew install --cask virtualbox virtualbox-extension-pack
```

> 로컬에서 쿠버네티스를 쉽게 실행시켜주는 `minikube`를 설치한다.

```sh
$ brew install minikube
```

> 쿠버네티스를 컨트롤 할 수 있는 `kubectl`을 설치한다.

```sh
$ curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl"

$ chmod +x ./kubectl
$ mv ./kubectl /usr/local/bin/kubectl
```

> `minkube`를 `virtualbox` 드라이버로 실행한다.

```sh
$ minikube start --vm-driver=virtualbox
```

> 쿠버네티스 클러스터의 대시보드를 확인하고 싶으면 아래와 같이 명령어를 입력한다.

```sh
$ minikube dashboard
```

<br>

## 참고

- [인프런 - 대세는 쿠버네티스 ^o^](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88)

- [KubeTM Blog](https://kubetm.github.io/)
