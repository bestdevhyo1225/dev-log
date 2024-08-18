# MySQL 에러 핸들링

## Error No

- 4자리 정수로 구분되나, 최근에 6자리로 늘어났다.
- 1 ~ 999 -> MySQL Global Error
- 1000 ~ 1999 -> MySQL Server Error
- 2000 ~ 2999 -> MySQL Client(or Connector) Error
- 3000 ~ -> MySQL Server Error
- MY-010000 ~ -> MySQL Server Error

3000 이후 대역과 MY-010000 이후 대역은 MySQL 8.0 이후 추가되었다.

## SQL State

- 5글자 영문/숫자로 구성되어 있다.
- ANSI-SQL에서 제정한 Vendor에 비 의존적인 에러 코드이다.
- SQL-STATE는 두 파트로 구분된다.
  - 앞 두글자 : 상태 값 분류
    - `00` : 정상
    - `01` : 경고
    - `02` : 레코드 없음
    - `HY` : ANSI-SQL에서 아직 표준 분류를 하지 않은 상태 (벤더 의존적 상태 값)
    - 나머지는 모두 에러
  - 뒷 세글자 : 주로 숫자 값이며, 각 분류별 상세 에러 코드 값 (영문 가끔 포함됨)  

## Error Message

- 사람이 인식할 수 있는 문자열
