# MySQL 인덱스 비교조건의 종류와 효율성

## 복합 인덱스 순서에 따른 비교조건

### 예시 쿼리

```mysql-sql
SELECT *
FROM dept_emp
WHERE dept_no = 'd002'
AND emp_no >= 10114;
```

### 인덱스 종류

```
케이스 A: (dept_no, emp_no)   
케이스 B: (emp_no, dept_no)   
```

- 인덱스 케이스 A의 `dept_no`, `emp_no` 컬럼은 모두 `작업 범위 결정 조건` 으로 수행되었다.
- 인덱스 케이스 B의 `emp_no`, `dept_no` 컬럼에서는 `emp_no` 은 `작업 범위 결정 조건` 으로 수행되었지만, `dept_no` 은 `필터링 조건(체크 조건)` 으로 수행되었다.

### 정리

- 작업 범위를 결정하는 조건이 많으면 많을수록 쿼리의 처리 성능을 높인다.
- 필터링 조건(체크 조건)은 많다고 해서 쿼리의 처리 성능을 높이지는 못한다. (최종적으로 가져오는 레코드는 작게 만들지 몰라도..)

## 인덱스를 타지 않는 조건

- `작업 범위 결정 조건` 으로 사용할 수 없다는 것을 의미하며, 경우에 따라서는 `필터링 조건(체크 조건)` 으로 인덱스를 사용할 수 있다.

### `Not Equal` 로 비교된 경우

```mysql-sql
WHERE column <> ?
WHERE column NOT IN (?, ?)
WHERE column IS NOT NULL
```

### `LIKE '%?'` (뒷 부분 일치) 형태로 문자열 패턴이 비교된 경우

```mysql-sql
WHERE column LIKE '%?'
WHERE column LIKE '_?'
WHERE column LIKE '%?%'
```

### 스토어드 함수나 다른 연산자로 인덱스 컬럼이 변형된 경우

```mysql-sql
WHERE SUBSTRING(column, 1.1) = '?'
```

### NOT-DETERMINISTIC 속성의 스토어드 함수가 비교 조건에 사용된 경우

```mysql-sql
WHERE column = deterministic_func()
```

### 데이터 타입이 서로 다른 비교 (인덱스 칼럼의 타입을 변환해야 비교가 가능한 경우)

```mysql-sql
WHERE char_column = 10
```

### 문자열 데이터 타입의 콜레이션이 다른 경우

```mysql-sql
WHERE utf8_bin_char_column = euckr_bin_char_column
```

## MySQL에서 NULL 값

- 다른 DBMS에서는 NULL 값은 인덱스에 저장되지 않지만, MySQL에서 NULL 값은 인덱스로 관리된다.
- `WHERE` 절에서 `IS NULL 조건` 은 `작업 범위 결정 조건` 으로 인덱스를 사용한다.

```mysql-sql
WHERE column IS NULL
```

## 인덱스를 사용하는 경우, 사용하지 못한 경우

```mysql-sql
INDEX ix_test(column_1, column_2, column_3, column_4, ... , column_n)
```

### 작업 범위 결정 조건으로 인덱스를 사용하지 않는 경우

- column_1 컬럼에 대한 조건이 없는 경우
- column_1 컬럼의 비교 조건이 위의 `인덱스 사용 불가 조건` 중 하나인 경우

### 작업 범위 결정 조건으로 인덱스를 사용하는 경우 

- column_1 ~ column_(i-1) 컬럼까지 `Equal(=)` 또는 `IN 절` 로 비교하는 경우
  - `i-1` 은 2 보다는 크고 n 보다는 작은 임의의 값을 의미
- column_i 컬럼에 대한 다음 연산자 중 하나로 비교
  - `Equal(=)` 또는 `IN 절`
  - `>` 또는 `<`
  - `LIKE '?%'` (좌측 일치 패턴)

### 위의 ix_test 인덱스를 사용하는 경우들을 확인해보자

- 아래의 column_1 컬럼은 인덱스를 사용할 수 없다.

```mysql-sql
WHERE column_1 <> ?
```

- 아래의 column_1, column_2 컬럼들은 `작업 범위 결정 조건` 으로 사용된다.

```mysql-sql
WHERE column_1 = 1 AND column_2 > 10
```

- 아래의 column_1, column_2, column_3 컬럼들은 모두 `작업 범위 결정 조건` 으로 사용된다.

```mysql-sql
WHERE column_1 IN (1, 2)
AND column_2 = 2
AND column_3 <= 10
```

- 아래의 column_1, column_2, column_3 컬럼들은 `작업 범위 결정 조건` 으로 사용되고, column_4 컬럼은 `필터링 조건(체크 조건)` 으로 사용된다.

```mysql-sql
WHERE column_1 = 1
AND column_2 = 2
AND column_3 IN (10, 20, 30)
AND column_4 <> 100
```

- 아래의 column_1, column_2, column_3, column_4 컬럼들은 모두 `작업 범위 결정 조건` 으로 사용된다.

```mysql-sql
WHERE column_1 = 1
AND column_2 = (2, 4)
AND column_3 = 30
AND column_4 LIKE '문자열%'
```

- 아래의 column_1, column_2, column_3, column_4, column_5 컬럼들은 모두 `작업 범위 결정 조건` 으로 사용된다.

```mysql-sql
WHERE column_1 = 1
AND column_2 = (2, 4)
AND column_3 = 30
AND column_4 = '문자열'
AND column_5 = '서울'
```

## 참고

- 개발자와 DBA를 위한 Real MySQL
