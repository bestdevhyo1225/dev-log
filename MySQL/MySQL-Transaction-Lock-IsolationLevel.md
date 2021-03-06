## MySQL에서 트랜잭션, 잠금, 트랜잭션의 격리 수준

<br>

### :book: 간단하게 트랜잭션에 대해 말해본다면?

우선 `트랜잭션`은 어떠한 작업이 이루어지는 단위를 말하며, 데이터의 정합성을 보장하기 위한 기능으로 사용된다.

<br>

### :book: MySQL에서 트랜잭션

하나의 논리적인 작업 셋에서 하나의 쿼리가 있던지 여러 개의 쿼리가 있던지 그것과 상관없이 논리적인 작업 셋 자체가 100% 적용되거나 또는 아무것도 적용되지 않는것을 보장해 주는 것이다.

* COMMIT

* ROLLBACK

* 트랜잭션을 ROLLBACK 시키는 오류가 발생했 때

<br>

### :book: MySQL에서 트랜잭션을 사용할 때 고려해야 하는 점이 있다면? 또는 주의해야 할 점?

**DBMS의 커넥션과 동일하게 꼭 필요한 최소의 코드에만 적용하는 것이 좋다.** 이는 프로그램 코드에서 트랜잭션의 범위를 최소화하라는 의미이다. 다음 예제를 보면서 이해하면 된다. Real MySQL 책을 참고한 예제이다.

    1) 처리 시작
    ==> 데이터베이스 커넥션 생성
    ==> 트랜잭션 시작
    2) 사용자의 로그인 여부 확인
    3) 사용자의 글쓰기 내용의 오류 여부 확인
    4) 첨부로 업로드 된 파일 확인 및 저장
    5) 사용자의 입력 내용을 DBMS에 저장
    6) 첨부 파일 정보를 DBMS에 저장
    7) 저장된 내용 또는 기타 정보를 DBMS에서 조회
    8) 게시물 등록에 대한 알림 메일 발송
    9) 알림 메일 발송 이력을 DBMS에 저장
    <== 트랜잭션 종료(COMMIT)
    <== 데이터베이스 커넥션 반납
    10) 처리 완료

위 예제에서 DBMS의 트랜잭션 처리에 좋지 않은 영향을 끼치는 부분은 어떤 부분일까??

* 실제로 DBMS에 대이터를 저장하는 작업(트랜잭션)은 5번부터 시작된다는 것을 알 수 있다.

* 2번과 3번 그리고 4번의 절차가 아무리 빨리 처리된다 하더라도 DBMS의 트랜잭션으로 포함시킬 필요는 없다.

* 데이터베이스 커넥션은 개수가 제한적이다. 따라서 각 단위 프로그램이 커넥션을 소유하는 시간이 길어질수록 사용 가능한 여유 커넥션의 개수는 줄어들 것이다. 어느 순간에는 각 프로그램들이 커넥션을 가져가기 위해 기다려야 하는 상황이 발생할 수도 있다.

* 제일 위험한 작업은 8번이다. 메일 전송과 같은 네트워크를 통해 원격 서버와 통신하는 작업은 트랜잭션에서 제거해야한다. 만약에 메일 서버와 통신할 수 없는 상황이 발생한다면, 웹 서버 뿐만 아니라 DBMS 서버까지 위험해지는 상황이 발생할 것이다.

* DBMS의 작업이 크게 4개가 있다. 5번, 6번, 7번, 9번이다. 5번과 6번은 반드시 하나의 트랜잭션으로 묶고, 7번 작업은 저장된 데이터의 단순 확인 및 조회이므로 트랜잭션에 포함할 필요도 없고, 사용할 필요도 없다. 그리고 9번 작업은 조금 성격이 다르다. 따라서 트랜잭션에 포함할 필요가 없다.

위의 내용을 참고하여 다시 설계한 처리 절차

    1) 처리 시작
    2) 사용자의 로그인 여부 확인
    3) 사용자의 글쓰기 내용의 오류 발생 여부 확인
    4) 첨부로 업로드 된 파일 확인 및 저장
    ==> 데이터베이스 커넥션 생성(또는 커넥션 풀에서 가져오기)
    ==> 트랜잭션 시작
    5) 사용자의 입력 내용을 DBMS에 저장
    6) 첨부 파일 정보를 DBMS에 저장
    <== 트랜잭션 종료(COMMIT)
    7) 저장된 내용 또는 기타 정보를 DBMS에서 조회
    8) 게시물 등록에 대한 알림 메일 발송
    ==> 트랜잭션 시작
    9) 알림 메일 발송 이력을 DBMS에 저장
    <== 트랜잭션 종료(COMMIT)
    <== 데이터베이스 커넥션 종료(또는 커넥션 풀에 반납)
    10) 처리 완료

<br>

### :book: 잠금이란??

`잠금`은 동시성을 제어하기 위한 기능이다. 만약에 `잠금`이 없다면, 여러 커넥션에서 한 데이터를 동시에 변경해버릴 수 있는 위험성을 가지고 있다.

<br>

### :book: 트랜잭션의 격리 수준

`격리 수준`은 하나의 트랜잭션 내에서 또는 여러 트랜잭션 간의 작업 내용을 어떻게 공유하고 차단할 것인지를 결정하는 레벨을 의미한다.

<br>

### :bookmark: 참고

* 개발자와 DBA를 위한 Real MySQL