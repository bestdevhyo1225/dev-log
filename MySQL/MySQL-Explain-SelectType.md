# 실행 계획 - select_type 컬럼

- `SIMPLE`

- `PRIMARY`

- `UNION`

- `DEPENDENT UNION`

- `UNION RESULT`

- `SUBQUERY`

- `DEPENDENT SUBQUERY`

- `DERIVED`

- `UNCACHEABLE SUBQUERY`

- `UNCACHEABLE UNION`

<br>

## SIMPLE

UNION이나 서브 쿼리를 사용하지 않는 단순한 SELECT 쿼리인 경우, 해당 쿼리 문장의 select_type은 `SIMPLE`로 표시된다. (쿼리에 조인이 포함된 경우에도 마찬가지)

<br>

## PRIMARY

UNION이나 서브 쿼리가 포함된 SELECT 쿼리의 실행 계획에서 가장 바깥쪽(Outer)에 있는 단위 쿼리는 select_type이 `PRIMARY`로 표시되며, select_type이 `PRIMARY`인 단위 SELECT 쿼리는 하나만 존재한다.

<br>

## 참고

- 개발자와 DBA를 위한 Real MySQL
