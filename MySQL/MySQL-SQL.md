### SQL 모음

<br>

* AUTO_INCREMENT 초기화 하기

```sql
ALTER TABLE 테이블명 AUTO_INCREMENT=1;
SET @COUNT = 0;
UPDATE 테이블명 SET AUTO_INCREMENT로 지정된 컬럼명 = @COUNT:=@COUNT+1;
```

* 현재 테이블에서 인덱스 보기

```sql
SHOW INDEX FROM 테이블명
```