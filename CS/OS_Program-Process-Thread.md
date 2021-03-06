## Promgram & Process & Thread

<br>

모바일 앱으로 PC를 제어하는 프로젝트를 개발했었다. 당시에 C++과 MFC를 사용하여 PC를 이벤트로 제어하는 기능과 클라이언트의 요청에 따라 파일을 송신하는 기능을 멀티 스레드로 구현했었다. 그리고 IT 분야에서 자주 접하는 단어인데 누군가에게 설명하라고 하면 제대로 해야겠다는 생각이 들어서 요약하려고 한다.

<br>

### :book: 프로그램이 뭘까?

어떠한 작업을 위해서 실행할 수 있는 파일을 `프로그램`이라고 한다.

<br>

### :book: 그렇다면 프로세스는 뭘까?

`프로세스`는 프로그램이 컴퓨터 메모리에 올라와서 연속적으로 실행되고 있는 프로그램의 인스턴스를 말한다. 즉, OS가 프로그램을 메모리에 할당해서 실행하면 그것이 바로 `프로세스`이다. 그리고 프로세스는 각각 독립된 메모리 영역(`Code`, `Data`, `Stack`, `Heap`)을 할당 받는다. 그리고 프로세스는 최소 1개 이상의 `스레드`를 가지고 있다.

**`Code 영역`**

> 프로세스가 실행할 코드와 매크로 상수가 기계어로 저장되어 있는 공간이다. 컴파일 타임에 이 작업이 결정된다.

**`Data 영역`**

> 코드에서 선언한 전역 변수나 static 변수가 저장된 공간이다.

**`Stack 영역`**

> Stack 영역은 함수 안에 있는 지역 변수, 매개 변수, 리턴 값등이 저장되어 있으며 함수를 호출할 때 저장한다. 함수가 종료가 되면 자료구조 Stack 메커니즘에 따른 후입 선출(LIFO) 방식을 따른다.

**`Heap 영역`**

> Heap 영역은 프로그래머가 필요할 때 사용할 수 있는 영역이다. 다른 영역들은 기본적으로 컴파일 타임에 결정되지만, Heap 영역은 런타임에 결정된다. 주로 가변적인 배열을 할당할 때 많이 사용한다. 주의할 점은 메모리에 할당 했으면, 반드시 해제해야 한다. 해제하지 않으면 Memory Leak이라는 치명적인 문제가 발생한다.

<br>

### :book: 스레드?

프로세스 내에서 실행되는 여러 작업의 단위라고 말할 수 있다. 예를 들면, A작업을 처리하는 스레드가 있고 B작업을 처리하는 스레드가 있다. 각자의 스레드는 Stack영역만 따로 할당받고 나머지 영역을 공유한다.

<br>

### :book: 프로세스와 스레드는 뭔가를 실행해서 작업하는 것 같은데 공통점과 차이점은 뭐지?

**`공통점`**

> 프로세스와 스레드는 공통점은 여러 작업의 흐름이 `동시에` 진행되는 점이다.

**`차이점`**

> 프로세스는 독립적으로 실행되어 개별적으로 메모리를 할당받는다. 하지만 스레드는 프로세스 내에서 Stack 영역만 할당을 받고 Data, Code, Heap 영역은 여러 스레드가 공유할 수 있다는 점이 차이점이다.