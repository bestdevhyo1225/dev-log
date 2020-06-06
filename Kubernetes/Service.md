# Service

<br>

## :book: Service 요약

- Service는 자신의 클러스터 IP를 가지고 있다.

- Service를 Pod에 연결시켜 놓으면, Service IP를 통해서 Pod에 접근할 수 있다.

<br>

## :question: Service를 쓰는 이유

- Pod는 시스템 장애, 성능 장애와 같은 이유로 언제든지 죽을 수 있고 다시 재 생성되도록 설계 되어있는 오브젝트이다.

- Pod의 IP는 재 생성되면 변하기 때문에 Pod IP는 신뢰성이 떨어진다.

- Service는 사용자가 직접 지우지 않는 이상 삭제되거나 재 생성되지 않기 때문에 사용한다.
