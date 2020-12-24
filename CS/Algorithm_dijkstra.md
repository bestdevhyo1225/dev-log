## 다익스트라 (Dijkstra)

---

### 다익스트라 (Dijkstra) 란?

- **하나의 정점에서 다른 모든 정점까지의 최단 경로를 구하는 문제이다. (single source shortest path problem)**
- 가중치가 존재하지 않으면 BFS (너비 우선 탐색) 이 가능하지만 가중치가 존재 한다면 다익스트라 (Dijkstra) 알고리즘으로 구현해야 한다.
- 음수 가중치를 갖는 간선이 존재할 경우에는 사용할 수 없다.

![image](https://user-images.githubusercontent.com/23515771/103049326-d989c480-45d4-11eb-85af-df87cc011058.png)

- 위의 그림은 너비 우선 탐색(BFS)이 최단 거리를 찾지 못하는 하나의 예를 보여주고 있다.

- 너비 우선 탐색은 자기 자신과 인접한 정점을 먼저 방문한다. s 정점에서 a 정점과 c 정점을 방문하게 되고 그 이후에 b 정점을 방문 한다. 하지만 s 정점에서 c 까지 가는 최단 경로는 s -> a -> b -> c ( 2 + 4 + 3 ) 이다. 최단 거리 순서대로 정점들을 방문하려면 a, b, c 순서로 각 정점을 방문해야만 한다.

- 이것이 의미하는 바는 더 늦게 발견한 정점이라도 더 먼저 방문할 수 있어야 한다는 의미이다.

### 다익스트라 (Dijkstra) 구현 방법 - 우선 순위 큐 (Priority Queue) 방식

- 내가 다익스트라 문제를 해결할 때 주로 사용하는 방식은 **우선 순위 큐(Priority Queue)** 자료구조로 문제를 해결하곤 한다.
- c++ 기준으로 보자면 너비 우선 탐색 (BFS) 에서는 큐에 다음 정점의 번호를 넣었지만, 다익스트라 알고리즘을 해결해야하는 우선 순위 자료구조에서 방법이 조금 다르다. 우선 순위 큐에는 **다음 정점의 번호와 지금까지 찾아낸 해당 정점까지의 최단 거리를 쌍으로 넣는다.**

```c++
// BFS - Queue
queue<int> q;
q.push(vertex);

// Dijkstra - Priority Queue
priority_queue<pair<int, int> > pq;
pq.push(make_pair(0, vertex));
```

- 주의할 점은 pair<int, int> 에서 첫 번째 원소를 먼저 비교하므로 정점까지의 거리를 첫 번째 원소로 두고, 정점의 번호를 두 번째 원소로 하는 것을 볼 수 있다.
- priority_queue 는 기본적으로 **가장 큰 원소가 위로 가도록 큐를 구성**하기 때문에 거리의 부호를 바꿔서 거리가 작은 정점부터 꺼내지도록 한다.

```c++
priority_queue<pair<int, int> > pq;
pq.push(make_pair(0, vertex));

while (!pq.empty()) {
  int currentDist = -pq.top().first;	// 부호를 바꿔 준다.
  int currentVertex = pq.top().second;
  pq.pop();
  ...
  ...
  ...
  // 나중에 큐에 담을때도 부호를 바꿔준다.
  pq.push(make_pair(-nextDist, nextVertex));
}
```

- 또 한가지 유의해야 할 사항이 있다. 아래 그림에 그래프를 다시 보자

![image](https://user-images.githubusercontent.com/23515771/103049326-d989c480-45d4-11eb-85af-df87cc011058.png)

- 시작 정점이 s 를 방문하게 되면, 큐에는 (-2, a) 그리고 (-12, c) 가 들어간다.

![image](https://user-images.githubusercontent.com/23515771/103049379-faeab080-45d4-11eb-95df-dadda6cf2b68.png)

- 그 다음 a 정점을 큐에서 꺼낸 뒤 인접한 b 정점을 방문하면서 (-4, b) 를 큐에 넣게 된다.

![image](https://user-images.githubusercontent.com/23515771/103049441-16ee5200-45d5-11eb-9541-f229fdd8fe7d.png)

- **가장 큰 원소가 위로 가도록 하는 우선 순위 큐** 의 특징을 통해 b 정점을 큐에서 꺼낸 뒤 인접한 c 정점을 방문하게 되는데, 우리가 이미 알고 있던 c 정점의 길이인 12보다 더 짧은 9의 길이(2 + 4 + 3)로 이동할 수 있다는것을 알게 된다. 이 때 길이의 정보를 갱신해야 하는데 두 가지 방법이 있다.

  1. 우선 순위 큐 내에서 (-12, c) 를 찾아내 (-9, c) 로 바꾼다.
  2. (12, c) 를 그대로 두고 (9, c) 를 추가한 뒤, 나중에 큐에서 (12, c) 가 꺼내지면 무시한다.

- 대게 실제로 사용하는 방법은 후자이다. 전자의 연산은 대개 표준 라이브러리의 우선 순위 큐에서 지원하지 않을 뿐더러 직접 구현하기에는 복잡하고 까다롭다.

```c++
vector<int> dist(VertexCount, INF);	// INF 값으로 초기화 해야한다.

priority_queue<pair<int, int> > pq;
pq.push(make_pair(0, vertex));

while (!pq.empty()) {
  int currentDist = -pq.top().first;	// 부호를 바꿔 준다.
  int currentVertex = pq.top().second;
  pq.pop();
  // dist 배열에 기록되어 있는 현재 정점의 값이 우선순위 큐에 들어있었던 기존의 값보다 작다면 무시한다.
  // 더 이상 밑에 연산을 할 필요가 없다.
  // why? 다음 거리들은 currentDist + ? 이기 때문에 dist 배열에 기록되어 있는 값보다
  // 무조건 커질 수 밖에 없기 때문에 continue 로 무시한다.
  if (dist[currentVertex] < currentDist) continue;
  for (int i = 0; i < adj[currentVertex].size(); i++) {
    int nextDist = currentDist + adj[currentVertex][i].second;
    int nextVertex = adj[currentVertex][i].first;
    if (dist[nextVertex] > nextDist) {
      dist[nextVertex] = nextDist;
      pq.push(make_pair(-nextDist, nextVertex));
    }
  }
}
```

- if 조건이 성립하는 이유는 현재 정점의 dist 배열에 있는 값은 최솟값이고, 다음 연산을 진행 하더라도 'currentDist + ?(특정 거리)' 값은 계속해서 커지게 된다. 따라서 dist[currentVertex] 값보다 절대로 작아질 수 없기 때문에 무시하고 불 필요한 연산을 막는 것이다.

---

### 참고

- 프로그래밍 대회에서 배우는 알고리즘 문제 해결 전략
- [https://hsp1116.tistory.com/42](https://hsp1116.tistory.com/42)
