## 퀵 정렬 (Quick Sort)

---

### 퀵 정렬 알고리즘 개념 요약

* 퀵 정렬은 **불안정 정렬**에 속한다.
* 다른 원소와의 비교만으로 정렬을 수행하는 **비교 정렬**에 속한다.
* **분할 정복** 알고리즘의 하나이다.
* 평균적으로 **매우 빠른 수행 속도**를 자랑하는 정렬 방법



### 퀵 정렬 알고리즘 과정

1. 분할(Divide) : 입력 배열을 pivot 기준으로 비균등하게 2개의 부분 배열로 분할한다. 
    (pivot을 중심으로 왼쪽은? pivot보다 작은 원소로만 이루어져 있다.)
    (pivot을 중심으로 오른쪽은? pivot보다 큰 원소로만 이루어져 있다.)
2. 정복(Conquer) : 부분 배열을 정렬한다.
3. 결합(Combine) : 정렬된 부분 배열들을 하나의 배열로 병합한다.



### 소스 코드
```c++
// quick sort
#include <vector>
using namespace std;

void swap(vector<int>& arr, int s, int e) {
    int temp = arr[s];
    arr[s] = arr[e];
    arr[e] = temp;
}

int partition(vector<int>& arr, int s, int e) {
    int pivot = arr[(s + e) / 2];
    while (s <= e) {
        while (pivot > arr[s]) s++;
        while (pivot < arr[e]) e--;
        if (s <= e) {
            swap(arr, s, e);
            s++;
            e--;
        }
    }
    return s;
}

void quickSort(vector<int>& arr, int s, int e) {
    int p = partition(arr, s, e);
    if (s < p-1) quickSort(arr, s, p-1);
    if (p < e) quickSort(arr, p, e);
}
```

### 적당히 잘 구현하려면??
* 중간 크기의 숫자를 pivot으로 선정하는 것이 효율적이다. 대부분 랜덤으로 값 3개를 추출하여 중위값을 pivot으로 지정한다.



### 퀵 정렬 알고리즘 시간 복잡도

| Best  |  Avg  |  Worst  |
| :----:| :---: | :-----: |
| nlogn | nlogn | n^2   |