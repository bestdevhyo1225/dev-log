## 합병 정렬 (Merge Sort)

---

### 합병 정렬 알고리즘 개념 요약

* 안정 정렬에 속한다.
* 분할 정복 알고리즘의 하나이다. (분할, 정복, 결합)
* 분할 (Divide) : 입력 배열을 같은 크기의 2개의 부분 배열로 분할한다.
* 정복 (Conquer) : 부분 배열을 정렬한다.
* 결합 (Combine) : 정렬된 배열들을 하나의 배열에 합병한다.



### 합병 정렬 알고리즘 과정

* 오름차순 기준
1. 입력 배열을 2개의 부분 배열로 나누는 과정을 진행한다.
2. 부분 배열을 오름차순 기준으로 정렬하는 과정을 진행한다.
3. 합병하는 과정을 진행한다.
4. 1 ~ 3 과정을 반복하며 정렬한다.



### 소스 코드
```c++
// merge sort
void merge(vector<int>& arr, int start, int mid, int end) {
    vector<int> sorted(arr.size());
    int idx = start, i = start, j = mid+1;
    while (i <= mid && j <= end) {
        if (arr[i] < arr[j]) sorted[idx++] = arr[i++];
        else sorted[idx++] = arr[j++];
    }
    while (i <= mid) sorted[idx++] = arr[i++];
    while (j <= end) sorted[idx++] = arr[j++];
    for (int k = start; k <= end; k++) arr[k] = sorted[k];
}

void mergeSort(vector<int>& arr, int start, int end) {
    if (start < end) {
        int mid = (start + end) / 2;
        mergeSort(arr, start, mid);
        mergeSort(arr, mid+1, end);
        merge(arr, start, mid, end);
    }
}
```



### 합병 정렬 알고리즘 시간 복잡도

| Best  |  Avg  |  Worst  |
| :----:| :---: | :-----: |
| nlogn | nlogn | nlogn   |



### 참고 사이트
[합병 정렬(Merge sort)이란?](https://gmlwjd9405.github.io/2018/05/08/algorithm-merge-sort.html)