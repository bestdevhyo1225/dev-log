# MySQL Architecture

<br>

MySQL의 아키텍처를 자세히 설명하기보다 해당 키워드를 작성하고 시간이 나는대로 직접 찾아서 보자

<br>

## :book: MySQL의 전체 구조

![MySQL Architecture Image](https://user-images.githubusercontent.com/23515771/65847567-0cff8400-e37d-11e9-9b07-05297cd98e87.png)

<br>

## :book: 가장 먼저 알아야 핵심은 MySQL 엔진과 스토리지 엔진이다.

`MySQL 엔진`은 DMBS의 두뇌에 해당하는 처리를 수행한다.

* 클라이언트로부터의 접속 및 쿼리 요청을 처리하는 `커넥션 핸들러`

* 쿼리 문장을 토큰으로 분리해 트리 형태의 구조로 만들어 내는 `SQL 파서`

* 파서 트리를 기반으로 쿼리 문장에 구조적인 문제점이 있는지 확인하는 `전처리기`

* 쿼리의 최적화된 실행을 위한 `옵티마이저`

* 성능 향상을 위한 MyISAM의 키 `캐시`나 InnoDB의 `버퍼` 풀과 같은 보조 저장소 기능

`스토리지 엔진`은 실제 데이터를 디스크 스토리지에 저장하거나 디스크 스토리지로부터 데이터를 읽어오는 부분을 담당한다.

<br>

## :book: MySQL 엔진과 스토리지 엔진은 어떻게 데이터를 주고 받을까?

MySQL 엔진의 쿼리 실행기에서 데이터를 쓰거나 읽어야 한다면? `핸들러 API`를 이용해서 스토리지 엔진에게 쓰기 또는 읽기를 요청할 수 있다.

<br>

## :book: MySQL 서버는 프로세스 기반이 아니라 스레드 기반으로 작동한다!

`Foreground Thread - 클라이언트 스레드`

* MySQL 서버에 접속된 클라이언트의 수만큼 존재하며, 주로 각 클라이언트의 사용자가 요청하는 쿼리 문장을 처리하는 것이 임무다. MyISAM 테이블의 경우에는 MySQL의 데이터 버퍼나 캐시로부터 가져온다. 그러나 버퍼나 캐시에 없는 경우에는 직접 디스크의 데이터나 인덱스 파일로부터 데이터를 읽어와서 작업을 처리한다. InnoDB 테이블의 경우에는 데이터 버퍼나 캐시까지만 Foreground Thread가 처리하고, 나머지 버퍼로부터 디스크까지 기록하는 작업은 Background Thread가 처리한다.

`Background Thread`

* 가장 중요한 역할은 로그 스레드(Thread)와 버퍼의 데이터를 디스크로 내려 쓰는 작업을 처리하는 쓰기 스레드(Write thread)이다. 이유는 다음과 같다. 읽기 스레드(Read thread)는 클라이언트 스레드에서 처리하기 때문에 읽기 스레드를 많이 설정할 필요가 없지만, 쓰기 스레드(Write thread)는 아주 많은 작업을 백그라운드로 처리하기 때문에 일반적인 내장 디스크를 사용할 때와 같은 스토리지를 사용할 때는 여유있게 스레드의 개수를 설정해줘야 한다.

<br>

## :book: MySQL의 메모리 구조는?

`글로벌 메모리 영역`

* 클라이언트 스레드의 수와 무관하게 일반적으로는 하나의 메모리 공간만 할당된다. 즉, 모든 스레드가 공유하는 영역이다.

`로컬 메모리 영역`

* 각 클라이언트 스레드별로 독립적으로 할당되며 절대 공유되어 사용되지 않는다는 특징이 있다.

<br>

## :book: 쿼리의 실행 구조에서 핵심은 옵티마이저, 샐행 엔진, 핸들러(스토리지 엔진)이다.

`옵티마이저(Optimizer)`

* 최적화된 쿼리를 실행하기 위한 개념이다. 즉, 쿼리 문장을 저렴한 비용으로 가장 빠르게 처리할지 결정하는 역할을 수행한다.

`실행 엔진`

* 만들어진 실행 계획대로 각 핸들러에게 요청한 결과를 받아서, 다른 핸들러 요청의 입력으로 연결하는 역할을 수행한다.

`핸들러(스토리지 엔진)`

* 실행 엔진에 요청에따라 데이터를 디스크에 저장하거나 디스크에서 데이터를 읽어오는 역할을 수행한다.

<br>

## :book: 복제(Replication)와 쿼리 캐시는 책을 자세히 읽어보자.

<br>

## :bookmark: 참고

* 개발자와 DBA를 위한 Real MySQL
