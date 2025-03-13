---
title: =를 넣을지 말지 영원히 헷깔리는 이진 탐색 문제 정복하기
categories:
  - DSA
tags:
  - leetcode
  - algorithm
---

[Neetcode](https://neetcode.io/)는 leetcode에서 주요한 문제들을 카테고리별로 묶어서 공부할 수 있게 만들어둔 사이트.

#### 이진탐색이란

이진탐색은 정렬된 배열에서 특정 값을 빠르게 찾는 알고리즘으로, 탐색 범위를 절반씩 줄여 나가면서 O(log N)의 시간 복잡도를 갖는 것이 특징.

기본적인 개념은 다음과 같습니다:
1. 중앙값을 선택하여 찾고자 하는 값과 비교합니다.
2. 찾고자 하는 값이 중앙값보다 작다면 왼쪽 절반을, 크다면 오른쪽 절반을 탐색합니다.
3. 위 과정을 반복하여 결국 값을 찾거나 탐색 범위가 사라질 때까지 진행합니다.

이진탐색은 하나의 차이로 인해 경계 조건(=)을 어디에 넣느냐가 중요한 알고리즘. 특히, 반복문 조건과 중간값(mid) 갱신 방식에 따라 구현 방식이 달라질 수 있음. 경계를 포함하는지 여부(=)에 따라 탐색 종료 조건과 mid 갱신 방식이 달라지므로, 구현 시 주의해야 함.

또한, Python의 bisect 모듈을 활용하면 직접 구현하지 않고도 효율적으로 이진탐색을 수행할 수 있습니다.
* bisect_left(arr, x): x가 들어갈 가장 왼쪽 인덱스를 반환. (x가 이미 존재하면, 기존 위치 중 가장 왼쪽을 반환)
* bisect_right(arr, x): x가 들어갈 가장 오른쪽 인덱스를 반환. (x가 이미 존재하면, 기존 위치의 바로 다음 인덱스를 반환)

```python
import bisect

arr = [1, 3, 3, 5, 7]

print(bisect.bisect_left(arr, 3))  # 1 (3이 처음 등장하는 위치)
print(bisect.bisect_right(arr, 3)) # 3 (3 다음에 들어갈 위치)
```

#### [704. Binary Search](https://leetcode.com/problems/binary-search/description/) (Easy)

문제의 포인트
* an array of integers nums which is sorted in ascending order: 이미 정렬되어 있다!
* write an algorithm with O(log n) runtime complexity: 하나씩 탐색하면 최소 O(n). O(logn)을 달성하려면 이진 탐색을 써야한다.
* If target exists, then return its index. Otherwise, return -1: While 루프 안에서 처리가 안되면 -1 리턴.

```python
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        l = 0
        r = len(nums)
        while l < r:
            mid = (l + r) // 2
            num = nums[mid]
            if num == target: # target을 정확히 찾는 문제이므로 찾으면 ealry return.
                return mid
            elif num < target:  # 위에서 ealry return을 했기 때문에 경계 조건을 처리하기가 맘편하다.
                l = mid + 1
            else:
                r = mid
        return -1
```

#### [74. Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/description/) (Medium)

이 문제는 사실 전체가 sort가 되어있다고 했으니, target보다 큰 수가 나오기 전까지 아래로 내려갔다가, 해당 row에서 오른쪽으로만 가서 종료해야겠군 하고 생각했고 beat 100%가 나왔다. binary search로 분류되는 이유를 알아보니 전체가 sort 되어 있으니 전체 칸을 하나의 긴 list로 생각해서 이진탐색해서 찾는다고 한다. 생각해보니 내 방법은 1xn이나 nx1이 나오고 target이 맨 마지막 숫자라면 최악의 경우 O(m+n)이겠구나 하는 생각은 들었다. (leetcode의 testcase가 worst case만 테스트하진 않나보다.)
`O(m+n)`이 오래걸리냐 `O(log(m*n))`이 오래걸리냐는 `O(m+n)`은 결국은 `O(n)`이고 `O(log(m*n))`은 `O(log(m)+log(n))`이니 `O(logn)`이니 후자가 훨씬 낫겠다. (여기서 곱의 로그를 오랜만에 되돌려보았다.)

```python
class Solution(object):
    # 최초 Solution O(m+n)
    def searchMatrix(self, matrix, target):
        """
        :type matrix: List[List[int]]
        :type target: int
        :rtype: bool
        """
        x_pos = 0
        y_pos = 0

        if len(matrix) == 0:
            return False

        while x_pos < len(matrix[0]):
            if matrix[y_pos][x_pos] == target:
                return True
            if (y_pos < len(matrix) - 1) and matrix[y_pos+1][x_pos] <= target:
                y_pos += 1
                continue
            x_pos += 1
        return False
```
```python
class Solution(object):
    # 이진트리 적용한 버젼. O(log(width*height)) -> O(logn)
    def searchMatrix(self, matrix, target):
        """
        :type matrix: List[List[int]]
        :type target: int
        :rtype: bool
        """
        l = 0
        height = len(matrix)
        width = len(matrix[0])
        r = height * width

        while l < r:
            mid = (l+r)//2
            num = matrix[mid//width][mid%width] # 요부분에서 matrix에서 값을 읽는 것만 처리해주면 위 문제와 똑같아진다.
            if num == target:
                return True
            elif num > target:
                r = mid
            else:
                l = mid + 1
        return False
```

#### [875. Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/description/) (Medium)

처음 떠올린 생각.
```text
maximum 10(array의 모든 값의 합) hours / 1 banana per hour
minimum 4(length) hours / 4 bananas per hour
maximum 9 hours / k? bananas per hour
```
첫번째 예시 기준, 나이브하게는 배열 중 가장 큰 숫자인 4개씩 먹으면 length 만큼이면 다 먹을 수 있다. 혹은 무조건 1개씩 먹으면 10시간이면 다먹을 수 있다. 이 때 9시간안에 먹기 위한 가장 작은 k를 구하는 것이 문제이다. k=1이 최소고, k=4가 max일 때 소요시간이 9보다 작거나 같은 최적의 k값을 구해야한다.

k = 1부터 하나씩 올라가면서 h안에 먹을 수 있는 첫 숫자를 만났을 때 리턴하면 O(n). 이거보다 효율적으로 해야 O(logn)이 가능하다. 여기서 binary search의 등장.

이진 탐색에서 범위를 줄여나갈 때, 탐색 종료 후 l이 최적의 k 값이 되도록 하기 위해 비포함 경계(while l < r)를 유지해야 한다.
* l = 1 (최소 속도: 한 시간에 1개씩 먹기)
* r = max(piles) (최대 속도: 한 시간에 가장 큰 더미를 한 번에 먹는 속도)
* while l < r을 유지하면 l==r일 때 탐색이 종료되고, 탐색 종료 시점에 l이 정답을 보장함.

```python
class Solution(object):
    def minEatingSpeed(self, piles, h):
        """
        :type piles: List[int]
        :type h: int
        :rtype: int
        """
        l = 1
        r = max(piles)

        def cal_hours(k):
            h = 0
            for bananas in piles:
                h += bananas // k
                h += 1 if(bananas%k) else 0
            return h

        while l < r:
            k = (l + r) //2
            consuming_time = cal_hours(k)
            if consuming_time > h:
                # 목표 시간보다 많이 걸림. 한시간에 더 많이 먹어야 함.
                l = k+1
            else: # 목표시간과 같거나 적게 걸림. 한시간에 조금 덜 먹어보자.
                r = k
        return l
```

#### [153. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/)

처음 이 문제를 접했을 때, 이진 탐색(Binary Search)으로 해결할 수 있다는 것이 직관적으로 와닿지 않았다.
특히, 예외 없이 절반씩 탐색 범위를 줄일 수 있다는 점이 이해하기 어려웠다.

__문제의 핵심: 회전된 정렬 배열__

이 배열은 원래 오름차순으로 정렬된 상태에서 한 번 회전된 형태다.
즉, 하나의 정렬된 배열을 두 개의 정렬된 부분 배열로 나눈 후 합친 것과 같다.

예를 들어, 배열 [3,4,5,6,1,2] 를 보면:
* [3,4,5,6] (첫 번째 그룹)
* [1,2] (두 번째 그룹)

이렇게 두 개의 정렬된 구간으로 나뉘어 있으며, 우리가 찾아야 하는 최소값은 두 번째 그룹의 첫 번째 원소다.
또한, 배열의 마지막 값(nums[r])은 항상 두 번째 그룹의 마지막 값이 된다.

* Case 1. nums[mid] < nums[r]: 둘이 정상적으로 정렬되어 있으므로 mid부터 r까지가 같은 그룹2에 있음. 최소값(그룹2의 첫번째 값)은 mid이거나 mid보다 앞에 존재. r=mid로 갱신하여 mid 이후의 값들을 탐색 범위에서 제외한다.

* Case 2. nums[mid] > nums[r]: 정렬되지 않은 상태이므로 mid와 r이 서로 다른 그룹에 있음. 최소값(그룹2의 첫번째 값)은 mid+1부터 r 사이에 있음. l=mid+1로 갱신해 mid와 그 앞의 값들을 탐색의 범위에서 제외한다.

```python
class Solution:
    # 시간복잡도: O(logn)
    # 공간복잡도: O(1)
    def findMin(self, nums: List[int]) -> int:
        l, r = 0, len(nums) - 1
        while l<r:
            mid = (l+r)//2
            if nums[mid] < nums[r]:
                r = mid
            else:
                l = mid + 1
        return nums[l]
```

이 문제에서는 min값을 찾으라고 했기 때문에 두번째 그룹의 첫번째 값을 찾아가고 있다.
만약 max값을 찾으라고 했다면 첫번째 그룹의 마지막 값을 찾으려고 했을 것이다.

* Case 1. nums[mid] > nums[l]: 둘이 정상적으로 정렬되어 있으므로 l부터 mid까지가 같은 그룹2에 있음. 최대값(그룹1의 마지막 값)은 mid이거나 mid보다 뒤에 존재. l=mid로 갱신하여 mid보다 앞의 값들을 탐색 범위에서 제외한다.

* Case 2. nums[mid] < nums[l]: 정렬되지 않은 상태이므로 mid와 l이 서로 다른 그룹에 있음. 최대값(그룹1의 마지막 값)은 mid-1부터 l 사이에 있음. r=mid-1로 갱신해 mid와 그 뒤의 값들을 탐색의 범위에서 제외한다.

#### [33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/description/)

위 문제를 이해하고 나니 이거는 자연스럽게 풀렸다. 포인트는 '정렬되어있는 쪽'을 기준으로 비교해서 기준을 찾으라는 것.

```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        l, r = 0, len(nums) - 1
        while  l <= r:
            mid = (l + r) //2
            if nums[mid] == target:
                return mid
            
            # mid까지는 그룹1 -> 그룹 1로 가면 nums[l]과 nums[mid]사이의 값이 타겟이 됨.
            if nums[l] <= nums[mid]: 
                if target >= nums[l] and target < nums[mid]:
                    # target이 그 사이에 있다면 앞쪽으로 보냄.
                    r = mid - 1
                else: # nums[l]과 nums[mid] 사이에 없다면 뒤쪽 그룹으로 보냄.
                    l = mid + 1
                
            else: # mid이후로 정렬된경우.
                if nums[mid] < target and target <= nums[r]:
                    # target이 그 사이에 있다면 뒤쪽으로 보냄.
                    l = mid + 1
                else:
                    r = mid - 1
        return -1
```

#### [981. Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/description/)

key-value가 포함되어서 헷갈릴 수 있지만, 결론적으로 get을 할 때 저장된 값들 중에서 이진 탐색을 수행해야 하는 문제다.
여기서 주의해야 할 점은, 정확한 timestamp가 존재하지 않을 경우, 주어진 timestamp보다 작거나 같은 값 중에서 가장 큰 값을 반환해야 한다는 것이다.

Find Minimum in Rotated Sorted Array 문제와의 차이를 보이는 것이, 최소값은 상대적인 값이며, 무조건 배열 안에 존재하므로 탐색이 종료될 때 남는 값이 결과가 된다. 따라서 while l < r을 사용하여 l == r 상태에서 루프가 종료되도록 만들고, nums[l]을 반환한다. (이때 l == r 이므로, nums[l] 과 nums[r] 은 동일한 값을 가리킨다.)

반면, 이 문제에서는 원하는 timestamp가 탐색 범위 내에 없을 수도 있다. 이진 탐색이 끝난 후 남은 l과 r의 상태를 분석하여, timestamp보다 작거나 같은 값 중에서 가장 큰 값을 선택해야 한다. 아래와 같은 예시를 보자.

nums = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
target = 55

이 상태로 target을 찾아서 탐색한다고 생각하면 

1. l = 0, r = 9 → mid = (0 + 9) // 2 = 4 (nums[mid] = 50)
   - nums[mid] < target → l = mid + 1 = 5

2. l = 5, r = 9 → mid = (5 + 9) // 2 = 7 (nums[mid] = 80)
   - nums[mid] > target → r = mid - 1 = 6

3. l = 5, r = 6 → mid = (5 + 6) // 2 = 5 (nums[mid] = 60)
   - nums[mid] > target → r = mid - 1 = 4

결국 `l = 5, r = 4` 상태로 루프를 빠져나오게 된다. 이 때 l은 55보다 큰 숫자의 최초 index, r은 55보다 작은 숫자의 마지막 인덱스이다.

* l을 반환해야 하는 경우
  * 목표: target 이상의 값 중에서 가장 작은 값을 찾아야 한다.
  * 탐색 종료 후 l은 항상 target 이상의 첫 번째 값을 가리킨다.

* r을 반환해야 하는 경우
  * 목표: target 이하의 값 중에서 가장 큰 값을 찾아야 한다.
  * 탐색 종료 후 r은 항상 target 이하의 가장 큰 값을 가리킨다.

이 문제에서는 target이 없으면 target보다 작은 값중 가장 큰 값을 돌려줘야 한다, 즉 탐색 종료 후 r값을 리턴해야 한다.

```python
class TimeMap(object):

    def __init__(self):
        self.key_to_values = {}
    def set(self, key, value, timestamp):
        """
        :type key: str
        :type value: str
        :type timestamp: int
        :rtype: None
        """
        if key in self.key_to_values:
            self.key_to_values[key].append((value, timestamp))
        else:
            self.key_to_values[key] = [(value, timestamp)]

    def get(self, key, timestamp):
        """
        :type key: str
        :type timestamp: int
        :rtype: str
        """
        # value that recorded timestamp is equal or less than `timestamp`

        if key not in self.key_to_values:
            return ""
        
        records = self.key_to_values[key]

        # 첫 값보다 timestamp가 작다면 굳이 탐색하지 말고 early return.
        if records[0][1] > timestamp:
            return ""

        l, r = 0, len(records) - 1
        while l <= r:
            mid = (l+r)//2
            v, t = records[mid]
            if t == timestamp:
                return v
            if t < timestamp:
                l = mid + 1
            else:
                r = mid - 1
        # 끝까지 탐색했는데 정확히 일치하는 값이 없었다면 
        return records[r][0]
```

이 l,r의 관계를 생각하면 파이썬의 bisect의 동작을 자연스럽게 떠올릴 수 있다. l을 반환하는 경우는 bisect_left와 같고, r을 반환하는 경우는 bisect_right - 1과 같다.

```python
import bisect
nums = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
target = 55

pos_left = bisect.bisect_left(nums, target)   # 5
pos_right = bisect.bisect_right(nums, target) # 5
```

bisect_left(nums, target)의 동작
* target이 배열에 있을 경우, 처음 등장하는 위치를 반환 (즉, target 이상 중 가장 작은 위치).
* target이 배열에 없을 경우, target이 들어갈 수 있는 가장 왼쪽 인덱스를 반환.

bisect_right(nums, target)의 동작
* target이 배열에 있을 경우, 마지막 등장한 위치의 오른쪽 인덱스를 반환.
* target이 배열에 없을 경우, target보다 큰 값이 처음 등장하는 위치를 반환.
* 즉, bisect_right - 1을 하면 target 이하 중에서 가장 큰 값의 위치를 찾을 수 있다.
