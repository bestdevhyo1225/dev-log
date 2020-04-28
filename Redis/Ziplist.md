## Ziplist of Hashes

<br>

### :book: 메모리 절약 데이터 구조

기본적으로 Hashes는 내부적으로 2가지 데이터 타입을 사용한다.

- 필드 개수가 적거나 길이가 작을때 `Zip List`를 사용한다.

- 필드 개수가 많거나 길이가 클 때 `Hash Table`을 사용한다.

Hashes의 주 데이터 타입은 `Hash Table`인데 메모리를 많이 사용한다.

redis.conf의 ADVANCED CONFIG 섹션을 보면, Hashes의 내부 타입을 결정할 2개의 파라미터가 있다.

- `hash-max-ziplist-entries 512`

  > 필드의 개수가 512개 이하이면 `Zip List`에 저장되고, 513개 이상이면 `HashTable`에 저장한다.

- `hash-max-ziplist-value 64`

  > String 바이트 수가 64바이트 이하이면 `Zip List`에 저장되고, 65바이트부터는 `HashTable`에 저장한다.

메모리가 부족하면, 이 파라미터를 적절히 조절해서 메모리를 적게 사용할 수 있도록 도와준다.

<br>

## :book: Functions

`HSET Command`는 3개의 function이 수행된다.

- `hashTypeLookupWriteOrCreate()`

  - key를 찾고, 만약 key가 없으면 필요한 object를 생성한다.

  - 새로운 Key이면 redis.conf의 hash-max-ziplist-entries, value 값과 관계없이 `Zip List`를 생성한다. 다음에 해당 파라미터 값을 검사해서 조건에 맞으면 `Hash Table`로 변환한다.

- `hashTypeTryConversion()`

  - 값의 길이를 검사해서 설정된 것보다 크면 `Zip List`를 `Hash Table`로 변환한다.

- `hashTypeSet()`

  - 필드 수를 검사해서 설정된 값보다 크면 `Zip List`를 `Hash Table`로 변환하고 아니면, `Zip List`에 필드와 값을 저장한다.

`HGET Command`은 2개의 function이 수행된다.

- `lookupKeyReadOrReply()` : key가 있는지 확인하고, key를 찾으면 value object를 찾는다.

- `addHashFieldToReply()` : 찾은 value object로 필드를 찾고, 값을 찾는다.

<br>

### :bookmark: 참고

- [ZIP LIST - 메모리 절약 데이터 구조](http://redisgate.kr/redis/configuration/ds_ziplist_hashes.php)
