# MySQL 쿼리 최적화 - WHERE 비교 조건 사용시 주의사항

## NULL 비교

- MySQL에서는 `NULL` 값이 포함된 레코드도 인덱스로 관리된다.
- 인덱스에서는 `NULL` 을 하나의 값으로 인정해서 관리한다는 것을 의미한다.
- `NULL` 을 비교할 때는 `ISNULL()` 함수보다 `IS NULL` 연산자를 사용하길 권장한다.

```mysql
# 인덱스를 사용하지 못한다.
WHERE ISNULL(column) = 1;

# 인덱스를 사용하지 못한다.
WHERE ISNULL(column) = true;
```

## 문자열이나 숫자열 비교

- 그 타입에 맞는 상숫값을 사용할 것을 권장한다.

```mysql
# column_1 타입이 '숫자열' 인 경우
WHERE column_1 = 1;

# column_2 타입이 '문자열' 인 경우
WHERE column_2 = 'apple';
```

## 날짜 비교

- `DATE`, `DATETIME` 인 경우, MySQL이 내부적으로 문자열 값을 `DATE`, `DATETIME` 타입의 값으로 변환해서 비교를 수행한다.
- `DATE`, `DATETIME` 타입의 컬럼을 변경하지 말고, 상수를 변경하는 형태로 조건을 사용하는 것이 좋다.

```mysql
# 인덱스를 사용하지 못함
WHERE DATE_FORMAT(column_1, '%Y-%m-%d') > '2011-07-23';

# 상수를 변경하는 형태 조건으로 변경하면, 인덱스 사용 가능
WHERE column_1 > DATE_SUB('2011-07-23', INTERVAL 1 YEAR);
```

- `DATETIME` 타입의 값을 `DATE` 타입으로 만들지 않고, 그냥 비교하면 MySQL 서버가 `DATE` 타입의 값을 `DATETIME` 으로 변환해서 같은 타입을 만든 다음에 비교를 수행한다.
- 컬럼이 `DATETIME` 타입이라면, `FROM_UNIXTIME()` 함수를 이용해 `TIMESTAMP` 값을 `DATETIME` 타입으로 만들어서 비교해야 한다.
- 컬럼이 `TIMESTAMP` 타입이라면, `UNIX_TIMESTAMP()` 함수를 이용해 `DATETIME` 값을 `TIMESTAMP` 로 변환해서 비교해야 한다.

## Short-Circuit Evaluation

- 여러 개의 표현식이 `AND` 또는 `OR` 논리 연산자로 연결된 경우, 선행 표현식의 결과에 따라 후행 표현식을 평가할지 말지 결정하는 최적화를 `Short-Circuit Evaluation` 이라고 한다.
- `WHERE` 절의 조건중에서 인덱스를 사용할 수 있는 조건이 있다면, `Short-Circuit Evaluation` 과는 무관하게 MySQL 서버는 그 조건을 가장 최우선으로 사용한다. 그래서 `WHERE`
  조건절에 **`나열된 조건의 순서가 인덱스 사용 여부를 결정하지는 않는다.`**
    1. MySQL 서버는 `WHERE` 절에서 `인덱스를 사용할 수 있는 조건을 먼저 평가한다.`
    2. 그리고 나서 `WHERE` 절에 `인덱스를 사용한 컬럼을 제외하고, 나머지 조건을 순서대로 평가한다.`

- 아래의 쿼리들을 해석해보자

```mysql
SELECT *
FROM employees e
WHERE e.last_name = 'Aamodt' # 2번
  AND e.first_name = 'Matt' # 1번
  AND MONTH(e.birth_date) = 1; # 3번
```

- 위의 `employees` 테이블에는 `idx_firstname(first_name)` 인덱스가 생성되어 있다.
- 위의 쿼리에서 `WHERE` 절에 나열 순서와 상관없이 인덱스를 통해 `first_name` 조건을 평가한다.
- 그리고 나서 `first_name` 조건을 제외한 `last_name` 조건을 평가하고, `birth_date` 조건을 평가한다.
    - 즉, `last_name`, `birth_date` 는 인덱스를 사용하지 못하기 때문에 `WHERE` 절에 나열된 순서대로 조건을 평가한다.
- 복잡한 연산 또는 다른 테이블의 레코드를 읽어야 하는 서브쿼리 조건들은 `WHERE` 절의 뒤쪽으로 배치하는 것이 성능상 도움이 될 것이다.
- 가장 핵심은 `WHERE` 절에서 인덱스를 사용할 수 있는 조건이라면, `WHERE` 절 어느 위치에 나열되든지 그 순서에 관계없이 가장 먼저 평가된다.

## 참고

- 개발자와 DBA를 위한 Real MySQL
