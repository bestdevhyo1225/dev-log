## 기수 정렬 (Radix Sort)

---

### 기수 정렬 알고리즘 개념 요약

* 낮은 자릿수 부터 비교하여 정렬하는 알고리즘이다.
* 계수 정렬과 동일하게 비교를 하지 않는 알고리즘이다.
* 안정 정렬인 계수 정렬을 가지고 기수 정렬을 한다.
* 상대적인 위치가 바뀌면 안되기 때문에 안정 정렬을 사용한다.



### 기수 정렬 알고리즘 과정

1. 배열에 있는 원소중에서 값이 가장 큰 원소를 찾는다.
2. 가장 큰 원소의 자릿수를 구한다.
3. 1의 자리부터 가장 큰 원소의 자릿수 까지 확인한다.
    - 0 ~ 9값을 담는 queue를 10개 선언한다.
    - 해당 자릿수의 값이 0이면, queue[0]에 담고 해당 자릿수의 값이 5이면, queue[5]에 담는다.
    - queue에 있는 값들을 원래 배열에 다시 담는다.
4. 1 ~ 3의 과정을 반복한다.



#### **[ 1의 자리의 정렬 ]**

![image-20190620155903693](https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190620155903693.png?raw=true)

* **1의 자리에서 0 ~ 9 값을 저장할 Queue 공간을 할당합니다.**

![image-20190620160853270](https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190620160853270.png?raw=true)

* **1의 자리의 값을 구한 다음, 그 값이 해당하는 Queue 자리에 저장합니다.**

![image-20190620161146692](https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190620161146692.png?raw=true)

* **1의 자리 정렬한 결과는 다음과 같습니다.**



#### **[ 10의 자리의 정렬 ]**

![image-20190620161412498](https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190620161412498.png?raw=true)

* **10의 자리에서 0 ~ 9 값을 저장할 Queue 공간을 할당합니다.**

![image-20190620161736645](https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190620161736645.png?raw=true)

* **10의 자리 값을 구한 다음에 값이 해당하는 Queue 자리에 저장합니다. **

![image-20190620161938762](https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190620161931875.png?raw=true)

* **10의 자리를 정렬한 결과는 다음과 같습니다.**



#### **[ 100의 자리의 정렬 (마지막) ]**

![image-20190620162127087](https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190620162127087.png?raw=true)

* **100의 자리에서 0 ~ 9 값을 저장할 Queue 공간을 할당합니다.**

![image-20190620162404126](https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190620162404126.png?raw=true)

* **100의 자리 값을 구한 다음에 값이 해당하는 Queue 자리에 저장합니다. **

![image-20190620162516588](https://github.com/bestdevhyo1225/image_repository/blob/master/image-20190620162516588.png?raw=true)

* **100의 자리를 정렬한 결과는 다음과 같습니다.**



### 소스 코드
```c++
// Radix Sort
#include <iostream>
#include <algorithm>
#include <string>
#include <vector>
#include <queue>
using namespace std;

size_t getMaxSize(vector<int>& arr) {
    return to_string(*max_element(arr.begin(), arr.end())).length();
}

void radixSort(vector<int>& arr) {
    queue<int> q[10];
    int tenSquared = 1;
    int maxSize = (int)getMaxSize(arr);
    for (int i = 0; i < maxSize; i++) {
        for (int j = 0; j < arr.size(); j++) {
            q[(arr[j] / tenSquared) % 10].push(arr[j]);
        }
        int index = 0;
        for (int k = 0; k < 10; k++) {
            while (!q[k].empty()) {
                arr[index++] = q[k].front();
                q[k].pop();
            }
        }
        tenSquared *= 10;
    }
}
```



### 기수 정렬 알고리즘 시간 복잡도

| Best  |  Avg  |  Worst  |
| :----:| :---: | :-----: |
| d * n | d * n | d * n   |

d는 자릿수를 의미하며, n은 배열의 길이를 의미한다. 



### 참고 사이트
[정렬 알고리즘 - 기수 정렬](https://lktprogrammer.tistory.com/48)