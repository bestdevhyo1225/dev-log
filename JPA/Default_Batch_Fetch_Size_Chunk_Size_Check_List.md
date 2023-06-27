# default_batch_fetch_size, @Batch Size, ChunkSize 사용시, MySQL 옵션 확인하기

## MySQL Index Range Scan 작동 원리

### Index Dive

- `index range scan` 을 위한 최적화시, `row` 를 예측하기위해 직접 `index` 를 이용해서 `row` 를 예측하는데, 이 방식이 `index dive` 이다.

### Index Statistics

- `MySQL 5.6` 부터는 `index dive` 방식이 아니라, `인덱스 통계 정보` 를 바탕으로 `실행 계획` 을 세울 수 있도록 하는데, 이 방식이 `index statistics` 방식이다.

## MySQL의 옵션을 확인하자

### eq_range_index_dive_limit 옵션

```mysql
show variables like 'eq_range_index_dive_limit';
```

- `eq_range_index_dive_limit` 옵션으로 인해서 실행 계획 선택 방식이 `IN` 절 개수에 따라 달라진다.
- `MySQL 5.7.4` 이상의 버전들은 `200` 개의 디폴트 값으로 설정되어 있다.
    - 참고) `MySQL 5.7.3` 이하의 버전들은 `10` 개의 디폴트 값으로 설정되어 있다.
  - `200` 개 값을 초과하면, `index dive` 방식이 아닌, `인덱스 통계 정보` 를 바탕으로 `실행 계획` 을 세울 수 있도록 하는 `index statistics` 를 사용한다.
- 통계 정보가 정확할때도 있지만, 아직까지 100% 신뢰할 수는 없기 때문에 `200` 개를 초과할 경우, 생각지 못한 성능 저하가 발생할 수 있다.
- 물론, 정확하게 통계가 반영될 수도있기 때문에 `IN` 의 값으로 `200` 개로 하면 `수백 ~ 수천번의 쿼리 실행` 이 필요할 경우,
  위와 같은 경우에는 `IN` 의 개수를 서서히 늘려가면서 성능 테스트를 해봐야한다.

### range_optimizer_max_mem_size 옵션

```mysql
show variables like 'range_optimizer_max_mem_size';
```

- 추가로 `IN` 절 갯수가 똑같아도 `테이블 풀 스캔` 혹은 `다른 인덱스` 를 실행하게 된다면, 옵션 값을 조정하자
- `0` 으로 변경 (설정 `OFF`)
- `IN` 절에 맞게 사이즈 조절

## JPA, Spring Batch에서 고려할 점

- `JPA` 에서는 위에서 언급한 것처럼 `default_batch_fetch_size` 옵션을 통해 `IN` 절 쿼리를 사용하는데, `eq_range_index_dive_limit` 값과 동일하게 맞춰주도록 하자
- `Spring Batch` 에서도 `chunkSize` 를 활용할 때, `eq_range_index_dive_limit` 값과 동일하게 맞춰 주도록 하자
- `eq_range_index_dive_limit` 값과 동일하게 맞춰줘야 `index dive` 방식을 사용한다고 보장할 수 있기 때문에 되도록이면 맞춰서 사용하자

## 참고

- [MySQL IN절을 통한 성능 개선 방법](https://jojoldu.tistory.com/565)
- [MySQL5.6 IN(val1, ..., valN) 를 index range scan 작동원리](http://small-dbtalk.blogspot.com/2016/02/)
