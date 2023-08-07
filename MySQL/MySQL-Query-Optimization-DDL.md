# MySQL 쿼리 최적화 - DDL (스키마 조작)

## Online DDL

- MySQL 8.0 버전 이후부터는 대부분의 스키마 변경 작업은 MySQL 서버에 내장된 `Online DDL` 기능으로 처리가 가능해졌다.
    - `pt-online-schema-change` 도구를 사용하지 않아도 된다.

### Online DDL 알고리즘

- `ALTER TABLE` 명령을 실행하면, MySQL 서버는 다음과 같은 순서로 스키마 변경에 적합한 알고리즘을 찾는다.
    1. `ALGORITHM=INSTANT` 로 스키마 변경이 가능한지 확인 후, 가능하다면 선택한다.
        - `INSTANT` 는 `MySQL 8.0.12 버전` 부터 사용할 수 있는 알고리즘이다.
        - 즉, `ALGORITHM` 의 기본 값이 `INSTANT` 이다.
    2. `ALGORITHM=INPLACE` 로 스키마 변경이 가능한지 확인 후, 가능하다면 선택한다.
    3. `ALGORITHM=COPY` 알고리즘을 선택한다.
- 스키마 변경 알고리즘의 우선순위가 낮을수록 MySQL 서버는 스키마 변경을 위해서 `더 큰 잠금` 과 `많은 작업` 을 필요로 하고, `서버의 부하도 많이 발생시킨다.`
    - `MySQL 8.0.12 버전` 기준, `INSTANT`, `INPLACE`, `COPY` 순으로 우선순위를 따른다.
- `ALGORITHM` 과 `LOCK` 옵션이 명시되지 않으면, MySQL 서버가 적절한 수준의 알고리즘과 잠금 수준을 선택하게 된다.

#### INSTANT

- 테이블의 데이터는 전혀 변경하지 않고, 메타데이터만 변경하고 작업을 완료한다.
- 레코드 건수와 무관하게 작업 시간은 매우 짧다.
- 스키마 변경 도중 테이블의 읽고 쓰기는 대기하게 되지만, 스키마 변경 시간이 매우 짧기 때문에 다른 커넥션의 쿼리 처리에는 크게 영향을 미치지 않는다.

#### INPLACE

- 임시 테이블로 데이터를 복사하지 않고, 스키마 변경을 실행한다.
- 레코드의 복사 작업은 없지만, 테이블의 모든 레코드를 리빌드 하는 경우도 있기 때문에 테이블의 크기에 따라 많은 시간이 소요될 수도 있다.
- 스키마 변경 중에도 테이블의 읽기와 쓰기 모두 가능하다.
- 최초 시작 시점과 마지막 종료 시점에는 테이블의 읽고 쓰기가 불가능하다.
    - 이 시간은 매우 짧기 때문에 다른 커넥션의 쿼리 처리에 대한 영향도는 높지 않다.
- 데이터 재구성(테이블 리빌드)이 필요한 경우, 잠금을 필요로 하지 않기 때문에 읽고 쓰기는 가능하지만, 여전히 테이블의 레코드 건수에 따라 상당히 많은 시간이 소요될 수도 있다.
- 데이터 재구성(테이블 리빌드)이 필요하지 않은 경우, `INPLACE` 알고리즘을 사용하지만, `INSTANT` 알고리즘과 비슷하게 작업이 매우 빠르게 완료될 수 있다.

> INPLACE 알고리즘 처리 과정

1. `INPLACE` 스키마 변경이 지원되는 스토리지 엔진의 테이블인지 확인한다.
2. `INPLACE` 스키마 변경을 준비한다.
    - 스키마 변경에 대한 정보를 준비해서 `Online DDL` 작업 동안 변경되는 데이터를 추적할 준비를 한다.
3. 테이블 스키마 변경 및 새로운 `DML` 을 로깅한다.
    - 실제 스키마 변경을 수행하는 과정이다.
    - 해당 작업이 수행되는 동안은 다른 커넥션의 `DML` 작업이 대기하지 않는다.
    - 스키마를 온라인으로 변경함과 동시에 다른 스레드에서는 사용자에 의해서 발생한 `DML` 들에 대해서 별도의 로그로 기록한다.
4. 로그를 적용한다.
    - `Online DDL` 작업 동안 수집된 `DML` 로그를 테이블에 적용한다.
5. `INPLACE` 스키마 변경을 완료한다.
    - `COMMIT`

> INPLACE 알고리즘 처리 과정 해석

- `2번` 과 `4번` 단계에서는 잠깐의 `배타적 작금(Exclusive Lock)` 이 필요하다.
    - 이 시점에는 다른 커넥션의 `DML` 들이 잠깐 대기한다.
- 실제 변경 작업이 실행되면서, 많은 시간이 필요한 `3번` 단계는 다른 커넥션의 `DML` 작업이 `대기 없이 즉시 처리된다.`
    - 작업이 진행되는 동안 새로 유입된 `DML` 쿼리들에 의해 변경되는 데이터를 `온라인 변경 로그(Online alter log)` 라는 `메모리` 공간에 쌓아 두었다가 온라인 스키마 변경이
      완료되면, `로그의 내용을 실제 테이블로 일괄 적용하게 된다.`
    - `온라인 변경 로그(Online alter log)` 는 디스크가 아닌 `메모리` 에만 생성되며, 크기는 `innodb_online_alter_log_max_size` 시스템 설정 변수에 의해 결정된다.
    - `온라인 변경 로그(Online alter log)` 의 기본 크기는 `128MB` 이다.

#### COPY

- 변경된 스키마를 적용한 임시 테이블을 생성하고, 테이블의 레코드를 모두 임시 테이블로 복사한 후, 최종적으로 임시 테이블을 RENAME 해서 스키마 변경을 완료한다.
- 테이블 읽기만 가능하고, DML(INSERT, UPDATE, DELETE)은 실행할 수 없다.

### Online DDL 잠금

- `ALGORITHM` 과 `LOCK` 옵션이 명시되지 않으면, MySQL 서버가 적절한 수준의 알고리즘과 잠금 수준을 선택하게 된다.
- `INSTANT` 알고리즘은 테이블의 메타데이터만 변경하기 때문에 매우 짧은 시간동안의 메타데이터 잠금만 필요로 한다. 그래서 `LOCK` 옵션을 명시할 수 없다.
- `INPLACE` 나 `COPY` 알고리즘을 사용하는 경우, `LOCK` 은 다음 3가지 중 하나를 명시할 수 있다.

#### NONE

- 아무런 잠금을 걸지 않는다.
- 알고리즘으로 `INPLACE` 가 사용되는 경우, 대부분 잠금은 `NONE` 으로 설정이 가능하다.

#### SHARED

- 읽기 잠금을 걸고, 스키마 변경을 실행하기 때문에 스키마 변경 중 읽기는 가능하지만, 쓰기(INSERT, UPDATE, DELETE)는 불가하다.
- `INPLACE` 알고리즘을 사용하더라도 가끔 `SHARED` 수준까지 설정해야 할 수도 있다.

#### EXCLUSIVE

- 쓰기 잠금을 걸고, 스키마 변경을 실행하기 때문에 테이블의 읽고 쓰기가 불가하다.

### Online DDL 실패 케이스

- `INSTANT` 알고리즘을 사용하는 경우에는 거의 시작과 동시에 작업이 완료되기 때문에 `작업 도중 실패할 가능성은 거의 없다.`
- `INPLACE` 알고리즘으로 실행되는 경우에는 내부적으로 `테이블 리빌드` 과정이 필요하고, `최종 로그 적용` 과정이 필요해서 중간 과정에서 `실패할 가능성이 상대적으로 높은 편이다.`

#### Case 1

```markdown
ERROR 1799 (HY000): Creating index 'idx_col1' required more than 'innodb_online_alter_log_max_size' bytes of
modification log. Please try again.
```

- `ALTER TABLE` 명령이 장시간 실행되는 상황에서 동시에 다른 커넥션에서 `DML` 이 많이 실행되는 경우
- `온라인 변경 로그(Online alter log)` 공간이 부족한 경우

#### Case 2

```markdown
ERROR 1062 (23000): Duplicate entry 'd005-10001' for key 'PRIMARY'
```

- `ALTER TABLE` 명령이 실행되는 동안 `ALTER TABLE` 이전 버전의 테이블 구조에서는 아무런 문제가 안되지만, `ALTER TABLE` 이후의 테이블 구조에는 적합하지 않은
  레코드가 `INSERT`, `UPDATE` 되는 경우
    - `온라인 변경 로그(Online alter log)` 에 수집된 `DML` 을 테이블에 적용하는 시점에 발생한다.

#### Case 3

```markdown
ERROR 1864 (0A000): LOCK=NONE is not supported. Reason: Adding an auto-increment column requires a lock. Try
LOCK=SHARED.
```

- 스키마 변경을 위해서 필요한 잠금 수준보다 낮은 잠금 옵션이 사용된 경우

#### Case 4

```markdown
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

- 온라인 스키마 변경은 `LOCK=NONE` 으로 실행된다고 하더라도 변경 작업의 처음과 마지막 과정에서 잠금이 필요한데, 잠금을 획득하지 못하고 타임 아웃이 발생하는 경우

### Online DDL 진행 상황 모니터링

- MySQL 서버의 `performance_schema` 를 통해 진행 상황을 모니터링 할 수 있다.
- 가장 먼저 `performance_schema` 시스템 변수가 `ON` 으로 활성화 되어야 하고, `performance_schema` 옵션인 `Instrument` 와 `Consumer` 옵션이 활성화돼야
  한다.

```mysql
SET GLOBAL performance_schema=ON;
```

- `performance_schema` 시스템 변수를 활성화 하는데, MySQL 서버를 재시작 해야한다.

```mysql
UPDATE performance_schema.setup_instruments
SET ENABLED = 'YES', TIMED = 'YES'
WHERE NAME LIKE 'stage/innodb/alter%';
```

- `stage/innodb/alter%` instrument 활성화한다.

```mysql
UPDATE performance_schema.setup_consumers
SET ENABLED = 'YES'
WHERE NAME LIKE '%stages%';
```

- `%stages%` consumer 활성화한다.

```mysql
SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED
FROM performance_schema.events_stages_current;
```

- `performance_schema` 의 진행 상황을 알 수 있다.

### Online DDL Tool 비교

- [devlog - MySQL Online DDL Tool 공통점, 차이점](https://github.com/bestdevhyo1225/dev-log/blob/master/MySQL/MySQL-Online-DDL-Tool.md)

## 컬럼 변경

### 컬럼 추가

- `MySQL 8.0` 에서 테이블 컬럼 추가는 대부분 `INPLACE` 알고리즘을 사용하는 `Online DDL` 로 처리가 가능하다.
    - `MySQL. 5.7` 에서도 `INPLACE` 로 `Online DDL` 이 가능하다. (정확히는 `마이너 버전` 까지 확인해서 처리 가능 여부를 확인해야한다.)
- `MySQL 8.0` 에서는 테이블의 제일 `마지막` 에 컬럼을 추가할 경우에는 `INSTANT` 알고리즘으로 즉시 추가된다.
- 테이블에서 기존 컬럼들 `중간` 에 컬럼을 추가할 경우에는 `테이블 리빌드` 가 필요하기 때문에 `INPLACE` 알고리즘을 사용해야 한다.
- 그래서 `MySQL 8.0` 을 사용하면서 테이블이 큰 경우라면, 마지막 컬럼에 새로운 컬럼을 추가하는 것을 권장한다.

### 컬럼 삭제

- 항상 `테이블의 리빌드` 를 필요로 하기 때문에 `INSTANT` 알고리즘을 사용할 수 없다.
- 항상 `INPLACE` 알고리즘으로만 컬럼 삭제가 가능하다.

### 컬럼 이름 및 타입 변경

```mysql
ALTER TABLE salaries CHANGE to_date end_date DATE NOT NULL, ALGORITHM=INPLACE, LOCK=NONE;
```

- 이름만 변경하기 때문에 `테이블 리빌드` 가 필요하지 않으며, `INSTANT` 알고리즘과 같이 빠르게 작업이 완료된다.

```mysql
ALTER TABLE salaries MODIFY salary VARCHAR(20), ALGORITHM=COPY, LOCK=SHARED;
```

- 컬럼의 데이터 타입을 변경하는 경우에는 `COPY` 알고리즘을 사용해야 한다.
- 스키마 변경 도중에는 테이블의 쓰기 작업은 불가하다.

```mysql
ALTER TABLE employees MODIFY last_name VARCHAR(30) NOT NULL, ALGORITHM=INPLACE, LOCK=NONE;
```

- `VARCHAR` 타입의 길이를 확장하는 경우는 현재 길이와 확장하는 길이 관계에 따라 `테이블 리빌드가 필요할 수도 있고, 아닐 수도 있다.`
    - `INPLACE` 알고리즘으로 `VARCAHR(10)` 에서 `VARCAHR(20)` 으로 변경하는 경우라면, 둘 다 `255Byte` 이하이므로 `테이블 리빌드가 필요없다.`
    - `UTF8MB4` 문자 셋은 한 글자가 최대 `4Byte` 를 사용할 수 있기 때문에 `VARCHAR(64)` 는 최대 `256Byte` 를 사용한다. 그래서 이 경우에는 컬럼 값의 길이를 `1Byte`
      에서 `2Byte` 로 변경해야 하므로 `테이블의 레코드 전체를 다시 리빌드 해야한다.`

```mysql
ALTER TABLE employees MODIFY last_name VARCHAR(10) NOT NULL, ALGORITHM=COPY, LOCK=SHARED;
```

- `VARCHAR` 타입의 길이를 축소하는 경우는 완전히 다른 타입으로 변경되는 경우와 같이 `COPY` 알고리즘을 사용해야 한다.
- 스키마를 변경하는 도중 해당 테이블의 데이터 변경은 허용되지 않으므로 `LOCK` 은 `SHARED` 로 사용돼야 한다.

## 인덱스 변경

- `MySQL 8.0` 버전에서는 대부분의 인덱스 변경 작업이 `Online DDL` 로 처리 가능하도록 개선했다.

### 인덱스 추가

```mysql
ALTER TABLE employees ADD PRIMARY KEY (emp_no), ALGORITHM=INPLACE, LOCK=NONE;
ALTER TABLE employees ADD UNIQUE INDEX ux_empno (emp_no), ALGORITHM=INPLACE, LOCK=NONE;
ALTER TABLE employees ADD INDEX idx_lastname (last_name), ALGORITHM=INPLACE, LOCK=NONE;
ALTER TABLE employees ADD FULLTEXT INDEX fx_firstname_lastname (first_name, last_name), ALGORITHM=INPLACE, LOCK=SHARED;
ALTER TABLE employees ADD SPATIAL INDEX fx_loc (last_location), ALGORITHM=INPLACE, LOCK=SHARED;
```

- `전문 검색을 위한 인덱스(FULLTEXT INDEX)`, `공간 검색을 위한 인덱스(SPATIAL INDEX)` 는 `INPLACE` 알고리즘으로 인덱스 생성이 가능하지만, `SHARED` 잠금이 필요하다.
- 프라이머리 키라고 하더라도 `INPLACE` 알고리즘에 `잠금 없이` 온라인으로 인덱스 생성이 가능하다.

### 인덱스 이름 변경

- `INPLACE` 알고리즘을 사용하더라도 `테이블 리빌드가 필요하지 않기 때문에` 빠른 시간내로 인덱스를 교체할 수 있다.

### 인덱스 가시성 변경 (MySQL 8.0 이후)

```mysql
ALTER TABLE employees ALTER INDEX ix_firstname INVISIBLE;
```

- `MySQL 8.0` 버전부터는 인덱스의 가시성을 제어할 수 있는 기능이 도입됐다.
- 인덱스의 가시성이란, MySQL 서버가 쿼리를 실행할 때 해당 인덱스를 사용할 수 있게 할지 말지를 결정하는 것이다.
- 특정 인덱스가 사용되지 못하게 하는 `DDL` 문장이다.
- MySQL 옵티마이저는 `INVISIBLE` 상태의 인덱스는 없는 것으로 간주하고, 실행 계획을 수립한다.

```mysql
ALTER TABLE employees ALTER INDEX ix_firstname VISIBLE;
```

- `INVISIBLE` 상태의 인덱스를 다시 사용할 수 있게 하려면, `VISIBLE` 옵션을 명시하면 된다.
- 최초 인덱스를 생성할 때도 가시성을 설정할 수 있다.

### 인덱스 삭제

```mysql
ALTER TABLE employees DROP PRIMARY KEY, ALGORITHM=COPY, LOCK=SHARED;
ALTER TABLE employees DROP INDEX ux_empno, ALGORITHM=INPLACE, LOCK=NONE;
ALTER TABLE employees DROP INDEX fx_loc, ALGORITHM=INPLACE, LOCK=NONE;
```

- 세컨더리 인덱스 삭제 작업은 `INPLACE` 알고리즘을 사용하지만, 실제 `테이블 리빌드를 필요로 하지 않는다.`
- 프라이머리 키의 삭제 작업은 세컨더리 인덱스의 리프 노드에 저장된 프라이머리 키 값을 삭제해야 하기 때문에 임시 테이블로 레코드를 복사해서 테이블을 재구축해야한다.
- 프라이머리 키 삭제 도중 레코드 쓰기는 불가능한 `SHARED` 모드의 잠금이 필요하다.

## 테이블 변경 묶음 실행

- `Online DDL` 로 빠르게 스키마 변경을 처리할 수 있다면, 개별로 실행하는 것이 좋지만 그렇지 않다면 모아서 실행하는 것이 효율적이다.

```mysql
ALTER TABLE employees
ADD INDEX idx_lastname (last_name, first_name),
ADD INDEX idx_birthdate (birth_date),
ALGORITHM=INPLACE, LOCK=NONE;
```

- 2개의 인덱스를 각각 `ALTER TABLE` 명령으로 생성하는 데, 걸리는 시간보다는 훨씬 시간을 단축할 수 있다.
- 2개의 스키마 변경 작업이 하나는 `INSTANT`, 다른 하나는 `INPLACE` 를 사용한다면, 모아서 실행할 필요는 없다.
- 가능하면 같은 알고리즘을 사용하는 스키마 변경 작업이라면, 모아서 실행하는 것이 효율적이다.
- 같은 `INPLACE` 알고리즘을 사용한다고 하더라도 `테이블 리빌드` 가 필요한 작업과 그렇지 않은 작업끼리도 구분하고 모아서 실행할 수 있다면, 더 효율적으로 스키마 관리를 할 수 있다.

## 참고

- Real MySQL 8.0 - 개발자와 DBA를 위한 MySQL 실전 가이드
