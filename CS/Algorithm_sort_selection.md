## 선택 정렬 (Selection Sort)

---

### 선택 정렬 알고리즘 개념 요약

* 제자리 정렬(in-place sorting) 알고리즘의 하나
* 해당 순서에 원소를 넣을 위치는 이미 정해져 있다. (핵심! 위치가 정해져 있다.)
* 위치는 정해져있고 **어떤 원소**를 넣을지 선택하는 과정



### 선택 정렬 알고리즘 과정

* 오름차순 기준
1. 해당 원소들 중에서 가장 작은 값을 가진 원소를 선택 한다.
2. 가장 작은값을 가진 원소를 선택했으면, 넣을 위치에 있는 원소와 교환 한다.
3. 1과 2의 과정을 반복해서 정렬 한다. 



### 소스코드

```c++
// selection sort
for (int i = 0; i < n-1; i++) {
    int minIndex = i;
    for (int j = i+1; j < n; j++) {
        if (arr[minIndex] > arr[j]) {
            minIndex = j;
        }
    } 
    if (minIndex != i) {
        swap(arr[minIndex], arr[i]);
    }
}
```



### 선택 정렬 알고리즘 시간 복잡도

| Best  |  Avg  |  Worst  |
| :----:| :---: | :-----: |
| n^2   | n^2   | n^2     |



### 참고사이트
* [선택 정렬(Selection Sort)이란?](https://gmlwjd9405.github.io/2018/05/06/algorithm-selection-sort.html)