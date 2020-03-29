## Docker 명령어 모음

<br>

### 도커 이미지 빌드

`docker build -t 서비스 이름 .`

### docker-compose.yml에 정의된 컨테이너들을 실행

`docker-compose up -d`

### 이미지 리스트 확인

`docker images`

### 컨테이너 리스트 확인

`docker ps / docker ps -a`

### volume 확인하기

`docker volume ls`

### 사용하지 않는 volume 삭제

`docker volume ls -qf "dangling=true" | xargs docker volume rm`
