
- heap (node, child, leaf) 설명된거 사진
- max heap 사진넣기
- time complexity 증명

#
- Heap, Priority Queue.
- Heap Time-complexity
- Heap sort

# Heap & Priority Queue

#### Priority Queue:
- Implement a set S of elements, each of elements associated with key

#### Prioirty operations
insert(S,x): insert element x into set S  
max(s): return element of S with largest key  
extract_max(s): retrun element of S with larest key and remove it from s  
increase_key(S,x,k): increase the value of element x's key to new value k


### Heap Feature
- Heap은 Priority queue (우선순위 큐)를 구현해놓은것이다.
- Heap은 nearly complete binary tree이다.

#### Heap structure and type

트리 구조  (left, right,  적은 그림)

Root of Tree (first element): i = 1  
parent(i) = i/2  
left(i) = 2i  
right(i) = 2i+1  

> i는 index, left(i)는 index i의 key(leaf)

max heap:
- max heap은 parent(i)가 left(i)와 right(i)보다 크다.
- parent node는 child node보다 항상 크다
min heap: min heap은 parent(i)가 left(i)와 right(i)보다 크다.

- max-heap 사진

#### Max-heap pseudocode

```python

def Build_max_heap(A):
    for i in range(n/2,1,-1):
        heap = Max_heapify(A,i)


def Max_heapify(A,i):
    l = left(i)
    r = right(i)

    if(l = heap-size(A) and A[l] > A[i]):
        largest = l
    else:
        largest = i 

    if(r = heap-size(A) and A[r] > A[largest]):
        largest = r

    if largest != i:
        A[i],A[largest] = A[largest],A[i]
        Max_heapify(A,largest)

```

- Build_max_heap: Unordered array에서 max heap을 만드는 연산    
- Max_heapify: max heap을 규칙을 어기는 하나의 node를 고치는 연산.  

#### Heap Time-complexity

트리 구조  (node, level, height 다 적은 그림)

- Heapify for single node: O(log(n))
- Heapify all element by single process: O(n log(n))
 

하지만 level 1부터 heapify하면 O(n)
이 방식은 array의 모든 값들을 한번에 heapify할때. 

- 수학증명

- Heapify all element: O(n)

<br/>

# Heap sort
- Heap은 정렬에 강점을 가지는 데이터 구조이다. (Binary search tree은 탐색에 강점)


### heap sorting algorithm
1. unordered array에서 Max heap (A9를 만든다. O(n)
2. A에서 가장 큰 element (node)를 찾는다 (A[1]). O(1)
3. A[1]과 A[n]을 swap 한다. O(1)
4. 최대값인 A[n]을 제외하고 새로만든 sorted array에 넣는다. O(1)
5. A[n]이 제외된 A[1:n-1]를 가지고 max heap을 다시 만든다. 이떄 A[1]에 대해서만 연산 해주면 된다. O(log(n))
6. 2번으로 돌아간다.

위와같은 과정을 모든 node (element)에 대해 수행해야되기 때문에 전체 시간복잡도는 O(n log(n))이 된다.

### pseudo code
``` python
def heap_sort(unsorted):
    n = len(unsorted)
    # BUILD-MAX-HEAP (A) : 위의 1단계
    # 인덱스 : (n을 2로 나눈 몫-1)~0
    # 최초 힙 구성시 배열의 중간부터 시작하면 
    # 이진트리 성질에 의해 모든 요소값을 
    # 서로 한번씩 비교할 수 있게 됨 : O(n)
    for i in range(n // 2 - 1, -1, -1):
        heapify(unsorted, i, n)
    # Recurrent (B) : 2~4단계
    # 한번 힙이 구성되면 개별 노드는
    # 최악의 경우에도 트리의 높이(logn)
    # 만큼의 자리 이동을 하게 됨
    # 이런 노드들이 n개 있으므로 : O(nlogn)
    for i in range(n - 1, 0, -1):
        unsorted[0], unsorted[i] = unsorted[i], unsorted[0]
        heapify(unsorted, 0, i)
    return unsorted
```

[코드 출처](https://ratsgo.github.io/data%20structure&algorithm/2017/09/27/heapsort/)




