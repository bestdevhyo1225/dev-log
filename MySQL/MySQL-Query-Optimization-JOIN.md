## MySQL 쿼리 최적화 JOIN

<br>

### :book: JOIN의 순서와 인덱스

우선 `인덱스 레인지 스캔`으로 레코드를 읽는 작업은 다음과 같다.

1. 인덱스 조건을 만족하는 값이 지정된 위치를 찾는다. -> `인덱스 탐색(Index Seek)`

2. 1번에서 탐색된 위치에서 필요한 만큼 인덱스를 쭉 읽는다. -> `인덱스 스캔(Index Scan)`

3. 2번에서 읽어들인 `인덱스 키`와 `레코드 주소`를 이용해 레코드가 저장된 페이지를 가져오고 `최종 레코드`를 읽어온다.

조인 작업에서 `드라이빙 테이블(Driving)`을 읽을 때와 `드리븐 테이블(Driven)`을 읽을 때를 확인해 보면 다음과 같다.

* `드라이빙 테이블 (Driving Table)`

    * `인덱스 탐색` 작업을 1번 수행하고, 그 이후부터는 `인덱스 스캔`만 실행하면 된다.

* `드리븐 테이블 (Driven Table)`

    * `인덱스 탐색`과 `인덱스 스캔` 작업을 드라이빙 테이블에서 읽은 레코드 건 수 만큼 반복한다.

따라서 `드리븐 테이블(Driven Table)`을 읽는 것이 훨씬 큰 부하를 차지한다. 그래서 `옵티마이저`는 항상 드라이빙 테이블이 아닌 **드리븐 테이블(Driven Table) 을 최적으로 읽을 수 있게 실행 계획을 수립한다.** ( MySQL JOIN은 Nested Loop를 따르기 때문에...)

<br>

### :book: 예제를 보면서 확인해보자!

```sql
SELECT *
FROM employees AS e, dept_emp AS de
WHERE e.emp_no = de.emp_no;
```

`두 컬럼 모두 인덱스가 있는 경우`

* 각 테이블의 통계 정보에 있는 레코드 건수에 따라 employees가 드라이빙 테이블이 될 수도 있고, dept_emp 테이블이 드라이빙 테이블로 선택될 수도 있다. 어느 쪽 테이블이 드라이빙 테이블이 되든 옵티마이저가 선택하는 방법이 최적일 때가 많다.

`employees.emp_no 에만 인덱스가 있는 경우`

* 이 때 옵티마이저는 employees 테이블을 드리븐 테이블로 선택해서 인덱스 스캔을 진행하고, dept_emp 테이블을 드라이빙 테이블로 선택해서 풀 스캔을 진행한다. (드리븐 테이블 읽는 것을 최소화 해야 하기 때문에 인덱스 스캔)

`dept_emp.emp_no 에만 인덱스가 있는 경우`

* 이 때 옵티마이저는 dept_emp 테이블을 드리븐 테이블로 선택해서 인덱스 스캔을 진행하고, employees 테이블을 드라이빙 테이블로 선택해서 풀 스캔을 진행한다. (드리븐 테이블 읽는 것을 최소화 해야 하기 때문에 인덱스 스캔)

`두 컬럼 모두 인덱스가 없는 경우`

* 어떤 테이블을 선택하더라도 드리븐 테이블의 풀 스캔은 발생하기 때문에 옵티마이저가 적절히 드라이빙 테이블을 선택한다. 단 레코드 건수가 적은 테이블을 드리븐 테이블로 선택하는 것이 효율적이다. 또한 드리븐 테이블을 읽을 때 조인 버퍼가 사용되기 때문에 실행 계획의 Extra 칼럼에 "Using join buffer" 가 표시된다.

<br>

### :bookmark: 참고

* Real MySQL - 쿼리 최적화 JOIN