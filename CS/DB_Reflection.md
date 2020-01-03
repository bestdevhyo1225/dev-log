## Reflection

<br>

### :book: Reflection에 대해서

* 쉽게 말하자면, `실시간 복제본 서버`라고 할 수 있다.

* 본래 서버는 `Master 서버`라고 칭한다.

* 실시간 복제되는 서버는 `Slave 서버`라고 칭한다.

* `Master 서버`에서는 `쓰기 연산`을 수행한다.

* `Slave 서버`에서는 `읽기 연산`을 수행한다.

* `쓰기 연산`을 담당하는 `Master 서버`에서 DB에 변동사항이 생겼을 시, 그 변동사항을 실시간으로 `Slave 서버`측에 복사되어 반영한다.

* `쓰기` 처리문은 `Master 서버`로, `읽기` 처리문은 `Slave 서버`가 맡아서 진행할 수 있게 된다.

<br>

### :book: 장점

Reflection을 사용하면, DB 서버에 부하 분산을 유도해낼 수 있다. 더 나아가 실 서비스에선 Clustering을 통해, `Master 서버`가 ShutDown 되었을 때, `Slave 서버`를 `Master`로 승격하여 FailOver를 통한 고가용성 DB서버를 구축할 수 있다.

<br>

### :bookmark: 참고

* [RDS Reflection](https://medium.com/@qhdrbs1341/rds-replication-with-sequelize-35f335ce07d3)