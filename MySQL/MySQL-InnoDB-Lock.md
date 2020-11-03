# InnoDB의 잠금 방식

들어가기에 앞서 `낙관적 잠금(Optimistic Locking)`과 `비관적 잠금(Pessimistic Locking)`을 간단히 알아보자

> 낙관적 잠금(Optimistic Locking)

- 기본적으로 각 트랜잭션이 같은 레코드를 변경할 가능성은 상당히 희박할 것이라고 가정한다. (낙관적으로...)

- 우선 변경 작업을 수행하고, 마지막에 잠금 충돌이 있었는지 확인하는데 문제가 있다면 ROLLBACK 처리한다.

> 비관적 잠금(Pessimistic Locking)

- 트랜잭션에서 변경하고자 하는 레코드에 대해 잠금을 획득하고, 변경 작업을 처리하는 방식이다.

- 현재 변경하고자 하는 레코드를 다른 트랜잭션에서도 변경할 수 있다는 비관적인 가정을 하기 때문에 먼저 잠금을 획득한 것

- 일반적으로 높은 동시성 처리에는 비관적 잠금이 유리하다고 알려져 있으며, **InnoDB는 비관적 잠금 방식을 채택하고 있다.**

<br>

## InnoDB의 잠금 종류

**`1. 레코드 락 (Record Lock, Record only Lock)`**

**`2. 갭 락 (Gap Lock)`**

**`3. 넥스트 키 락 (Next Key Lock)`**

**`4. 자동 증가 락 (Auto Increment Lock)`**

> 레코드 락 (Record Lock, Record only Lock)

- InnoDB 스토리지 엔진은 레코드 자체가 아니라 인덱스의 레코드를 잠근다는 점이다.

- 만약에 인덱스가 하나도 없는 테이블이라면, 내부적으로 자동 생성된 클러스터 인덱스를 이용해 잠금을 설정한다.

<br>

## 인덱스와 잠금

**InnoDB의 잠금은 레코드를 잠그는 것이 아니라 인덱스를 잠그는 방식으로 처리된다. 즉, 변경해야 할 레코드를 찾기 위해 검색한 인덱스의 레코드를 모두 잠가야 한다.**

```sql
-- employees라는 테이블이 있고, first_name, last_name, hire_date 컬럼이 있다.
-- first_name에 idx_first_name이라는 인덱스가 준비된 상황 idx_first_name(first_name)

-- employees 테이블에서 first_name = 'Georgi'인 사원은 전체 253명이 있으며,
-- first_name = 'Georgi' 이고 last_name = 'Klassen'인 사원은 딱 1명만 있는 상황이다.

SELECT COUNT(*) FROM employees WHERE first_name = 'Georgi';
-- +----------+
-- |      253 |
-- +----------+

SELECT COUNT(*) FROM employees WHERE first_name = 'Georgi' AND last_name = 'Klassen';
-- +----------+
-- |        1 |
-- +----------+

-- first_name = 'Georgi', last_name = 'Klassen' 인 사원의 입사 일자를 오늘로 변경하는 쿼리를 실행하면?
UPDATE employees SET hire_date = NOW()
WHERE first_name = 'Georgi' AND last_name = 'Klassen';
```

UPDATE가 실행되면, 1건의 레코드가 업데이트 된다. 이 1건의 UPDATE를 위해 몇 개의 레코드에 Lock을 걸어야 할까? UPDATE 쿼리 조건에서 인덱스를 이용할 수 있는 조건은 `first_name`이며, `last_name` 컬럼은 인덱스가 없기 때문에 `frist_name = 'Georgi'`인 253건의 레코드가 모두 잠긴다.

> idx_first_name 인덱스를 확인해보면, first_name = 'Georgi'인 모든 인덱스가 잠금 상태가 되버린다.

| first_name | emp_no | 잠금 상태 |
| :--------: | :----: | :-------: |
|   Georgi   | 10001  |     O     |
|   Georgi   | 13457  |     O     |
|   Georgi   | 18320  |     O     |
|   Georgi   | 18705  |     O     |
|   Georgi   | 19203  |     O     |
|    ...     |  ...   |    ...    |
|   Georgy   | 23125  |     X     |

> 실제 변경된 레코드는 first_name = 'Georgi', last_name = 'Klassen' 인 1건의 레코드뿐이다.

|  emp_no   | first_name |  last_name  | 잠금 상태 | 변경 상태 |        기타        |
| :-------: | :--------: | :---------: | :-------: | :-------: | :----------------: |
|   10001   |   Georgi   |   Facello   |     O     |     X     |                    |
|   13457   |   Georgi   |   Atchley   |     O     |     X     |                    |
|   18320   |   Georgi   |  Itzfeldt   |     O     |     X     |                    |
| **18705** | **Georgi** | **Klassen** |     O     |     O     | 실제 변경된 레코드 |
|   19203   |   Georgi   |   Barinka   |     O     |     X     |                    |

마지막으로 해당 테이블에 인덱스가 하나도 없다면, 테이블을 풀 스캔하면서 UPDATE 작업을 한다. 이 과정에서 만약 테이블에 있는 30여 만 건의 레코드가 있다면, 모두 잠그게 된다. 이것이 MySQL 방식이며, MySQL의 InnoDB에서 인덱스 설계가 중요한 이유 또한 이 때문이다.

<br>

## 참고

- 개발자와 DBA를 위한 Real MySQL
