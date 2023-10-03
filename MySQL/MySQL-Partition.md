# MySQL 파티션 - Range, List, Hash, Key

## Range

```mysql
ALTER TABLE `...` PARTITION BY RANGE (...) 
(
    PARTITION p0 VALUES LESS THAN(...) ENGINE=InnoDB,
    PARTITION p1 VALUES LESS THAN(...) ENGINE=InnoDB,
    PARTITION p2 VALUES LESS THAN(...) ENGINE=InnoDB,
    PARTITION p3 VALUES LESS THAN(...) ENGINE=InnoDB,
    PARTITION pMax VALUES LESS MAXVALUE THAN(...) ENGINE=InnoDB
);
```

### 특징

- 파티션 키의 연속된 범위로 파티션을 정의하는 방식이며, 일반적으로 가장 많이 사용되는 파티션 종류이다.

### 사용하기 적합한 경우

- 날짜 기반으로 데이터가 누적되고, 년도 또는 월, 일 단위로 적재하고, 삭제할 경우
- 범위 기반을로 데이터를 여러 파티션에 나누고자 할 때
- 파티션 키 위주로 검색이 자주 일어날 때
- 대량의 과거 데이터를 삭제할 때 (주기적 또는 반복적)
- 년도 단위에서는 `YEAR()`, 월/일 단위에서는 `TO_DAYS()` 를 사용한다.

## List

```mysql
ALTER TABLE `...` PARTITION BY LIST (...) 
(
    PARTITION p_red VALUES IN(...) ENGINE=InnoDB,
    PARTITION p_yellow VALUES IN(...) ENGINE=InnoDB,
    PARTITION p_green VALUES IN(...) ENGINE=InnoDB,
    PARTITION p_blue VALUES IN(...) ENGINE=InnoDB
);
```

### 특징

- `Range` 와 유사하지만, 파티션 키 값들을 리스트로 나열해야 한다.

### 사용하기 적합한 경우

- 파티션 키 값이 코드 값이거나, 카테고리와 같이 고정적인 경우
- 키 값이 연속되지 않고, 정렬 순서와 관계없이 파티션을 해야하는 경우
- 파티션 키 값 별로 레크도 건 수가 고르게 비슷하거나, 검색 조건에서 파티션 키 컬럼이 자주 사용될 경우

## Hash

```mysql
ALTER TABLE `...` PARTITION BY HASH (...) PARTITIONS 4
(
    PARTITION p1 ENGINE=InnoDB,
    PARTITION p2 ENGINE=InnoDB,
    PARTITION p3 ENGINE=InnoDB,
    PARTITION p4 ENGINE=InnoDB
);
```

### 특징

- 해시 함수에 의해 레코드가 저장될 파티션이 결정되는 방식이다.
- `PARTITION BY HASH (expr)` 를 사용하고, expr(표현식)은 항상 정수형 타입의 컬럼이거나, 정수를 반환하는 표현식만 사용할 수 있다.
- `PARTITIONS` 절을 명시하지 않는 경우 기본적으로 `PARTITIONS 1` 이 적용된다.

### 사용하기 적합한 경우

- 테이블의 모든 레코드가 비슷한 사용의 패턴이나 빈도로 사용하지만, 테이블의 사이즈가 매우 클 경우
- 테이블의 데이터가 특정 컬럼 값으로 영향을 받거나 치우치지 않고, 비슷한 사용 빈도를 보일 때

## Key

```mysql
ALTER TABLE `...` PARTITION BY KEY (...) PARTITIONS 4;
```

### 특징

- `Hash` 파티션과 비교해서 사용법과 특성이 비슷하지만, 분배 방식이 다르다.
- MD5() 함수를 사용해 해시 값을 계산하고, 그 값을 MOD 연산해서 데이터를 각 파티션에 분배한다.
- `Hash` 파티션에 비해 레코드를 균듕하게 분배한다.

### 사용하기 적합한 경우

- `Hash` 파티션과 동일하지 않을까?

## 참고

- [MySQL Partition 파티션(2) - 파티션 종류 와 파티션 테이블 생성 과 변경](https://hoing.io/archives/8527)
