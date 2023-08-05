# MySQL 쿼리 최적화 - JOIN

## JOIN의 순서와 인덱스

- `인덱스 레인지 스캔` 으로 레코드를 읽는 작업은 다음과 같다.
    1. 인덱스 조건을 만족하는 값이 지정된 위치를 찾는다. -> `인덱스 탐색(Index Seek)`
    2. 1번에서 탐색된 위치에서 필요한 만큼 인덱스를 쭉 읽는다. -> `인덱스 스캔(Index Scan)`
    3. 2번에서 읽어들인 `인덱스 키` 와 `레코드 주소` 를 이용해 레코드가 저장된 페이지를 가져오고 `최종 레코드` 를 읽어온다.
- 조인 작업에서 `드라이빙 테이블(Driving)` 을 읽을 때와 `드리븐 테이블(Driven)` 을 읽을 때를 확인해 보면 다음과 같다.

### 드라이빙 테이블 (Driving Table)

- `인덱스 탐색` 을 `1번` 수행하고, 그 이후 부터는 `인덱스 스캔` 만 실행하면 된다.

### 드리븐 테이블 (Driven Table)

- `인덱스 탐색` 과 `인덱스 스캔` 작업을 `드라이빙 테이블에서 읽은 레코드 건 수` 만큼 반복한다.
- 따라서 `드리븐 테이블(Driven Table)` 을 읽는 것이 훨씬 큰 부하를 차지한다. 그래서 `옵티마이저` 는 항상 드라이빙 테이블이
  아닌 `드리븐 테이블(Driven Table)` 을 `최적으로 읽을 수 있게 실행
  계획을 수립한다.`

### 드라이빙(Driving), 드리븐(Driven) 테이블이 선택되는 기준

```mysql
SELECT e.*, de.*
FROM employees e,
     dept_emp de
WHERE e.emp_no = de.emp_no;
```

> 두 컬럼 모두 인덱스가 있는 경우

- 각 테이블의 통계 정보에 있는 레코드 건수에 따라 employees가 드라이빙 테이블이 될 수도 있고, dept_emp 테이블이 드라이빙 테이블로 선택될 수도 있다. 어느 쪽 테이블이 드라이빙 테이블이 되든
  옵티마이저가 선택하는 방법이 최적일 때가 많다.

> employees.emp_no 에만 인덱스가 있는 경우

- 이 때 옵티마이저는 employees 테이블을 드리븐 테이블로 선택해서 인덱스 스캔을 진행하고, dept_emp 테이블을 드라이빙 테이블로 선택해서 풀 스캔을 진행한다. (드리븐 테이블 읽는 것을 최소화 해야
  하기 때문에 인덱스 스캔)

> dept_emp.emp_no 에만 인덱스가 있는 경우

- 이 때 옵티마이저는 dept_emp 테이블을 드리븐 테이블로 선택해서 인덱스 스캔을 진행하고, employees 테이블을 드라이빙 테이블로 선택해서 풀 스캔을 진행한다. (드리븐 테이블 읽는 것을 최소화 해야
  하기 때문에 인덱스 스캔)

> 두 컬럼 모두 인덱스가 없는 경우

- 어떤 테이블을 선택하더라도 드리븐 테이블의 풀 스캔은 발생하기 때문에 옵티마이저가 적절히 드라이빙 테이블을 선택한다. 단 레코드 건수가 적은 테이블을 드리븐 테이블로 선택하는 것이 효율적이다. 또한 드리븐
  테이블을 읽을 때 조인 버퍼가 사용되기 때문에 실행 계획의 Extra 칼럼에 "Using join buffer" 가 표시된다.

## OUTER JOIN의 성능과 주의사항

```mysql
SELECT e.*, de.*, d.*
FROM employees e
LEFT JOIN dept_emp de ON de.emp_no = e.emp_no
LEFT JOIN departments d ON d.dept_no = de.dept_no AND d.dept_name = 'Development';
```

- 위의 `LEFT OUTER JOIN` 을 사용한 쿼리의 실행 계획은 다음과 같다.

| id | table |  type  |        key        |  rows  |    extra    |
|:--:|:-----:|:------:|:-----------------:|:------:|:-----------:|
| 1  |   e   |  ALL   |       NULL        | 299920 |    NULL     |
| 1  |  de   |  ref   | ix_empno_fromdate |   1    |    NULL     |
| 1  |   d   | eq_ref |      PRIMARY      |   1    | Using where |

- MySQL 옵티마이저는 `절대 아우터로 조인되는 테이블` 을 `드라이빙 테이블로 선택하지 못하기` 때문에 `풀 테이블 스캔` 이 필요한 `employees` 테이블을 `드라이빙 테이블` 로 선택한다.
    - 즉, 쿼리의 성능이 떨어지는 실행 계획을 수립한다.
- `INNER JOIN` 을 사용했다면, 다음과 같이 `depmartments` 테이블에서 부서명이 `Development` 인 레코드 1건만 찾아서 조인을 실행하는 계획을 선택했을 것이다.

```mysql
SELECT e.*, de.*, d.*
FROM employees e
INNER JOIN dept_emp de ON de.emp_no = e.emp_no
INNER JOIN departments d ON d.dept_no = de.dept_no
WHERE d.dept_name = 'Development';
```

- `INNER JOIN` 을 사용한 쿼리의 실행 계획은 다음과 같다.

| id | table |  type  |     key     | rows  |    extra    |
|:--:|:-----:|:------:|:-----------:|:-----:|:-----------:|
| 1  |   d   |  ref   | ux_deptname |   1   | Using index |
| 1  |  de   |  ref   |   PRIMARY   | 41392 |    NULL     |
| 1  |   e   | eq_ref |   PRIMARY   |   1   |    NULL     |

- `d.dept_name = 'Development'` 에 의해서 `departments` 테이블의 모수가 줄어들고, `departments` 테이블을 `드라이빙 테이블` 로 선택된다.
- 필요한 데이터와 조인되는 테이블 간의 관계를 정확히 파악해서 꼭 필요한 경우가 아니라면, `INNER JOIN` 을 사용하는 것이 `업무 요건을 정확히 구현함` 과 동시에 `쿼리의 성능도 향상시킬 수 있다.`

```mysql
SELECT e.*
FROM employees e
LEFT JOIN dept_manager mgr ON mgr.emp_no = e.emp_no
WHERE mgr.dept_no = 'd001';
```

- 위의 쿼리로 작성하게 되면, 옵티마이저는 `LEFT OUTER JOIN` 쿼리를 `INNER JOIN` 으로 변환해서 실행한다.

```mysql
SELECT e.*
FROM employees e
LEFT JOIN dept_manager mgr ON mgr.emp_no = e.emp_no AND mgr.dept_no = 'd001';
```

- 정상적으로 `LEFT OUTER JOIN` 이 되게 만들려면, `WHERE` 절의 `mgr.dept_no = 'd001'` 조건을 `LEFT OUTER JOIN` 의 `ON` 절로 옮겨야 한다.

```mysql
SELECT e.*
FROM employees e
LEFT JOIN dept_manager mgr ON mgr.emp_no = e.emp_no
WHERE mgr.emp_no IS NULL
LIMIT 10;
```

- 예외적으로 `WHERE` 절을 사용하고도 `LEFT OUTER JOIN` 이 실행되는 경우가 있으며, 다음과 같이 `안티 조인(ANTI-JOIN)` 효과를 기대하는 경우가 그렇다.
- 위의 쿼리의 형태만 유일하게 `WHERE` 절을 사용하고도 `OUTER JOIN` 이 실행되는 경우이다.
- 그 외의 경우, MySQL 서버는 `LEFT OUTER JOIN` 을 `INNER JOIN` 으로 자동 변환한다.

## JOIN과 외래키

- 외래키는 조인과 아무런 연관이 없다.
- 외래키를 생성하는 주목적은 데이터의 무결성을 보장하기 위해서이다.
    - 외래키와 연관된 무결성을 참조 무결성이라고 표현한다.
- SQL로 테이블 간의 조인을 수행하는 것은 무관한 컬럼을 조인 조건으로 사용해도 문법적으로는 문제가 되지 않는다.
- 데이터 모델을 데이터베이스에 생성할 때는 그 테이블 간의 관계는 외래키로 생성하지 않을때가 더 많다.
- 테이블 간의 조인을 사용하기 위해 외래키가 필요한 것은 아니다.

## 지연된 조인

- 조인을 사용해서 데이터를 조회하는 쿼리에 `GROUP BY`, `ORDER BY` 를 사용할 때, 각 처리 방법에서 인덱스를 사용한다면 이미 최적으로 처리되고 있을 가능성이 높다.
- 그렇지 못하다면, MySQL 서버는 우선 모든 조인을 실행하고 난 다음 `GROUP BY` 나 `ORDER BY` 를 처리할 것이다.
- 조인의 결과를 `GROUP BY` 하거나 `ORDER BY` 하면, 조인을 실행하기 전의 레코드에 `GROUP BY` 나 `ORDER BY` 를 수행하는 것보다 `많은 레코드를 처리해야 한다.`
- `지연된 조인` 이란 조인이 실행되기 이전에 `GROUP BY` 나 `ORDER BY` 를 처리하는 방식을 의미한다.
- `지연된 조인` 은 주로 `LIMIT` 가 함께 사용된 쿼리에서 `더 큰 효과를 얻을 수 있다.`

### 지연된 조인을 사용하지 않은 쿼리

```mysql
SELECT e.*
FROM salaries s, employees e
WHERE e.emp_no = s.emp_no
  AND s.emp_no BETWEEN 10001 AND 13000
GROUP BY s.emp_no
ORDER BY SUM(s.salary) DESC
LIMIT 10;
```

- 위 쿼리의 실행 계획은 다음과 같다.

| id | table | type  |   key   | rows |                    extra                     |
|:--:|:-----:|:-----:|:-------:|:----:|:--------------------------------------------:|
| 1  |   e   | range | PRIMARY | 3000 | Using where; Using temporary; Using filesort |
| 1  |   s   |  ref  | PRIMARY |  10  |                     NULL                     |

1. `employees` 테이블을 `드라이빙 테이블` 로 선택해서 `s.emp_no BETWEEN 10001 AND 13000` 조건을 만족하는 레코드 `3,000건` 을 읽는다.
2. `salaries` 테이블을 조인했다. 이 때, 조인을 수행한 횟수는 `12,000(3000 * 4)번` 이다.
    - 참고) 확인할 수 없지만, 예제에서는 `employees` 테이블의 `1건` 당 `salaries` 테이블의 결과가 `4건` 인 것 같다.
3. 조인의 결과인 `12,000건의 레코드` 를 `임시테이블에 저장` 하고, `GROUP BY` 처리를 통해 `3,000건` 으로 줄였다.
4. `ORDER BY` 를 처리해서 `3,000건` 중에 `상위 10건` 만 최종적으로 반환한다.

### 지연된 조인을 사용한 쿼리

```mysql
SELECT e.*
FROM 
  (SELECT s.emp_no
   FROM salaries s
   WHERE s.emp_no BETWEEN 10001 AND 13000
   GROUP BY s.emp_no
   ORDER BY SUM(s.salary) DESC
   LIMIT 10) tp_s,
  employees e
WHERE e.emp_no = tp_s.emp_no;
```

- `salaries` 테이블에서 `WHERE`, `GROUP BY`, `ORDER BY`, `LIMIT` 를 모두 처리하고 그 결과를 `임시 테이블` 에 저장한다.
- `임시 테이블` 의 결과를 `employees` 테이블과 조인하도록 고친 것이다.
- 위 쿼리의 실행 계획은 다음과 같다.

| id |   table    |  type  |   key   | rows  |                    extra                     |
|:--:|:----------:|:------:|:-------:|:-----:|:--------------------------------------------:|
| 1  | <derived2> |  ALL   |  NULL   |  10   |                     NULL                     |
| 1  |     e      | eq_ref | PRIMARY |   1   |                     NULL                     |
| 2  |     s      | range  | PRIMARY | 56844 | Using where; Using temporary; Using filesort |

1. `salaries` 테이블에서는 `56,844건` 을 읽어야 한다고 나왔지만, 실제로는 `28,606건` 만 읽어서 `임시 테이블` 에
   저장하고, `GROUP BY` 를 통해 `3,000건` 으로 줄였다.
2. 그리고 `ORDER BY` 를 처리해 `상위 10건` 만 `임시 또는 파생 테이블(derived2)` 에 저장한다.
3. `임시 또는 파생 테이블(derived2)` 이 `드라이빙 테이블` 로 선택 되었고, `10건` 을 읽는다.
4. `employees` 테이블과 조인을 `10(10 * 1)번` 만 수행해서 결과를 반환한다.

### 지연된 쿼리 정리 및 주의사항

- `임시 또는 파생 테이블(derived2)` 를 한 번 더 사용하기 때문에 느리다고 예상할 수 있지만, 저장되는 레코드가 `10건` 밖에 되지 않으므로 메모리를 이용해 빠르게 처리된다.
- 조인의 횟수를 비교해보면, `지연된 조인` 으로 변경된 쿼리의 조인 횟수가 훨씬 적다는 사실을 알 수 있다.
- `지연된 조인` 의 원리를 정확히 이해하지 못한 상태로 지연된 쿼리를 작성하면, 오히려 역효과가 날 수도 있다.
- `OUTER JOIN`, `INNER JOIN` 에 대해 다음과 같은 조건이 갖춰져야만 `지연된 쿼리` 로 변경해서 사용할 수 있다.
    - `LEFT (OUTER) JOIN`
        - `드라이빙 테이블` 과 `드리븐 테이블` 은 `1:1` 또는 `N:1` 관계여야 한다.
    - `INNER JOIN`
        - `드라이빙 테이블` 과 `드리븐 테이블` 은 `1:1` 또는 `N:1` 관계여야 한다.
        - `드라이빙 테이블` 에 있는 레코드는 `드리븐 테이블` 에 모두 존재해야 한다.
        - `드라이빙 테이블` 은 `서브쿼리` 로 만들고, 이 `서브쿼리` 에 `LIMIT` 를 추가해도 최종 결과의 건수가 변하지 않는다는 보증을 해주는 조건이기 때문에 반드시 정확히 확인한 후, 적용해야
          한다.
- `지연된 조인` 은 `조인의 개수를 줄이는 것` 뿐만 아니라 `GROUP BY` 나 `ORDER BY` 처리가 `필요한 레코드의 전체 크기를 줄이는 역할도 한다.`
    - 위의 예제에서 `첫 번째 쿼리` 는 `employees` 테이블과 `salaries` 테이블의 모든 컬럼을 `임시 테이블` 에 저장하고, `GROUP BY` 를 해야한다.
    - 하지만 `지연된 조인` 으로 개선된 쿼리는 `salaries` 테이블 컬럼만 `임시 테이블` 에 저장하고, `GROUP BY` 를 수행하면 되기 때문에 `첫 번째 쿼리`
      보다는 `GROUP BY`, `ORDER BY` 용 버퍼를 더 적게 필요로 한다.

### 지연된 쿼리 관련해서 응용

```mysql
SELECT a.*, # temp_b 임시 테이블의 GROUP BY 결과를 노출해도 되고
FROM
  (SELECT b.a_id a_id
   FROM B b
   WHERE b.id < 0
   GROUP BY b.id
   ORDER BY b.id DESC
   LIMIT 10) temp_b,
  A a,
WHERE a.id = temp_b.a_id;
```

- 위의 쿼리 처럼 `1 : N` 관계에서 `1` 테이블 기준으로 페이지네이션을 처리할 때, `GROUP BY`, `ORDER BY`, `LIMIT` 쿼리를 사용하여 `지연된 쿼리` 를 사용할 수 있지 않을까?

## NESTED LOOP JOIN

```mysql
SELECT *
FROM A a
INNER JOIN B a ON a.id = b.id;
```

```mysql
SELECT *
FROM A a, B b
WHERE a.id = b.id;
```

- `Join` 이 일어나면 두 테이블의 인덱스 유무를 살핀다.
- 인덱스가 있는 테이블을 `드리븐(Driven)` 으로 둔다.
    - 드라이빙 테이블이 드리븐 테이블에 대해서 `풀 테이블 스캔` 을 하지 않아 더 빠르기 때문
    - 드리븐 테이블은 `인덱스 탐색(Index Seek)`, `인덱스 스캔(Index Scan)` 작업이 드라이빙 테이블 레코드 수 만큼 진행되기 때문에 최적화를
      위해 `인덱스가 있는 테이블을 드리븐으로 둔다.`
- `양 쪽 모두 인덱스가 있거나 모두 없는 경우`, 각 테이블의 레코드 건 수, 이외의 통계 정보등을 통해 드라이빙 테이블을 판단한다.
- 드라이빙 테이블이 100건이고 드리븐 테이블이 1000건일 경우 조인 횟수는 100 X 1000이고, 반대의 경우에도 어차피 1000 X 100으로 똑같지 않나라는
  의문이 들었다. 하지만 이거는 `인덱스가 없을 때나 해당된다.`
- 다른 블로그 글을 찾아보니 드라이빙 테이블의 대상 건수를 줄이는 이유는 `드라이빙 테이블의 추출 건수가 곧 드리븐 테이블의 액세스 반복 횟수` 가 되므로 `데이터가 더 적은 테이블이 드라이빙 테이블로 선정되어야
  한다.` 라고 설명이 되어 있었다.
    - 둘 다 인덱스가 있다면, `옵티마이저는 데이터 건수가 더 적은 테이블을 드라이빙 테이블로 선정한다.`

### 예시1

```markdown
A (드라이빙 테이블): 100,000건
B (드리븐 테이블): 3건
```

<img width="800" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/73b4c6a6-67c2-43c8-b5f5-c083dba76180">

- A 테이블은 `1건` 마다 `인덱스 탐색(Seek), 스캔(Scan)` 통해 B 테이블 인덱스을 `총 100,000번` 접근한다.
    - A 테이블의 데이터 수만큼 `인덱스 탐색(Seek), 스캔(Scan)` 을 반복한다.
    - A 테이블의 데이터 수는 `100,000건` 이다.
- B 테이블 인덱스들은 B 테이블에 `총 100,000` 번 접근한다.
- A 테이블과 B 테이블의 조인은 `총 200,000` 번의 접근이 일어난다.

### 예시2

```markdown
A (드라이빙 테이블): 3건
B (드리븐 테이블): 100,000건
```

<img width="800" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/c08ca53e-c222-4136-a0ad-867127bd93a8">

- A 테이블은 `1건` 마다 `인덱스 탐색(Seek), 스캔(Scan)` 통해 B 테이블 인덱스을 `총 3번` 접근한다.
    - A 테이블의 데이터 수만큼 `인덱스 탐색(Seek), 스캔(Scan)` 을 반복한다.
    - A 테이블의 데이터 수는 `3건` 이다.
    - `A1` 의 경우, `1번` 의 `인덱스 탐색(Seek)과 스캔(Scan)` 으로 `B1 ~ B29831` 을 접근했다.
    - `A2` 의 경우, `1번` 의 `인덱스 탐색(Seek)과 스캔(Scan)` 으로 `B9823 ~ B98231` 을 접근했다.
    - `A3` 의 경우, `1번` 의 `인덱스 탐색(Seek)과 스캔(Scan)` 으로 `B923 ~ B77321` 을 접근했다.
- B 테이블 인덱스들은 B 테이블에 `총 100,000` 번 접근한다.
- A 테이블과 B 테이블의 조인은 `총 100,003` 번의 접근이 일어난다.

## :book: Nested Loop Join 방식 특징 및 주의 사항

### 특징

- 주로 좁은 범위에 유리
- 순차적으로 처리하며, Random Access 위주
- 후행(Driven) 테이블에는 조인을 위한 인덱스가 생성되어 있어야 한다.

### 주의 사항

- 데이터를 랜덤으로 액세스하기 때문에 결과 집합이 많으면 수행속도가 저하됨
- 선행(Driving) 테이블의 크기가 작거나, Where절 조건을 통해 결과 집합을 제한할 수 있어야 한다.
- 조인 연결고리 인덱스가 없거나, 조인 집합을 구성하는 검색 조건이 조인 범위를 줄여주지 못할 경우 비효율적이다.

## :book: Application 레벨에서 Join을 지양해야 하는 이유

- 결합 쿼리는 장기적인 관점에서 `데이터 증가에 따라 통계 혹은 힌트의 변화로 옵티마이저가 동작하는 방식이 변경될 수 있기에 성능 저하의 주범` 이 될 수 있다. 또한, bulk data가 쌓이면 쌓일수록 메모리
  이슈도
  발생할 수 있다.

## :bookmark: 참고

- Real MySQL - 쿼리 최적화 JOIN
- [[Database] Nested Loop 최적화](https://insight-bgh.tistory.com/500)
- [[데이터베이스] SQL 튜닝 - 드라이빙 테이블](https://programming-workspace.tistory.com/67)
