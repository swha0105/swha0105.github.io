

- Kind of sorting 
- Recursion tree expansion



# why sorting


Sorting problem

Input: array A[1:n]
Output: permutation of B[1:n] of A (B[1] <= B[2] <= B[n])

using for Binary search, data compression, computer graphics... 


## kind of soring algorithm

1. insertion sort

### Algorithm
1. key (A[j]) 를 이미 정렬된 sub-array (A[1:j-1])에 **pairwise swap** 으로 위치를 찾아 insert.
2. 1번 과정을 j = (2,len(A))에 대해 반복한다.

### pseudo code

```python

for j in range(2,len(A)):
    key = A[j]
    i = j-1

    while i>0 and A[i] > key:
        A[i+1] = A[i]
        i = i-1
    A[i+1] = key

```

### Time complexity:
O(n^2), In place algorithm

- comparison are substantially more expensive than swaps.

<br/>

2. binary insertion sort

### Algorithm
1. key (A[j]) 를 이미 정렬된 sub-array (A[1:j-1])에 **binary search** 으로 위치를 찾아 insert.
2. 1번 과정을 j = (2,len(A))에 대해 반복한다.

### Features
- 알고리즘은 pairwise swap에서 binary search로 바뀐것을 제외하고 insertion sort와 동일하다.
- Key를 insert할때 이미 정렬된 sub-array에서 key가 들어갈 곳을 만들기 위해 값들을 이동시킨다. 이때, elementwise하게 이동시킬 경우와 block단위로 이동시킬 경우 time complexity가 다르다.


### Time complexity: 
elementwise move: O(n^2)  
block move: O(n logn)  
In place algorithm

<br/>

3. merge sort

### Algorithm
A[1:n]에 대해,
1. n = 1이면 done. (base condition)
2. 1번 조건이 아니면 A[1:n/2], A[n/2+1:n]으로 recursive하게 나눈후 sort한다.
3. 나누어진 sub-array들을 합친다.

### Example code (recursive)

``` python

def merge_sort(list):
    if len(list) <= 1:
        return list
    mid = len(list) // 2
    leftList = list[:mid]
    rightList = list[mid:]
    leftList = merge_sort(leftList)
    rightList = merge_sort(rightList)
    return merge(leftList, rightList)

def merge(left, right):
    result = []
    while len(left) > 0 or len(right) > 0:
        if len(left) > 0 and len(right) > 0:
            if left[0] <= right[0]:
                result.append(left[0])
                left = left[1:]
            else:
                result.append(right[0])
                right = right[1:]
        elif len(left) > 0:
            result.append(left[0])
            left = left[1:]
        elif len(right) > 0:
            result.append(right[0])
            right = right[1:]
    return result

```
[출처](https://ratsgo.github.io/data%20structure&algorithm/2017/10/03/mergesort/)

### Features
- divide & conquer approach
- in-place algorithm을 array의 반을 keep하는 방법으로 구현가능 하지만 성능이 안좋아 잘 사용되지 않는다.
- 따라서, **insertion sort보다 공간복잡도 면에서 불리함이 존재한다.**

### Time complexity
- Time: O(n logn), Space: O(n) 
- Time: O(n logn), Space: O(1) (in-place implement, but not good) 


# recursion tree expansion

- Recursion algorithm의 time complexity를 증명하는 방법 중 하나.

Merge sort를 예로 들자면 다음과 같다.  

$$ T(n) = c1 + 2 * T(n/2) + cn (c > 0 )$$

> T(n): time complexity of work done for n items
> c1: constant time in order to divide array
> c: merge part

여기서 c1은 무시하도록 한다. 

![recursion tree](https://swha0105.github.io/assets/intro_algorithm/image/lec3_1.png)  

n의 크기를 가지는 array A를 merge sort할때의 time complexity를 계산하는 그림이다.  
각 단계(세로) 별로 leaf의 개수는 n으로 동일하다. 그리고 n이 더이상 나누어질때 까지 연산하는 횟수는  log(n)이 된다.   
따라서, n번의 연산을 log(n)만큼 하기 때문에 시간복잡도는 O(n log(n))이 된다.