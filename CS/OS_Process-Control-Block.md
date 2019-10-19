## 프로세스 제어 블록(Process Control Block, PCB)

<br>

### :book: 프로세스 제어 블록이 뭔가요?

시스템 내에서 프로세스들을 관리하기 위해 프로세스당 유지하는 정보들을 담는 `커널 내의 자료구조`이다.

<br>

### :book: 프로세스 제어 블록(PCB)의 구성 요소는?

* 프로세스의 상태 (Process State)

* 프로그램 카운터 값 (Program Counter)

* CPU 레지스터 (CPU Register)

* CPU 스케줄링 정보 (CPU Scheduling Information)

* 메모리 관리 정보 (Memory Management Information)

* 자원 사용 정보 (Accounting Information)

* 입출력 상태 정보 (I/O Status Information)

<br>

### :book: 프로세스 제어 블록(PCB)의 구조

![image](https://user-images.githubusercontent.com/23515771/67143682-db594900-f2a8-11e9-9fbc-9c3080b5c31b.png)

`1. OS가 관리상 사용하는 정보`

> Process State, Process Number(PID), Scheduling Information, Priority

`2. CPU 수행 관련 하드웨어 값`

> Program Counter, Registers

`3. 메모리 관련`

> Code, Data, Stack의 위치 정보

`4. 파일 관련`

> Open file descriptors...