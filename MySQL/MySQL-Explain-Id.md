# 실행 계획 - Id 컬럼

- 실행 계획에서 가장 왼쪽에 표시되는 `Id` 컬럼은 단위 SELECT 쿼리별로 부여되는 식별자 값이다.

- 하나의 SELECT 문장에서 여러개 테이블을 조인하면, 조인되는 테이블 개수만큼 실행 계획 레코드가 출력되지만, 같은 `Id`가 부여된다.

  ```sql
  EXPLAIN
  SELECT e.emp_no, e.first_name, s.from_date, s.salary
  FROM employees e, salaries s
  WHERE e.emp_no = s.emp_no
  LIMIT 10;
  ```

  | id  | select_type | ... |
  | :-: | :---------: | :-: |
  |  1  |     ...     | ... |
  |  1  |     ...     | ... |

<br>

- 쿼리 문장이 3개의 단위 SELECT 쿼리로 구성되어 있으면, 실행 계획의 각 레코드가 각기 다른 `Id`를 지닌다.

  ```sql
  EXPLAIN
  SELECT
  ( (SELECT COUNT(*) FROM employees) + (SELECT COUNT(*) FROM departments) ) AS total_count;
  ```

  | id  | select_type | ... |
  | :-: | :---------: | :-: |
  |  1  |     ...     | ... |
  |  2  |     ...     | ... |
  |  3  |     ...     | ... |
