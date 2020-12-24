## 그래프 탐색 알고리즘 - 깊이 우선 탐색 (Depth First Search, DFS)

<br>

### :book: 깊이 우선 탐색 (Depth First Search, DFS)

* 트리의 순회와 같이 그래프의 모든 정점들을 특정한 순서에 따라 방문하는 알고리즘이다.

    `그래프에서 특정 정점을 찾아내는 인상을 받기 쉬운데, 특정 정점이나 최단 경로를 찾기 위한 알고리즘보다는 정점들을 정해진 순서대로 둘러보기 위한 알고리즘에 더 가깝다.`

* 트리의 순회는 사실 트리에 있는 모든 정점들을 모두 확인한다는 것 외에는 큰 의미가 없다.

* 그래프는 트리보다 구조가 훨씬 복잡할 수 있기 때문에 탐색 과정에서 얻어지는 정보가 아주 중요하다.

* 탐색 과정에서 다음과 같은 정보를 통해 그래프의 구조를 알 수 있다.

    `어떤 간선이 사용되었는지?`

    `어떤 순서로 정점들이 방문되었는지?`

* 그래프의 모든 정점을 발견하는 가장 단순하고, 고전적인 방법이다.

    `현재 정점과 인접한 간선들을 하나씩 검사하다가, 아직 방문하지 않은 정점으로 향하는 간선이 있다면 그 간선을 무조건 따라가는 것이다. 이 과정에서 더 이상 갈 곳이 없는 막힌 정점에 도달하면 포기하고, 마지막에 따라왔던 간선을 따라 뒤로 돌아간다.`

<br>

### :book: 깊이 우선 탐색(DFS)의 과정

![image-20190807204100696](https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190807204100696.png?raw=true)

1. 시작 정점은 0번 정점이고, 0번 정점을 방문 했음을 표시한다.

2. 1번 정점을 탐색한다. `1번 정점을 방문했으면, 방문 표시한다.`

3. 2번 정점을 탐색한다. `2번 정점을 방문했으면, 방문 표시한다.`

4. 3번 정점을 탐색한다. `3번 정점을 방문했으면, 방문 표시한다.`

5. 4번 정점을 마지막으로 탐색한다. `4번 정점을 방문했으면, 방문 표시한다.`

6. 4번 정점에서는 더 이상 탐색할 정점이 없으므로 이전 정점인, 3번 정점으로 돌아간다.

7. 3번 정점으로 돌아 왔을때 3번 정점과 연결되어 있는 1번 정점과 2번 정점을 확인한다.( 4번 정점은 직전에 방문했음으로 확인 x )
    더 이상 탐색할 정점이 없음을 판단하고, 이전 정점인 2번 정점으로 돌아간다.

    `1번 정점과 2번 정점은 이미 방문이 완료된 상태이기 때문에 탐색할 필요가 없다.`

8. 2번 정점으로 돌아 왔을때 2번 정점과 연결되어 있는 1번 정점, 4번 정점을 확인한다.( 3번 정점은 직전에 방문했음으로 확인 x)
    더 이상 탐색할 정점이 없음을 판단하고, 이전 정점인 1번 정점으로 돌아간다.

    `1번 정점과 4번 정점은 이미 방문 완료된 상태`

9. 1번 정점으로 돌아 왔을때 1번 정점과 연결되어 있는 0번 정점, 3번 정점을 확인한다.( 2번 정점은 직전에 방문했음으로 확인 x)
    더 이상 탐색할 정점이 없음을 판단하고, 이전 정점인 0번 정점으로 돌아간다.

    `0번 정점과 3번 정점은 이미 방문 완료된 상태`

10. 0번 정점으로 돌아오면 종료한다.

* 현재 그래프는 간단한 형식이기 때문에 DFS 과정이 복잡하지 않았지만, 그래프의 정점과 간선이 많아지면 DFS 과정이 복잡해진다.

* DFS 알고리즘은 주로 `재귀 호출`을 통해 구현한다. ( 1 ~ 10번 과정이 재귀 호출로 수행됨 )

<br>

### :book: 깊이 우선 탐색(DFS) 구현

* 위의 그림을 이용해서 깊이 우선 탐색 알고리즘을 구현해보자

```c++
#include <iostream>
#include <vector>

using namespace std;

const int VERTEX_SIZE = 5;
vector<int> adj[VERTEX_SIZE];
vector<bool> visited;

void dfs(int current) {
    cout<< "현재 정점 : " << current << '\n'; 
    for (int i = 0; i < adj[current].size(); i++) {
        int next = adj[current][i];
        if (!visited[next]) {
            visited[next] = true;
            dfs(next);
        }
    }
}

int main() {
    // 양방향 그래프라고 가정!
    // 그래프 모델링을 한다. (인접 리스트로 구현)
    // 0번 정점의 인접 리스트 구현 (0번 정점과 연결되어 있는 정점의 번호 저장)
    // 0 -> 1, 4
    adj[0].push_back(1);
    adj[0].push_back(4);

    // 1번 정점의 인접 리스트 구현 (1번 정점과 연결되어 있는 정점의 번호 저장)
    // 1 -> 0, 2, 3
    adj[1].push_back(0);
    adj[1].push_back(2);
    adj[1].push_back(3);

    // 2번 정점의 인접 리스트 구현 (2번 정점과 연결되어 있는 정점의 번호 저장)
    // 2 -> 1, 3, 4
    adj[2].push_back(1);
    adj[2].push_back(3);
    adj[2].push_back(4);

    // 3번 정점의 인접 리스트 구현 (3번 정점과 연결되어 있는 정점의 번호 저장)
    // 3 -> 1, 2, 4
    adj[3].push_back(1);
    adj[3].push_back(2);
    adj[3].push_back(4);

    // 4번 정점의 인접 리스트 구현 (4번 정점과 연결되어 있는 정점의 번호 저장)
    // 4 -> 0, 2, 3
    adj[4].push_back(0);
    adj[4].push_back(2);
    adj[4].push_back(3);

    visited = vector<bool>(VERTEX_SIZE);
    visited[0] = true;  // 시작 정점 방문 표시
    dfs(0); // dfs 수행

    return 0;
}
```

* Baekjoon Online Judge 사이트에 다음과 같은 문제가 있는데 풀어보면 좋을것 같다.

    * [1260번 - DFS와 BFS](https://www.acmicpc.net/problem/1260)