# MySQL Left Join 주의 사항

## 작성하고자 하는 의도에 따라 ON, WHERE 절에 조건을 잘 명시해야 한다.

### user, user_coupon 테이블의 COUNT

|    테이블 명    |    COUNT    |
|:-----------:|:-----------:|
|    user     | 30,000 rows |
| user_coupon | 3,000 rows  |

### user_coupon 테이블

| coupon_id |   COUNT    |
|:---------:|:----------:|
|     1     | 1,000 rows |
|     2     | 1,000 rows |
|     3     | 1,000 rows |

### ON절에서 coupon_id = 3만 가져오는 경우

```mysql
SELECT u.id, u.name, uc.coupon_id, uc.use_yn
FROM user u
         LEFT JOIN user_coupon uc ON uc.user_id = u.id AND uc.coupon_id = 3
```

아래와 같이, user 테이블의 전체 결과를 조회하고, 그 중에서 coupon_id = 3인 경우에 데이터를 채우고 없으면, 나머지를 null로 채운다.

```shell
30000 rows in set (0.04 sec)
```

### WHERE절에서 coupon_id = 3만 가져오는 경우

```mysql
SELECT u.id, u.name, uc.coupon_id, uc.use_yn
FROM user u
         LEFT JOIN user_coupon uc ON uc.user_id = u.id
WHERE uc.coupon_id = 3
```

user 테이블의 전체 결과를 조회하고, 그 중에서 coupon_id = 3인 경우만 **필터링** 처리하기 때문에 총 1,000건의 데이터만 조회된다.

- 즉, `INNER JOIN` 과 동일하게 처리된다.

```shell
1000 rows in set (0.00 sec)
```

## 정리

- `LEFT JOIN` 을 사용하고자 한다면, Driven(Inner) Table 컬럼의 조인 조건은 `반드시 ON절에 명시해서 사용해야 한다. (IS NULL 조건은 예외)`
- `LEFT JOIN` 과 `INNER JOIN` 은 결과 데이터 및 쿼리 처리 방식이 매우 다르므로, 필요에 맞게 올바르게 사용하는 것이 중요하다.
- `LEFT JOIN` 쿼리에서 `COUNT` 를 사용하는 경우, `LEFT JOIN` 이 필요하지 않다면 `LEFT JOIN` 을 제거해서 사용하는 것이 성능에 조금 더 도움이
  된다.
