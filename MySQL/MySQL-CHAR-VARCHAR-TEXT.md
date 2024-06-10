# MySQL CHAR vs VARCHAR vs TEXT

## CHAR vs VARCHAR

### CHAR 타입을 선택해야 하는 경우

- 저장되는 문자열의 가변 길이 폭이 좁고
- 자주 변경되는 컬럼의 경우, CHAR 타입이 적합하다.
  - 특히 인덱스 된 컬럼인 경우

### CHAR 타입을 선택해야하는 경우인데, VARCHAR 타입을 선택한 경우

- 데이터 페이지 내부 조각화 현상이 심해진다.
- CHAR 타입보다 공간 효율이 떨어진다.
- 내부적으로 빈번한 Page Reorganize 작업이 필요하다.

## VARCHAR vs TEXT

### VARCHAR 타입을 선택해야 하는 경우

- 상대적으로 저장되는 데이터 사이즈가 많이 크지 않은 경우
  - 상대적이라는 표현 때문에 사이즈가 많이 크다는 것에 대한 기준이 없긴하다.
- 컬럼 사용이 빈번한 경우
- DB 서버의 메모리 용량이 충분한 경우

## 참고

- [인프런 - Real MySQL 시즌 1 - Part 1](https://www.inflearn.com/course/real-mysql-part-1/dashboard)
