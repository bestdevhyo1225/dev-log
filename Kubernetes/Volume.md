# Volume

<br>

## emptyDir

- 컨테이너들끼리 데이터를 공유하기 위한 목적으로 사용된다.

- Pod 생성시 만들어지고, 삭제시 없어지기 때문에 일시적인 사용목적에 의한 데이터만 넣는게 좋다.

<br>

## hostPath

- **Pod의 데이터를 저장하기 위한 용도가 아니라 Node에 있는 데이터를 Pod에서 쓰기위한 용도이다.**

- 각각의 Node에는 기본적으로 Node 자신을 위해서 사용되는 파일들이 있고, Pod가 할당되어 있는 host에 데이터를 읽거나 써야할 때 사용된다.

- 각각의 Pod들이 마운트에서 공유하기 때문에 Pod들이 죽어도 마운트에 있는 데이터는 사라지지 않는다.

- 주의할 점은 Pod가 재 생성될 때 꼭 원래 있었던 Node에 생성되는 법은 없다. (Node Scheduler가 각 노드의 자원 및 상황을 파악해서 Pod를 배치하기 때문에)

<br>

## PVC / PV

- Pod에 영속성 있는 볼륨을 제공하기 위한 개념이다.

- User 영역과 Admin 영역을 나눠서 관리한다.

  - User 영역 : Pod, PVC (Persistent Volume Claim)

  - Admin 영역 : Persistent Volume (PV), Volume

- User는 Pod에 서비스를 만들고 배포를 관리하는 서비스 담당자이다.

- Admin은 쿠버네티스를 담당하는 운영자이다.

- 생성 과정은 아래와 같이 요약할 수 있다.

  1. PV 정의 생성

  2. PVC 생성

  3. PV 연결

  4. Pod 생성시 PVC 마운팅
