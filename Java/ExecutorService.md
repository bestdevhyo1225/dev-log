# ExecutorService

## 최적의 쓰레드 풀 수

Java Concurrency in Practice에 나오는 내용

```
N Thread = N CPU * U CPU * (1 + W/C)
```

- `N CPU` : `Runtime.getRuntime().availableProcessors()` 가 반환하는 Core 갯수
- `U CPU` : 0 ~ 1 사이의 값을 갖는 CPU 활용 비율
- `W/C` 는 `대기시간(I/O 시간 등)` 과 `계산 시간(CPU가 직접 작동하는 시간)` 의 비율이다.
  - I/O가 높으면 100, I/O는 낮고 CPU 활용시간이 많으면 0에 가깝게 설정한다.
- 쓰레드 수가 너무 많으면 오히려 서버가 크래시 될 수 있으므로 하나의 `Executor에서 사용할 쓰레드의 최대 개수는 100이하로 설정하는게 바람직하다.`
