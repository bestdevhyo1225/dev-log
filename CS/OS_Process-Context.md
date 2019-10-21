## 프로세스 문맥

<br>

### :book: 프로세스 문맥이란?

프로세스가 현재 어떤 상태에서 수행되고 있는지에 대해서 알 수 있는 정보를 의미한다.

<br>

### :book: 시분할 시스템을 통해 자세히 알아보자

시분할 시스템 환경에서는 타이머 인터럽트에 의해 짧은 시간 동안 CPU를 사용한 후, CPU를 선점당했다가 추후에 다시 CPU를 획득하는 식으로 CPU 관리가 이루어진다. 따라서 CPU를 다시 획득하는 시점에서 직전 수행 상태를 정확하게 알 필요가 있다. 이 정확한 재현을 위해 필요한 정보가 `프로세스 문맥`이다.

<br>

### :book: 프로세스 문맥은 크게 세 가지로 분류할 수 있다.

* 하드웨어 문맥

    * CPU의 수행 상태를 나타내는 것으로 프로그램 카운터(PC)와 각종 레지스터에 저장하고 있는 값들을 의미한다.

* 프로세스의 주소 공간

    * 코드, 데이터, 스택으로 구성되는 자기 자신만의 독자적인 주소 공간을 가지고 있다.

* 커널 상의 문맥

    * 프로그램이 수행되어 프로세스가 되면 운영체제는 프로세스를 관리하기 위한 자료 구조를 유지하는데, PCB(Process Control Block: 프로세스 제어 블록)와 커널 스택(Kernel Stack)이 이에 해당된다.