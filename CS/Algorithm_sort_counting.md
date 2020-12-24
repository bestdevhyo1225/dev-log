## 계수 정렬 (Counting Sort)

---

### 계수 정렬 알고리즘 개념 요약

* 원소를 비교하지 않고 정렬하는 알고리즘이다.
* 각 원소가 몇개 있는지 갯수를 세서 정렬하는 알고리즘이다.
* 계수 정렬은 안정 정렬의 형태를 갖는다. 
    (안정 정렬? 같은 값을 가지는 복수의 원소들이 정렬 후에도 정렬 전과 같은 순서를 가지는 것이다.)



### 계수 정렬 알고리즘 과정

1. 원소를 입력한다.
2. counting 배열 index로 원소의 값을 넣고 count를 +1 한다.
3. 1 ~ 2번 과정을 반복한다.



### 계수 정렬의 심각한 단점
* 원소의 값의 범위가 굉장히 크다면 메모리를 많이 차지한다.


### 소스 코드
```c++
// Counting Sort
vector<int> countingSort(vector<int>& arr) {
    // 만약 원소 값의 범위가 1 ~ 1000이라면..
    vector<int> coutingArr(1001);
    for (int i = 0; i < arr.size(); i++) {
        countingArr[arr[i]]++;
    }
    
    vector<int> sorted;
    for (int num = 1; num <= 1000; num++) {
        if (countingArr[num] == 0) continue;
        while (countingArr[num]--) sorted.push_back(num);
    }

    return sorted;
}
```


### 계수 정렬 시간 복잡도

| Best  |  Avg  |  Worst  |
| :----:| :---: | :-----: |
| n + k | n + k | n + k   |

정렬을 위한 길이 n의 배열 하나, 계수를 위한 길이 k의 배열 하나.