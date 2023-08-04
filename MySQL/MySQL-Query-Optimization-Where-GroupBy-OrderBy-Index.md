# MySQL 쿼리 최적화 - WHERE, GROUP BY, ORDER BY 절의 인덱스 사용

## 인덱스를 사용하기 위한 기본 규칙

- 인덱스된 컬럼의 값 자체를 변환하지 않고, 그대로 사용한다는 조건을 만족해야한다.
- SQL을 작성할 때는 데이터의 타입에 맞춰서 비교 조건을 사용하길 권장한다.

## WHERE 절의 인덱스 사용

- `WHERE` 조건이 인덱스를 사용하는 방법은 크게 `작업 범위 결정 조건` 과 `체크 조건` 의 두가지 방식으로 구분할 수 있다.
- `작업 범위 결정 조건` 은 `=(동등 비교 조건)`, `IN` 으로 구성된 조건에 사용된 컬럼들이 인덱스의 컬럼 구성과 좌측에서부터 비교했을때 얼마나 일치하는가에 따라 달라진다.
- `WHERE` 조건절의 순서에 나열된 조건들의 순서는 실제 인덱스의 사용 여부와 무관하다.
- `작업 범위 결정 조건` 으로 사용되지 못하는 컬럼은 `체크 조건` 으로 사용된다.

### 작업 범위 결정 조건, 체크 조건

```mysql
CREATE INDEX idx_test_01 ON test (col_1, col_2, col_3, col_4);
```

```mysql
SELECT t.*
FROM test t
WHERE t.col_2 = ?
AND t.col_4 = ?
AND t.col_3 > ?
AND t.col_1 = ?
```

- 위의 쿼리에서는 `col_3` 의 조건이 `동등 비교 조건` 이 아닌 `범위 비교 조건` 이므로 뒤 컬럼인 `col_4` 는 `작업 범위 결정 조건` 으로 사용되지 못하고, `체크 조건` 으로 사용된다.
- 인덱스 순서상 `col_4` 의 직전 컬럼인 `col_3` 가 `동등 비교 조건` 이 아니라 `범위 비교 조건` 으로 사용됐기 때문에 `col_3` 까지는 인덱스를 사용하지만, `col_4` 는 인덱스를 사용할
  수 없다.
    - 즉, `범위 비교 조건` 으로 사용되는 컬럼이 있으면, `인덱스 순서상 뒤에 있는 컬럼들은 인덱스를 타지 않는다.`
- 인덱스의 순서를 `(col_1, col_2, col_3, col_4)` 에서 `(col_1, col_2, col_4, col_3)` 로 변경하면, 4개 컬럼이 모두 인덱스를 사용한다.
- `col_3` 컬럼의 조건을 `IN` 으로 풀어내는 것이 가능하다면, 4개 컬럼이 모두 인덱스를 사용한다.
- `WHERE` 조건에 `OR` 연산자가 있다면 주의해야 한다.

### OR 연산자 주의사항

```mysql
CREATE INDEX idx_employees_01 ON employees (first_name);
```

```mysql
SELECT e.*
FROM employees e
WHERE e.first_name = 'Kebin'
   OR e.last_name = 'Poly';
```

- 두 조건이 `AND` 연산자로 연결됐다면, `first_name` 의 인덱스를 사용하겠지만, `OR` 연산자로 연결됐기 때문에 옵티마이저는 `풀 테이블 스캔` 을 선택할 수 밖에 없다.
- `풀 테이블 스캔(last_name) + 인덱스 레인지 스캔(first_name)` 작업량 보다는 `풀 테이블 스캔` 한 번이 더 빠르기 때문이다.
- `first_name` 과 `last_name` 에 각각의 컬럼이 있다면, `index_merge` 접근 방법으로 실행할 수 있다.
    - `풀 테이블 스캔` 보다는 빠르지만, 여전히 `제대로 된 인덱스 하나를 레인지 스캔하는 것보다는 느리다.`

## GROUP BY 절의 인덱스 사용

- `GROUP BY` 절에 명시된 컬럼이 인덱스 컬럼의 순서와 위치가 같아야 한다.
- 인덱스를 구성하는 컬럼 중에서 뒤쪽에 있는 컬럼은 `GROUP BY` 절에 명시되지 않아도 인덱스를 사용할 수 있다.
- 인덱스 앞쪽에 있는 컬럼이 `GROUP BY` 절에 명시되어 있지 않으면, 인덱스를 사용할 수 없다.
- `WHERE` 조건절과는 달리 `GROUP BY` 절에 명시된 컬럼이 하나라도 인덱스에 없으면, `GROUP BY` 절은 전혀 인덱스를 사용하지 못한다.

### 인덱스를 사용하지 못하는 경우

```mysql
CREATE INDEX idx_test_01 ON test (col_1, col_2, col_3, col_4);
```

```mysql
GROUP BY col_2, col_1;
GROUP BY col_1, col_3, col_2;
```

- 인덱스를 구성하는 컬럼의 순서와 일치하지 않는다.

```mysql
GROUP BY col_1, col_3;
```

- `col_3` 컬럼 앞에 `col_2` 컬럼이 명시되어 있지 않아서 인덱스를 사용할 수 없다.

```mysql
GROUP BY col_1, col_2, col_3, col_4, col_5;
```

- `col_5` 컬럼은 인덱스로 구성되어 있지 않아서 인덱스를 사용할 수 없다.

### 인덱스를 사용하는 경우

```mysql
CREATE INDEX idx_test_01 ON test (col_1, col_2, col_3, col_4);
```

```mysql
GROUP BY col_1;
GROUP BY col_1, col_2;
GROUP BY col_1, col_2, col_3;
GROUP BY col_1, col_3, col_3, col_4;
```

```mysql
WHERE col_1 = ? GROUP BY col_2, col_3;
WHERE col_1 = ? GROUP BY col_1, col_2, col_3;
WHERE col_1 = ? AND col_2 = ? GROUP BY col_3, col_4;
WHERE col_1 = ? AND col_2 = ? AND col_3 = ? GROUP BY col_4;
```

## ORDER BY 절의 인덱스 사용

- `ORDER BY` 절의 컬럼들이 인덱스에 정의된 컬럼의 왼쪽부터 일치해야 하는 것에는 변함이 없다.

### 인덱스를 사용하지 못하는 경우

```mysql
CREATE INDEX idx_test_01 ON test (col_1, col_2, col_3, col_4);
```

```mysql
ORDER BY col_2, col_3;
```

- `col_1` 컬럼이 명시되지 않아서 사용할 수 없다.

```mysql
ORDER BY col_1, col_3, col_2;
```

- 인덱스를 구성하는 컬럼의 순서와 일치하지 않는다.

```mysql
ORDER BY col_1, col_2 desc, col_3;
```

- 다른 컬럼은 모두 오름차순인데, `col_2` 컬럼만 내림차순이라 인덱스를 사용할 수 없다.
- 인덱스를 사용하려면, `(col_1 asc, col_2 desc, col_3 asc, col_4 asc)` 로 정의해야 한다.

```mysql
ORDER BY col_1, col_3;
```

- `col_2` 컬럼이 명시되지 않아서 사용할 수 없다.

```mysql
ORDER BY col_1, col_2, col_3, col_4, col_5;
```

- `col_5` 컬럼은 인덱스로 구성되어 있지 않아서 인덱스를 사용할 수 없다.

```mysql
WHERE col_1 > ? ORDER BY col_2, col_3;
```

- `col_1` 동등 비교 조건이 아니라 범위 비교 조건으로 검색 되었기 때문에 `ORDER BY` 절은 인덱스를 사용할 수 없다.
    - `WHERE` 절의 `col_1` 은 인덱스를 사용한다.
    - 별도의 정렬 처리 과정(`Using filesort`) 을 거쳐 정렬을 수행한다.
- `WHERE`, `ORDER BY` 절을 모두 인덱스로 처리하기에는 불가능하다.

```mysql
WHERE col_1 = ? ORDER BY col_3, col_4;
```

- `col_2` 컬럼이 빠졌기 때문에 `ORDER BY` 절은 인덱스를 사용할 수 없다.
    - `WHERE` 절의 `col_1` 은 인덱스를 사용한다.
    - 별도의 정렬 처리 과정(`Using filesort`) 을 거쳐 정렬을 수행한다.
- `WHERE`, `ORDER BY` 절을 모두 인덱스로 처리하기에는 불가능하다.

```mysql
WHERE col_1 IN (?, ?, ?, ?, ?) ORDER BY col_2;
```

- `col_1` 컬럼이 빠졌기 때문에 `ORDER BY` 절은 인덱스를 사용할 수 없다.
    - `WHERE` 절의 `col_1` 은 인덱스를 사용한다.
    - 별도의 정렬 처리 과정(`Using filesort`) 을 거쳐 정렬을 수행한다.
- `WHERE`, `ORDER BY` 절을 모두 인덱스로 처리하기에는 불가능하다.

### 인덱스를 사용하는 경우

```mysql
CREATE INDEX idx_test_01 ON test (col_1, col_2, col_3, col_4);
```

```mysql
ORDER BY col_1;
ORDER BY col_1, col_2;
ORDER BY col_1, col_2, col_3;
ORDER BY col_1, col_3, col_3, col_4;
```

```mysql
WHERE col_1 = ? ORDER BY col_2, col_3;
WHERE col_1 = ? ORDER BY col_1, col_2, col_3;
WHERE col_1 = ? AND col_2 = ? ORDER BY col_3, col_4;
WHERE col_1 = ? AND col_2 = ? AND col_3 = ? ORDER BY col_4;
# col_1, col_2, col_3 순서대로 모두 명시 되었기 때문에 WHERE, ORDER BY 절 모두 인덱스를 사용한다.
WHERE col_1 > ? ORDER BY col_1, col_2, col_3;
```

## WHERE 조건과 ORDER BY 또는 GROUP BY 절의 인덱스 사용

- `WHERE` 절과 `ORDER BY` 절이 동시에 같은 인덱스를 사용한다.
    - 하나의 인덱스에 연속해서 포함돼 있을 때 이 방식으로 인덱스를 사용할 수 있다.
    - 나머지 2가지 방식보다 훨씬 빠르 성능을 보이기 때문에 가능하다면, 이 방식으로 처리할 수 있게 쿼리를 튜닝하거나, 인덱스를 생성하는 것이 좋다.
- `WHERE` 절만 인덱스를 사용한다.
    - `ORDER BY` 절은 인덱스를 이용한 정렬이 불가능하다.
    - 인덱스를 통해 검색된 결과 레코드를 `별도의 정렬 처리 과정(Using Filesort)` 을 거쳐 정렬을 수행한다.
    - `WHERE` 절의 조건에 일치하는 레코드의 건수가 많지 않을 때 효율적인 방식이다.
- `ORDER BY` 절만 인덱스를 사용한다.
    - `WHERE` 절은 인덱스를 사용하지 못한다.
    - `ORDER BY` 절의 순서대로 인덱스를 읽으면서 레코드 한 건씩 `WHERE` 절의 조건에 일치하는지 비교하고, 일치하지 않을 때는 버리는 형태로 처리한다.
    - 주로 `아주 많은 레코드를 조회해서 정렬` 해야 할 때는 이런 형태로 튜닝하기도 한다.

## ORDER BY, GROUP BY 절의 인덱스 사용

- `하나의 인덱스` 를 사용해서 처리되려면, `GROUP BY` 절에 명시된 컬럼과 `ORDER BY` 절에 명시된 컬럼의 순서와 내용이 모두 같아야 한다.
- `GROUP BY` 절과 `ORDER BY` 절이 같이 사용된 쿼리라면, `둘 중 하나라도 인덱스를 사용할 수 없을 때` 는 `둘 다 인덱스를 사용하지 못한다.`
- 즉, `GROUP BY` 절에서는 인덱스는 사용할 수 있지만, `ORDER BY` 절에서는 인덱스를 사용할 수 없을 때는 `GROUP BY`, `ORDER BY` 절 모두 인덱스를 사용하지 못한다.
- `MySQL 5.7` 버전까지는 `GROUP BY` 절을 사용하면 `GROUP BY` 컬럼에 대한 정렬까지 함께 수행하는 것이 기본 작동 방식이었다.
- `MySQL 8.0` 버전부터는 `GROUP BY`, `ORDER BY` 절을 모두 명시해야 `그루핑과 정렬을 수행한다.`

## WHERE 조건과 GROUP BY, ORDER BY 절의 인덱스 사용

- `WHERE`, `GROUP BY`, `ORDER BY` 절이 모두 포함된 쿼리가 인덱스를 판단하는 방법은 다음과 같다.

<img width="777" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/77670638-8978-4019-940a-382af32ccc79">

## 참고

- Real MySQL 8.0 - 개발자와 DBA를 위한 MySQL 실전 가이드 
