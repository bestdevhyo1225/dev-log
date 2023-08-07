# MySQL Online DDL Tool

## 들어가기에 앞서

- `MySQL 5.6` 버전 부터 `Online DDL` 기능을 제공하지만, 상황에 따라서 `Tool` 을 사용하기도 한다.

## 흔히 언급되는 Online DDL Tool

- `pt-osc(pt-online-schema-change)`
- `gh-ost`

## pt-osc(pt-online-schema-change), gh-ost 공통점

- `Table Copy` 방식을 사용한다.
- 기존 테이블의 `Row` 를 `Chunk` 단위로 임시 테이블에 복사한다.
- 복사 완료 후, `RENAME TABLE` 로 임시 테이블을 기존 테이블로 변경하여, 사용할 수 있다.
- `Tool` 을 원격 서버에서 사용 할 수 있다.
- `PK` 혹은 `Unique key` 반드시 필요하다.
    - 중복없이 `Chunk` 를 나누는 단위 때문에 그렇다.

## pt-osc(pt-online-schema-change), gh-ost 차이점

|                   | pt-osc(pt-online-schema-change) |                                                                    gh-ost                                                                    |
|:-----------------:|:-------------------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------:|
|  작업 중 DML 반영 방식   |             Trigger             |                                                              Binary Log(binlog)                                                              |
|   binary_format   |              상관 없음              |                                                           Row (Mixed 포맷은 지원되지 않음)                                                            |
| log_slave_updates |              상관 없음              |                                           Master에서 Binary Log(binlog)를 가져오면 상관 없고, Slave에서 가져오면 필수                                           |
| FK (Foreign Key)  |               지원                |                                                                     미지원                                                                      |
|      Trigger      |               지원                |                                                                     미지원                                                                      |
|     임의 중단 기능      |               미지원               |                                                            Throttle, Panic 옵션 지원                                                             |
|    작업 도중 설정 변경    |               미지원               | 지원 (참고 - [Interactive commands](https://github.com/github/gh-ost/blob/d2726c77f86cb65e25bce5c0ac5fe3fa0c997488/doc/interactive-commands.md)) |
|    작업 수행 DB 서버    |             Master              |                                                     Master / Slave(–migrate-on-replica)                                                      |

## 참고

- [MySQL online DDL을 위한 TOOL 비교 ( pt-osc & gh-ost )](https://kimdubi.github.io/mysql/online_schema_change/)
- [[Tech] 플랫폼 서버 엔지니어의 pt-osc 도입기](https://www.rapportlabs.kr/product_2023_ptosc)
- [소소한 데이터 이야기 – pt-online-schema-change 편](https://gywn.net/2017/08/small-talk-pt-osc/)
- [github/gh-ost](https://github.com/github/gh-ost)
- [MySQL 쿼리 최적화 - DDL (스키마 조작)](https://github.com/bestdevhyo1225/dev-log/blob/master/MySQL/MySQL-Query-Optimization-DDL.md)
