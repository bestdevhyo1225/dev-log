# MySQL 쿼리 최적화 - WHERE(문자열이나 숫자 비교)

특정 필드의 인덱스가 걸려있고, 그 필드를 Where절에서 사용하려면 주의할 점이 있기 때문에 정리하려고 함

<br>

## 테이블의 필드 타입은 Int 또는 BigInt 이면서, 인덱스가 걸려있는 경우

> emp_no 컬럼 타입이 Int 또는 BigInt 이다. 또한, 비교하는 상수 값도 같은 타입이기때문에 인덱스가 적용된다.

```sql
-- employees 테이블 (emp_no : int or bigint type)
-- emp_on 컬럼에는 인덱스가 걸려있음
SELECT *
FROM employees
WHERE emp_no = 1;
```

> emp_no 컬럼 타입이 Int 또는 BigInt 이다. 또한, 비교하는 상수 값은 문자열 상수값이지만, 숫자로 타입을 변환해서 비교를 수행하기 때문에 특별히 성능 저하는 발생하지 않는다.

```sql
-- employees 테이블 (emp_no : int or bigint type)
-- emp_on 컬럼에는 인덱스가 걸려있음
SELECT *
FROM employees
WHERE emp_no = '1';
```

<br>

## 테이블의 필드 타입은 Varchar 이면서, 인덱스가 걸려있는 경우

> first_name 컬럼 타입 Varchar 이며, 인덱스가 걸려있다. 이 때 비교하는 값이 문자열 상수값이면, 문제없이 인덱스가 적용된다.

```sql
-- employees 테이블 (first_name : varchar)
-- first_name 컬럼에는 인덱스가 걸려있음
SELECT *
FROM employees
WHERE first_name = 'Hyo';
```

> **가장 중요!!!** <br>
> first_name 과 비교하는 값이 상수 값이면, first_name 컬럼 문자열을 숫자로 변환해서 비교를 수행한다. **하지만 first_name 컬럼 타입의 변환이 필요하기 때문에 인덱스를 사용하지 못한다.**

```sql
-- employees 테이블 (first_name : varchar)
-- first_name 컬럼에는 인덱스가 걸려있음
SELECT *
FROM employees
WHERE first_name = 1234;
```

<br>

## 참고

- 개발자와 DBA를 위한 Real MySQL
