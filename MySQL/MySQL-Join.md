## MySQL에서 JOIN

<br>

### :book: JOIN (INNER JOIN)

MySQL에서 `JOIN`은 네스티드-루프 방식만 지원한다. 네스티드-루프란 일반적으로 프로그램을 작성할 때 두 개의 FOR나 WHILE과 같은 반복 루프 문장을 실행하는 형태로 `JOIN`이 처리되는 것을 의미한다.

```javascript
// javascript로 표현하자면.. 약간 이런 느낌?
for ( const record1 in table1 ) { // 외부 루프 (OUTER)
    for ( const record2 in table2 ) { // 내부 루프 (INNER)
        if ( record1.join_column === record2.join_column ) {
            join_record_found(record1.*, record2.*);
        } else {
            join_record_notfound();
        }
    }
}
```
* OUTER 테이블 : `JOIN`에서 주도적인 역할을 한다고해서 드라이빙(Driving) 테이블이라고 한다.

* INNER 테이블 : `JOIN`에서 끌려가는 역할을 한다고 해서 드리븐(Driven) 테이블이라고 한다.

```sql
SELECT *
FROM employees AS e -- Driving 테이블 (OUTER)
JOIN salaries AS s -- Driven 테이블 (INNER)
ON e.emp_no = s.emp_no;
```

<br>

### :book: OUTER JOIN

```javascript
for ( const record1 in table1 ) { // 외부 루프 (OUTER)
    for ( const record2 in table2 ) { // 내부 루프 (INNER)
        if ( record1.join_column === record2.join_column ) {
            join_record_found(record1.*, record2.*);
        } else {
            join_record_found(record1.*, null);
        }
    }
}
```

**LEFT OUTER JOIN (LEFT JOIN)**

아래의 sql문을 보면, `LEFT OUTER JOIN` 키워드 왼쪽에 테이블이 사용됐고 오른쪽 salaries 테이블이 사용됐기 때문에 employees가 `OUTER 테이블`이 된다. 따라서 **최종 결과는 employees 테이블의 레코드에 의해 결정된다.**

```sql
SELECT *
FROM employees AS e -- LEFT JOIN이라 employees가 OUTER 테이블 
LEFT OUTER JOIN salaries AS s -- LEFT JOIN이라 salaries가 INNER 테이블
ON e.emp_no = s.emp_no;
```

**RIGHT OUTER JOIN (RIGHT JOIN)**

아래으 sql문을 보면, `RIGHT OUTER JOIN` 키워드 오른쪽에 employees 테이블이 사용됐고 왼쪽에 salaries 테이블이 사용됐으므로 employees가 `OUTER 테이블`이 된다. 따라서 **최종 결과는 employees 테이블의 레코드에 의해 결정된다.**

```sql
SELECT *
FROM salaries AS s -- RIGHT JOIN이라 salaries가 INNER 테이블
RIGHT OUTER JOIN employees AS e -- RIGHT JOIN이라 employees가 OUTER 테이블
ON e.emp_no = s.emp_no;
```

<br>

### :bookmark: 참고

* RealMySQL - 6장 실행계획 JOIN