# MySQL - 페이지네이션 쿼리

## 데이터 개수 기반 방식

### 범위 조건 컬럼 값 순서와 식별자 컬럼의 값 순서가 동일한 경우

- created_at, id
    - 생성일과 ID는 동일한 순서대로 생성된다.
    - created_at, id 순서대로 복합 인덱스 생성되어 있다고 가정한다.
- 아래의 쿼리 처럼 비교적 간단하게 N회차 쿼리를 작성할 수 있다.

| created_at          | id |
|---------------------|----|
| 2024-01-01 00:00:00 | 1  |
| 2024-01-01 00:00:01 | 2  |
| 2024-01-01 00:00:01 | 3  |
| 2024-01-01 00:00:01 | 4  |
| 2024-01-01 00:00:02 | 5  |
| 2024-01-01 00:00:02 | 6  |
| 2024-01-01 00:00:02 | 7  |
| 2024-01-01 00:00:03 | 8  |
| 2024-01-01 00:00:03 | 9  |
| 2024-01-01 00:00:03 | 10 |

- 1회차 조회 쿼리는 아래와 같다.

```mysql
SELECT *
FROM A a
WHERE a.created_at >= ${시작 날짜}
  AND a.created_at < ${종료 날짜}
ORDER BY a.created_at, a.id
LIMIT 30;
```

- 여기서 N회차 쿼리를 작성하려면 아래와 같이 작성하면 된다.

```mysql
SELECT *
FROM A a
WHERE a.created_at >= ${이전 마지막 데이터의 날짜 값}
  AND a.created_at < ${종료 날짜}
  AND a.id > ${이전 마지막 데이터의 ID 값}
ORDER BY a.created_at, a.id
LIMIT 30;
```

### 범위 조건 컬럼 값 순서와 식별자 컬럼의 값 순서가 동일하지 않은 경우

- finished_at, id
    - 종료일과 ID는 동일한 순서대로 생성되는 것을 보장되지 않는다.
    - finished_at, id 순서대로 복합 인덱스 생성되어 있다고 가정한다.

| finished_at         | id |
|---------------------|----|
| 2024-01-01 00:00:00 | 5  |
| 2024-01-01 00:00:01 | 1  |
| 2024-01-01 00:00:01 | 2  |
| 2024-01-01 00:00:01 | 3  |
| 2024-01-01 00:00:02 | 8  |
| 2024-01-01 00:00:02 | 9  |
| 2024-01-01 00:00:03 | 4  |
| 2024-01-01 00:00:03 | 6  |
| 2024-01-01 00:00:03 | 7  |
| 2024-01-01 00:00:03 | 10 |

- 여기서 올바르게 N회차 쿼리를 작성하려면 아래와 같이 작성하면 된다.

```mysql
SELECT *
FROM A a
WHERE ((a.finished_at = '2024-01-01 00:00:02' AND a.id > 8)
    OR (a.finished_at > '2024-01-01 00:00:02' AND a.finished_at < '2024-01-02 00:00:00'))
ORDER BY a.finished_at, id
LIMIT 5;
```

```mysql
SELECT *
FROM A a
WHERE ((a.finished_at = ${이전 마지막 데이터의 날짜 값} AND a.id > ${이전 마지막 데이터의 ID 값})
    OR (a.finished_at > ${이전 마지막 데이터의 날짜 값} AND a.finished_at < ${종료 날짜}))
ORDER BY a.finished_at, id
LIMIT 5;
```

- `(a.finished_at = ${이전 마지막 데이터의 날짜 값} AND a.id > ${이전 마지막 데이터의 ID 값})`
    - 이전 마지막 데이터의 날짜 값과 같으면서, 이전 마지막 데이터의 ID 값을 찾거나
- `(a.finished_at > ${이전 마지막 데이터의 날짜 값} AND a.finished_at < ${종료 날짜}))`
    - 위의 조건에 대한 데이터가 없으면, 이전 마지막 데이터의 날짜 값보다 크고 종료 날짜보다 작은 데이터를 찾는다.
