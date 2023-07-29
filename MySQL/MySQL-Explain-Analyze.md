# 실행 계획 - Analyze

## Analyze

- `MySQL 8.0.18` 버전 이상부터 쿼리의 `실행 계획` 과 `단계별 소요된 시간 정보` 를 확인할 수 있는 `EXPLAIN ANALYZE` 기능이 추가됐다.
- `EXPLAIN ANALYZE` 명령은 항상 결과를 `TREE` 포맷으로 보여주기 때문에 `EXPLAIN` 명령에 `FORMAT` 옵션을 사용할 수 없다.
    - 참고) `MySQL 8.0` 버전부터는 `FORMAT` 옵션을 사용해 실행 계획의 표시 방법을 `JSON`, `TREE`, `Table` 형태로 선택할 수 있다.

## Analyze 사용시, 쿼리 분석 방법

- 들여쓰기가 같은 레벨에서는 상단에 위치한 라인이 먼저 실행된다.
- 들여쓰기가 다른 레벨에서는 가장 안쪽에 위치한 라인이 먼저 실행된다.

## Analyze 사용

### 첫 번째 예시

```mysql
explain analyze
select p.id
from post p
where p.member_id in (1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
and p.deleted_datetime is null
and p.created_datetime >= '2023-07-26 00:00:00'
and p.id > 0
order by p.id asc
limit 200;
```

- 위의 쿼리를 `EXPLAIN ANALYZE` 명령을 통해 분석해보면 실행 순서는 다음과 같다.

```
-> Limit: 200 row(s)  (cost=3.47 rows=1.07) (actual time=0.0174..0.0363 rows=16 loops=1)
    -> Filter: ((p.member_id in (1,2,3,4,5,6,7,8,9,10)) and (p.deleted_datetime is null) and (p.created_datetime >= TIMESTAMP'2023-07-26 00:00:00') and (p.id > 0))  (cost=3.47 rows=1.07) (actual time=0.0166..0.0342 rows=16 loops=1)
        -> Index range scan on p using PRIMARY over (0 < id)  (cost=3.47 rows=16) (actual time=0.0139..0.0274 rows=16 loops=1)
```

- 위의 결과를 토대로 실행 순서를 해석해보면

```markdown
1. Index range scan on p using PRIMARY
2. Filter
3. Limit
```

```markdown
1. Post 테이블의 PRIMARY 키를 통해 Index Range Scan 수행한다.
2. (p.member_id in (1,2,3,4,5,6,7,8,9,10)) and (p.deleted_datetime is null) and (p.created_datetime >=
   TIMESTAMP'2023-07-26 00:00:00') and (p.id > 0) 조건에 일치하는 건만 가져온다.
3. 2번의 결과에서 200건만 가져온다.
```

## 주의할 점

- `EXPLAIN ANALYZE` 명령은 `실제 쿼리를 실행하고`, `사용된 실행 계획과 소요된 시간` 을 보여주는 것이다. 이에 따라 실행 계획이 아주 나쁜 경우라면, `EXPLAIN` 명령으로 실행 계획만
  확인해서 쿼리를 어느정도 개선한 다음에 `EXPLAIN ANALYZE` 명령을 실행하는 것이 좋다.

## 참고

- Real MySQL 8.0 - 개발자와 DBA를 위한 MySQL 실전 가이드 
