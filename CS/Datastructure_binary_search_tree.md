# 이진 검색 트리(Binary Search Tree, BST)

<br>

* 트리는 현실 세계의 계층적 구조를 표현하는 것 외에도 다양한 용도로 사용되는데 그 중 대표적인 것이 **검색 트리** 이다.

* 검색 트리는 자료를 담는 컨테이너이고, 자료들을 일정한 순서에 따라 **정렬한 상태**로 저장한다.

    `32비트 정수들을 작은 것부터 큰 것까지 정렬한 상태로 저장할 수도 있고, 문자열을 가나다순으로 정렬해서 저장할 수도 있다.`

* 검색 트리는 이 점을 이용해서 원소의 **추가**와 **삭제**만이 아니라 특정 원소의 **존재 여부 확인** 등의 다양한 연산을 빠르게 수행한다.

* 검색 트리가 많은 곳에서 사용되고 있다.

    `1. 이미 가입한 사용자들의 주민등록번호를 저장해 두고, 특정 사용자가 이미 가입되었나 찾기`

    `2. 사용자의 ID를 사용자 정보로 대응시키는 사전 객체 만들기`

    `3. 모든 학생들의 시험 점수를 저장해 놓고, 나보다 1등 위인 사람과 1등 아래인 사람을 찾기`

<br>

## :book: 이진 검색 트리(Binary Search Tree, BST)

### :pencil2: 정의

* 이진 트리란 각 노드가 왼쪽과 오른쪽, 최대 두 개의 자식 노드만을 가질 수 있는 트리를 말한다.

* 이진 검색 트리는 이진 탐색(Binary Search)에서 아이디어를 가져와서 만든 트리이다.

* 각 노드의 왼쪽 서브트리에는 해당 노드의 원소보다 작은 원소를 가진 노드들이 있고, 오른쪽 서브트리에는 해당 노드의 원소보다 큰 원소를 가진 노드들이 있다.

* 이진 검색 트리에서 원하는 값을 찾는 과정은 배열에서의 이진 탐색과 비슷하다.

| <img src="https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190805122252217.png?raw=true" width="400" height="300"> | <img src="https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190805122011604.png?raw=true" width="400" height="300"> |
| :-------------------: | :---------------------: |
| (a) 이진 검색 트리의 옳은 예 | (b) 이진 검색 트리의 잘못된 예 |

* 그림 (b) 의 트리가 이진 검색 트리가 아닌 이유는 무엇인지 생각해보기

<br>

### :pencil2: 순회

* 이진 검색 트리를 **중위 순회** 하면 크기 순서로 정렬된 원소의 목록을 얻을 수 있다.

    `현재 노드가 가진 원소보다 작은 원소들은 모두 왼쪽 서브트리에 있고, 그 보다 큰 원소들은 모두 오른쪽 서브트리에 있기 때문이다.`

* 트리를 중위 순회하면 정렬된 결과를 얻을 수 있다는 말은, 집합에 포함된 최대 원소나 최소 원소를 쉽게 얻을 수 있다는 애기이다.

    `중위 순회에서 가장 일찍 출력되는 노드는 루트에서 내려갈 수 없을때 까지 왼쪽으로 연결된 노드이며, 이 노드가 최소 원소가 된다. 반대로 오른쪽을 따라 계속 내려갔을때 만나는 노드가 최대 원소가 된다.`

<br>

### :pencil2: 자료의 검색

* 이진 검색 트리는 아주 간단하게 특정 원소가 존재하는지 확인할 수 있다.

    `그림 (a) 의 이진 검색 트리에서 15가 포함되었는지 확인할 때, 루트인 16을 보는 것만으로 트리의 어느쪽에서 15를 찾아봐야 할지 알 수 있다. 이진 검색 트리는 한 번 원소를 비교하는 것만으로 찾아야할 대상의 절반을 줄일 수 있기 때문에 실질적으로는 이진 탐색과 비슷한 속도로 자료를 찾는다.`

<br>

### :pencil2: 조작

* 지금까지 알아본 이진 검색 트리의 특성을 살펴보자면 정렬된 배열에 비해 나을 것이 없다.

    `정렬된 배열이라면 최대 원소와 최소 원소를 쉽게 구할 수 있고, 순회도 이진 검색을 이용하면 트리에 특정 원소가 존재하는지 빠르게 파악할 수 있다.`

* 이진 검색 트리가 진가를 드러내는 곳은 집합에 **원소를 추가**하거나 **삭제**하는 조작 연산을 할 때이다.

    * 원소 추가

        `정렬된 배열에서 새 원소를 삽입하려면 삽입할 위치를 찾고, 그 이후에 있는 원소들을 모두 한 칸씩 뒤로 옮겨야 한다. 하지만 이진 검색 트리에는 선형적인 구조의 제약이 없기 때문에 새 원소가 들어갈 위치를 찾고 노드만 추가하면 된다.`

    * 원소 삭제

        `삭제를 구현하는 여러가지 방법 중에서 '합치기' 연산을 구현하면 된다. (과정이 복잡하기 때문에 생략)`

<br>

## :book: 균형 잡힌 이진 검색 트리

### :pencil2: 시간 복잡도

* 이진 검색 트리에 대한 모든 연산은 모두 루트에서부터 한 단계씩 트리를 내려가며 수행된다. (보통 재귀 호출을 통해 수행됨)

* 재귀 호출을 통해 연산을 한다고 했을때, 호출의 횟수는 트리의 높이(h)와 같다. 따라서 모든 연산의 시간 복잡도가 트리의 높이 O(h)라고 할 수 있다.

<br>

### :pencil2: 기울어진 트리 (Skewed Tree)

* 입력이 특정한 순서로 들어와서 트리가 한 쪽으로 치우쳐져 시간 복잡도가 O(N)인 트리이다.

    `원소가 내림차순 or 오름차순으로 들어오는 경우`

<br>

### :pencil2: 레드-블랙 트리 (Red-Black Tree)

* 균형 잡힌 이진 검색 트리 종류중에 하나이고, 기울어진 트리가 되는 단점을 개선한 이진 검색 트리의 변종이다.

* 트리의 구조에 추가적인 제약을 정하고 이 제약이 만족되도록 노드들을 옮겨서 트리의 높이가 항상 O(logN)이 되도록 유지한다.

* 자세한 내용은 추후에 정리하려고 한다. (AVL 트리와 함께 정리할 것)

<br>

## :bookmark: 참고 문헌

* 프로그래밍 대회에서 배우는 알고리즘 문제 해결 전략 - 구종만