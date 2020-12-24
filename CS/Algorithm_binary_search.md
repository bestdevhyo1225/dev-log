# 이분 탐색(Binary Search), Lower Bound, Upper Bound

<br>

## :book: 이분 탐색, 이진 탐색(Binary Search)

* 탐색 기법중 하나로 원하는 탐색 범위를 분할해서 찾는 방식이다.

* 원래 배열의 전체를 탐색하는 방식의 속도보다 훨씬 빠르다.

* 이분 탐색 하는 방법은 다음과 같다.

    1. 탐색하려는 배열을 오름차순으로 정렬해야 한다. (진짜 중요하다!)

    2. 배열의 시작(start), 끝(end)을 정해준다.

        `보통 start는 배열의 첫 번째 인덱스로 설정하고, end는 배열의 마지막 인덱스로 설정한다.`

    3. mid 라는 인덱스를 지정하여 탐색을 시작한다.

        `mid 의 값은 start + end 를 2로 나눈 값으로 한다.`

    4. 배열에서 mid 인덱스의 값과 찾고자하는 key값을 비교한다.

        `mid 인덱스의 값이 찾고자하는 key값보다 작으면, start = mid + 1로 인덱스를 지정한다.`

        `mid 인덱스의 값이 찾고자하는 key값보다 크면, end = mid - 1로 인덱스를 지정한다.`

        `mid 인덱스의 값과 찾고자하는 key값이 같다면, mid 인덱스를 반환한다.`

    5. 1 ~ 4번 과정을 반복하는데 start 인덱스가 end 인덱스보다 클때까지 수행한다. ( start <= end 일때, 탐색 작업 수행 )

* 이분 탐색 과정을 코드로 구현해보면 다음과 같다. ( 우리가 찾고자하는 값이 배열에 몇 번째에 있는지 알고싶을때 사용한다. )

```c++
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

int my_binary_search(vector<int>&arr, int key) {
    int start = 0, end = (int)arr.size() - 1;
    int answer = -1;
    while (start <= end) {
        int mid = (start + mid) / 2;
        if (arr[mid] < key) {
            start = mid + 1;
        } else if (arr[mid] > key) {
            end = mid - 1;
        } else { // 찾은 경우
            answer = mid;
            break;
        }
    }
    return answer;
}

int main() {
    vector<int> arr = { 1, 6, 9, 37, 21, 7, 12 };
    sort(arr.begin(), arr.end()); // 반드시 정렬!

    cout << my_binary_search(arr, 7) << '\n';

    return 0;
}
```

<br>

## :book: Lower Bound

* 이분 탐색(Binnary Search)기반의 탐색 방법이다. 

* 이분 탐색(Binnary Search)과 유사하지만 조금 다르다.

* 같은 원소가 여러개 있어도 상관 없으며, 항상 유일한 해를 구할 수 있다.

* 핵심은 **key값 이상(같거나 큰)인 여러 원소 중에서 가장 작은 원소의 위치를 찾는다.** 라고 할 수 있다.

    1. 탐색하려는 배열을 오름차순으로 정렬한다.

    2. 탐색할 시작 인덱스(start)와 끝(end)인덱스를 설정한다. 

        `start는 배열의 첫 번째 인덱스로 지정하고, end는 이분 탐색과는 조금 다르게 배열의 크기로 지정한다.`

        * end를 배열의 크기로 지정하는 이유?
            
            `우선 lower_bound() 함수는 반환하는 값이 end 값이라는 점이다.`
            
            `만약에 탐색하려는 Key값이 배열에서 가장 큰 원소보다 크면, 마지막 인덱스(배열의 크기)를 반환하기 위해서이다.`

            `그렇기 때문에 end의 초기 값을 배열의 사이즈로 지정하는 것이다.`

    3. mid 라는 인덱스를 지정하여 탐색을 시작한다.

        `mid 의 값은 start + end 를 2로 나눈 값으로 한다.`

    4. 배열에서 mid 인덱스의 값과 찾고자하는 key값을 비교한다.

        `mid 인덱스의 값이 찾고자하는 key값보다 크거나 같으면(arr[mid] >= key), end = mid로 인덱스를 지정한다.`

        `mid 인덱스의 값이 찾고자하는 key값보다 작으면, start = mid + 1로 인덱스를 지정한다.`

    5. 1 ~ 4번 과정을 반복하는데 start 인덱스가 end 인덱스와 같을때까지 수행한다. ( start < end 일때, 탐색 작업 수행 )

    6. end에 있는 인덱스 값을 반환한다. 
        
        **`key값 이상(같거나 큰)인 여러 원소 중에서 가장 작은 원소의 위치 (arr[mid] >= key)`**

* Lower Bound를 코드로 구현하면 다음과 같다.

```c++
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

int my_lower_bound(vector<int>& arr, int key) {
    int start = 0, end = (int)arr.size();
    while (start < end) {
        int mid = (start + end) / 2;
        if (arr[mid] >= key) {
            end = mid;
        } else {
            start = mid + 1;
        }
    }
    return end;
}

int main() {
    vector<int> arr = {10, 50, 20, 30, 20, 20, 30, 30, 10};
    sort(arr.begin(), arr.end()); // 반드시 정렬!

    cout << "my lower1 index : " << my_lower_bound(arr, 30) << '\n';
    cout << "my lower2 index : " << my_lower_bound(arr, 40) << '\n';
    cout << "my lower3 index : " << my_lower_bound(arr, 70) << '\n';
    
    // 결과
    // my lower1 index : 5
    // my lower2 index : 8
    // my lower3 index : 9
    return 0;
}
```

<br>

## :book: Upper Bound

* 이분 탐색(Binnary Search)기반의 탐색 방법이다. 

* 이분 탐색(Binnary Search)과 유사하지만 조금 다르다.

* 핵심은 **key값을 초과하는 가장 첫 번째 원소의 위치를 찾는다.** 라고 할 수 있다.

    1. 탐색하려는 배열을 오름차순으로 정렬한다.

    2. 탐색할 시작 인덱스(start)와 끝(end)인덱스를 설정한다. 

        `start는 배열의 첫 번째 인덱스로 지정하고, end는 이분 탐색과는 조금 다르게 배열의 크기로 지정한다.`

        * end를 배열의 크기로 지정하는 이유? ( Lower Bound 내용 참고 )

    3. mid 라는 인덱스를 지정하여 탐색을 시작한다.

        `mid 의 값은 start + end 를 2로 나눈 값으로 한다.`

    4. 배열에서 mid 인덱스의 값과 찾고자하는 key값을 비교한다.

        `mid 인덱스의 값이 찾고자하는 key값보다 크면(arr[mid] > key), end = mid로 인덱스를 지정한다.`

        `mid 인덱스의 값이 찾고자하는 key값보다 작거나 같으면, start = mid + 1로 인덱스를 지정한다.`

    5. 1 ~ 4번 과정을 반복하는데 start 인덱스가 end 인덱스와 같을때까지 수행한다. ( start < end 일때, 탐색 작업 수행 )

    6. end에 있는 인덱스 값을 반환한다. 
        
        **`key값을 초과하는 가장 첫 번째 원소의 위치 (arr[mid] > key)`**

* Upper Bound를 코드로 구현하면 다음과 같다.

```c++
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

int my_upper_bound(vector<int>& arr, int key) {
    int start = 0, end = (int)arr.size();
    while (start < end) {
        int mid = (start + end) / 2;
        if (arr[mid] > key) {
            end = mid;
        } else {
            start = mid + 1;
        }
    }
    return end;
}

int main() {
    vector<int> arr = {10, 50, 20, 30, 20, 20, 30, 30, 10};
    sort(arr.begin(), arr.end()); // 반드시 정렬!

    cout << "my upper1 index : " << my_upper_bound(arr, 30) << '\n';
    cout << "my upper2 index : " << my_upper_bound(arr, 40) << '\n';
    cout << "my upper3 index : " << my_upper_bound(arr, 70) << '\n';
    
    // 결과
    // my upper1 index : 8
    // my upper2 index : 8
    // my upper3 index : 9
    return 0;
}
```

<br>

## :bookmark: 참고

* [https://blockdmask.tistory.com/168](https://blockdmask.tistory.com/168)