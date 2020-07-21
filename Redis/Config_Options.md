# Redis Config 옵션 정리

- [Network]()

- [General]()

- [Snap Shotting]()

- [Replication]()

- [Security]()

- [Clients]()

- [Memory Management]()

- [Append Only mode]()

- [Advanced Config]()


## :gear: Network

### Bind

`bind [ ip address ] (default 127.0.0.1)`

- 지정한 IP Address로만 레디스 서버에 접속할 수 있다.

- Bind 지시자를 지정하지 않으면, 서버에서 사용 가능한 모든 네트워크 인터페이스로 접속을 허용한다.

### Port

`port [ number ] (default 6379)`

- Connection 요청을 받을 포트를 지정한다.

### Timeout

`timeout [ second ]`

- 클라이언트의 idle 시간이 해당 초만큼 지속되면, connection을 닫는다.

- 0으로 지정할 시, 항상 connection을 열어둔다.

<br>

## :gear: General

### Demonize

`daemonize [ yes | no ] (default no)`

- 레디스 서버는 기본적으로 데몬으로 실행되지 않는다. 

- 운영(production) 환경에서는 yes로 설정해야 한다. 

- 레디스 서버는 데몬으로 실행될 때 /var/run/redis.pid 파일에 pid를 쓴다.

### Log Level

`loglevel [ debug | verbose | notice | warning ]`

- 로그 레벨을 지정할 수 있다.

### Log File Path

`logfile [ file path ]`

- 로그 파일이 쓰여질 파일 경로를 지정한다.

<br>

## :gear: Snap Shotting

### Save

`save [ seconds ] [ changes ]`

- 지정된 시간(초)동안 지정된 개수 이상의 키가 변경되면, DB에 있는 전체 데이터를 디스크에 저장한다.

- save 기능을 사용하지 않으려면 아래 save를 모두 주석처리하거나, `save ""` 하면된다.

<br>

## :gear: Replication

### Replicaof

`replicaof [ master ip ] [ master port ]`

- version 4에서는 `slaveof`를 사용했고, version 5에서는 `replicaof`를 사용한다.

- Master-Slave 관계를 구성할 수 있다.

- Master 데이터가 Slave에 복제된다.

### Master Auth

`masterauth [ master-password ]`

- 만약 Master 설정 파일에 Slave가 붙기 위한 암호가 있다면, Master가 지정한 패스워드를 설정해야 한다.

### Replica Serve Stale Data

`replica-serve-stale-data [ yes | no ] (default yes)`

- 복제 서버가 Master와 연결이 끊겼을 때, 복제 서버는 설정에 따라 동작한다. (repl-timeout 시간이 지났거나, Master가 실제로 다운되었을 경우)

    - yes : 클라이언트 요청에 응답한다.
    
    - no :  클라이언트가 요청하면 "(error) MASTERDOWN Link with MASTER is down and replica-serve-stale-data is set to 'no'." 에러를 리턴한다.
          
- 단 다음 명령은 정상 응답한다. (INFO, replicaOF, AUTH, PING, SHUTDOWN, REPLCONF, ROLE, CONFIG, SUBSCRIBE, UNSUBSCRIBE, PSUBSCRIBE, PUNSUBSCRIBE, PUBLISH, PUBSUB, COMMAND, LATENCY)

### Replica Read Only

`replica-read-only [ yes | no ] (default no)`

- 복제 서버의 조회 전용 여부를 설정한다.

- 레디스 서버 2.6부터 복제 서버는 디폴트로 조회 전용이다.

### Repl Ping Replica Period

`repl-ping-replica-period [ second ] (default 10)`

- 복제 서버는 미리 설정된 간격마다 마스터 서버에 Ping을 보낸다.

- repl-timeout을 정하는데 사용된다.

- 디폴트는 10초이고, 변경할 수 있다.

### Repl Timeout

`repl timeout [ second ] (default 60)`

- 조건에 따라 복제 timeout이 설정된다.

- 복제 서버 관점
    
    - 마스터로 부터 데이터가 timeout 시간 동안 오지 않거나 ping에 응답이 없을 때
    
    - 동기화(sync)중 마지막 전송 받은 시간이 timeout 시간을 초과할 때

- 마스터 서버 관점

    - 복제 서버로 replconf에 대한 응답(ack)이 timeout 시간 동안 없을 때
    
- repl-ping-replica-period 보다 길게 설정해야 한다. 그렇지 않으면, 매번 타임아웃 된다.

<br>

## :gear: Security

### Require Pass

`requirepass [ password ]`

- Slave가 Master에 접근할 때, 보안 설정을 위한 옵션이다.

- 하위 버전 호환성을 위해서 주석(Comment)처리한다. 대부분의 사용자들에 auth를 필요로 하지 않기 때문

<br>

## :gear: Clients

### Max Client

`maxclient [ size ] (default 10000)`

- 레디스 서버에 연결할 수 있는 최대 클라이언트 개수를 설정한다.

- 레디스 서버가 내부적으로 32개를 사용한다.

<br>

## :gear: Memory Management

### Max Memory

`maxmemory [ bytes ]`

- 레디스 서버가 사용할 수 있는 메모리 한계치를 설정할 수 있다.

- 메모리 한계에 도달하면, 입력을 받지 않을지 아니면 기존에 키 중에서 어떤 키를 삭제할 지 maxmemory-policy로 정할 수 있다.

- 이 옵션은 레디스를 LRU 또는 LFU 캐시 서버로 사용할 때나 인스턴스의 메모리 사용에 제한을 설정할 때 유용하다.

### Max Memory Policy

`maxmemory-policy [ policy ]`

- 설정한 메모리 한계치까지 사용했을 때, 어떻게 할지를 결정한다.

    - **volatile-lru** : 만료 시간이 설정된 키 중에서 근사 LRU로 삭제할 키를 정한다.
    
    - **allkeys-lru** : 모든 키 중에서 근사 LRU로 삭제할 키를 정한다.
    
    - **volatile-lfu** : 만료 시간이 설정된 키 중에서 근사 LFU로 삭제할 키를 정한다.
    
    - **allkeys-lfu** : 모든 키 중에서 근사 LFU로 삭제할 키를 정한다.
    
    - **volatile-random** : 만료 시간이 설정된 키 중에서 임의로 삭제할 키를 정한다.
    
    - **allkeys-random** : 모든 키 중에서 임의로 삭제할 키를 정한다.
    
    - **volatile-ttl** : 만료 시간이 가장 가까운 키 순으로 삭제한다.
    
    - **noeviction** : 키를 삭제하지 않는다. 쓰기 명령이 들어오면, 에러를 리턴한다.

- LRU, LFU, TTL은 근사 임의 알고리즘으로 구현되어 있다.

### Max Memory Samples

`maxmemory-samples [ number ] (default 5)`

- LRU, LFU, 최소 TTL 알고리즘은 정밀하지 않다. 하지만, 메모리를 절약하기 위한 적절한 알고리즘이다.

- 레디스는 5개의 키를 검사해서 그 중 하나를 선택한다.

- Sample 사이즈를 변경할 수 있다.

- 기본 값인 5가 적당하고, 10은 보다 정밀하며, 3은 빠르지만 정밀하지 않다. 

<br>

## :gear: Append Only mode

### Append Only

`appendonly no`

- 데이터 지속성을 유지하기 위한 옵션이다.

- 캐시 서버로 사용하는 경우에는 무조건 no로 해둘 것

<br>

## :gear: Advanced Config

### List

`list-max-ziplist-size [ option ] (default -2)`

- 리스트 내부의 노드 크기를 지정할 수 있다.

    - **-5** : max size 64 KB
    
    - **-4** : max size 32 KB
    
    - **-3** : max size 16 KB
    
    - **-2** : max size 8 KB (default)
    
    - **-1** : max size 4 KB
    
- 양수 값은 노드당 엔트리 수를 지정할 수 있다.

### Set

`set-max-intset-entries 512`

- 데이터가 정수이거나 필드의 개수가 512개 이하면, IntSet에 저장된다.

- 데이터가 정수가 아니거나 513개 부터는 HashTable에 저장된다.

### Sorted Set (ZSet)

`zset-max-ziplist-entries 128`

- 필드의 개수가 128개 이하면, ZipList에 저장된다.

- 129개 부터는 SkipList에 저장된다.

`zset-max-ziplist-value 64`

- Byte 수가 64Byte 이하면, ZipList에 저장된다.

- 65Byte 부터는 SkipList에 저장한다.
    
### Hashes

`hash-max-ziplist-entries 512`

- 필드의 개수가 512개 이하면, ZipList에 저장된다.

- 513개 부터는 HashTable에 저장한다.

`hash-max-ziplist-value 64`

- Byte 수가 64Byte 이하면, ZipList에 저장된다.

- 65Byte 부터는 HashTable에 저장한다.

