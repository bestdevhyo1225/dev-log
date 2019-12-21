## MySQL에서 JOIN

<br>

### :book: JOIN (INNER JOIN)

MySQL에서 `JOIN`은 네스티드-루프 방식만 지원한다. 네스티드-루프란 일반적으로 프로그램을 작성할 때 두 개의 FOR나 WHILE과 같은 반복 루프 문장을 실행하는 형태로 `JOIN`이 처리되는 것을 의미한다.

```
FOR ( record1 IN TABLE1 ) { // 외부 루프 (OUTER)
    FOR ( record2 IN TABLE2 ) { // 내부 루프 (INNER)
        IF ( record1.join_column == record2.join_column ) {
            join_record.found(record1.*, record2.*);
        } ELSE {
            join_record_notfound();
        }
    }
}
```