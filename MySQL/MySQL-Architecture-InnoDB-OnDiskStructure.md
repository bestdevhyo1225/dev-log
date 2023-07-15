# MySQL Architecture InnoDB On-Disk Structure

## 들어가기에 앞서

그 동안 `InnoDB` 를 사용하면서, `Index` 는 `In-Memory Structure` 영역에 있다고 생각했는데, 잘 못 알고 있었던 부분이다. `Table Data` 와 `Index`
는 `On-Disk Structure` 에 생성된다는 것을 알게 되었으며, 참고 문서를 통해 더 많은 내용을 알 수 있지만, 당장 학습하고 싶은 부분만 간단히 정리했다.

## InnoDB의 디스크 구조

<img width="777" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/4be3d4c9-b09c-4e0d-9434-6f965eb8affc">

- `InnoDB` 디스크 구조 관련 항목은 아래와 같다.
    - `Tablespace`
    - `Table`
    - `Index`
    - Doublewrite Buffer
    - Redo Log
    - Undo Log
- `InnoDB` 는 모든 데이터를 디스크 상의 `Tablespace` 라는 논리적인 공간에 저장한다.
- `InnoDB` 의 `File-Per-Table Tablespace` 에 기본적으로 테이블이 생성된다.

## Tablespace

- `Tablespace` 는 데이터를 저장하는데 사용되는 가장 큰 논리적 단위이며, 내부적으로 `Segment` -> `Extent` -> `Page` -> `Row` 의 형태로 구성된다.

<img width="837" alt="스크린샷 2023-07-14 오후 2 12 47" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/3910a54d-61c1-4254-98d2-10e1243ab818">

- `Tablespace` 는 저장하는 데이터의 종류와 방식에 따라 5가지로 분류된다.
    - `System Tablespace`
    - `File-Per-Table Tablespace`
    - `General Tablespace`
    - `Temporary Tablespace`
    - `Undo Tablespace`
- 5가지 `Tablespace` 중에서 그나마 실무에서 자주 접했던, 접해볼 만한 `File-Per-Table Tablespace` 에 대해 간단히 알아보고자 한다.

<img width="848" alt="스크린샷 2023-07-14 오후 2 14 57" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/e6bc5f38-14ca-467a-a2d3-b6287bdae752">

### File-Per-Table Tablespace

- 하나의 `InnoDB Table` 에 대한 `Data` 와 `Index` 를 **`하나의 파일(.ibd 파일)`** 로 관리하는 `Tablespace` 를 말한다.
- 별도의 `Tablespace` 생성 과정이 없으며, `innodb_file_per_table` 값이 `on` 이면, `File-Per-Table Tablespace` 에 `Table` 이 생성된다.
    - `innodb_file_per_table` 의 디폴트 값은 `on` 이다.
    - 즉, `File-Per-Table Tablespace` 에 디폴트로 `Table` 을 생성한다.
    - 참고) `innodb_file_per_table` 값을 비활성화하면, `System Tablespace` 에 생성된다.
- `Data` 파일은 `tablename.ibd` 형식으로 생성된다.

## Table

- `Table` 은 `System Tablespace`, `File-Per-Table Tablespace`, `General Tablespace` 안에 생성할 수 있다.
- `Table` 은 기본적으로 `File-Per-Table Tablespace` 에 생성된다.
- `Table` 은 `PK` 를 기준으로 정렬하여 저장한다.
- `Secondary Index` 들은 `Data` 의 실제 주소 대신 `PK` 를 논리적인 주소로 사용하며, `Clustered Index` 를 통해 Range 스캔을 빠르게 처리할 수 있다.
- `PK` 는 성능과 밀접한 연관이 있기 때문에 크거나 자주 사용되는 테이블에 대해서는 생성 시점에 `PK` 를 정의하는 것이 좋다.

## Index

- `Index` 는 `System Tablespace`, `File-Per-Table Tablespace`, `General Tablespace` 안에 생성할 수 있다.

### Clustered Index

<img width="777" alt="image" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/a58a2c36-6f5f-42cc-becd-84c857358e84">

- `Clustered Index` 는 `PK` 값을 이용하여, `Data` 를 정렬하여 저장한다.
- `Clustered Index` 는 일반적인 `B-Tree` 구조이며, `Leaf Node` 에는 모든 컬럼이 같이 저장되어 있다.
- `Clustered Index` 는 `PK` 기준으로 정렬된 `B-Tree` 인덱스와 동일한 값으로 정렬된 실제 데이터를 통칭한다. 그래서 `Clustered Index` 를 통해 접근하면, `Row` 가
  포함된 `Page` 로 이어져 빠른 속도로 처리가 가능하다.

### Non-Clustered Index (Secondary Index)

<img width="777" alt="스크린샷 2023-07-14 오후 3 48 02" src="https://github.com/bestdevhyo1225/dev-log/assets/23515771/d4f84f61-29d8-408d-849d-4df9b1807f9a">

- `Non-Clustered Index (Secondary Index)` 는 `Clustered Index` 를 제외한 나머지 `Index` 를 지칭한다.
- `Non-Clustered Index (Secondary Index)` 는 `Leaf Node` 에 `PK` 를 저장하고 있으며, `PK` 로 `Clustered Index` 를 추가로 검색해서
  원하는 `Data` 에 접근한다.
    - 즉, `Non-Clustered Index (Secondary Index)` -> `PK` -> `Clustered Index` -> `Data` 순으로 찾는다.
- `Non-Clustered Index (Secondary Index)` 가 `Leaf Node` 에 `PK` 를 가지고 있는 이유는 `Clustered Index` 의 `Data` 들이 생성, 수정, 삭제에
  의해서 변경이 되어도, 영향을 받지 않게끔 하기 위해서 이다.
    - `Data` 들이 생성, 수정, 삭제되면 `Data` 의 `Page` 번호가 `Page` 내 순서가 모두 변경된다.
    - `Non-Clustered Index (Secondary Index)` 의 `Leaf Node` 값이 `Data Page Number + #Offset` 를 가지고 있다면, 모두 수정해야하는 치명적인
      문제를 가지고 있다.
    - 따라서 `Non-Clustered Index (Secondary Index)`, `Clustered Index` 가 섞인 혼합 인덱스
      구조에서 `Non-Clustered Index (Secondary Index)` 의 `Leaf Node` 값은 `PK` 를 가지고 있는 것이다.

## 참고

- [DB 인사이드 | MySQL Architecture - 7. InnoDB : On-Disk Structure](https://blog.ex-em.com/1699)
- [데이터베이스 인덱스 (2) - 클러스터형 인덱스와 비클러스터형 인덱스](https://hudi.blog/db-clustered-and-non-clustered-index/)
