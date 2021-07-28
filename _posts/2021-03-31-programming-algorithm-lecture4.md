---
layout: post
title:  "[Intro Algorithms] Heaps and heap sort"
subtitle:   "Introduction to Algorithms"
categories: programming
tags: algorithms
comments: true
---

# 4. Heaps and heap sort
- Heap, Priority Queue.
- Heap Operation & Time-complexity
- Heap sort

<br/>

---

# Heap & Priority Queue

### Priority Queue:
- 집합 S의 element들이 각각에 해당되는 key와 연관(associated)되어있는 집합.

### Prioirty operations
- insert(S,x): element x를 집합 S에 insert
- max(s): 집합 s에서 가장 큰 key를 return
- extract_max(s): 집합 s에서 가장 큰 key를 return하고 그 key를 지움.
- increase_key(S,x,k): 집합 s의 element x의 key값을 k로 바꾼다. 

<br/>

### Heap 
- Heap은 Priority queue (우선순위 큐)를 구현해놓은것이다.
- Heap은 nearly complete binary tree이다.

>Complete binary tree 와 Full binary tree 비교  [그림 출처](https://ict-nroo.tistory.com/55)
![heap sturcture](https://swha0105.github.io/assets/intro_algorithm/image/lec4_trees.PNG)


### Heap structure and type

![heap sturcture](https://swha0105.github.io/assets/intro_algorithm/image/lec4_heap.PNG)


> Root of Tree (first element): i = 1  
> parent(i) = i/2,  node i에 대한 parent index return  
> left(i) = 2i,  node i에 대한 left index return  
> right(i) = 2i+1,  node i에 대한 right index return  


- max heap은 parent(i)의 key값이  left(i) 값과 right(i) 값보다 크다.  

<br/>

---

# Heap Operation & Time-complexity


### Max_heapify:
- Max heap을 규칙을 어기는 하나의 node를 고치는 연산.  

|![max_heapify](https://swha0105.github.io/assets/intro_algorithm/image/lec4_maxheap.png)
|:--:| 
| [출처](https://ratsgo.github.io/data%20structure&algorithm/2017/09/27/heapsort/) |

1. 위의 그림에서 root node인 `4`가 heap의 성질을 만족하지 못함 (max heap 기준)
2. left node와 비교하여 swap. (right node가 더 작지만 알고리즘상 고려 x)
3. heap 의 성질이 만족될때까지 위 과정을 반복

- worst case일때는 트리의 높이 (h = log n) 만큼 비교연산을 해야됨, 
- Time complexity: O(log n)의 시간복잡도를 가짐. 

<br/>

### Heap insert

|![heap insert](https://swha0105.github.io/assets/intro_algorithm/image/lec4_insert.PNG)   
|:--:| 
| [출처](https://ratsgo.github.io/data%20structure&algorithm/2017/09/27/heapsort/) |

S = [16,5,11,3] 에서 `18` 이 추가 되는 상황 
1. 새로운 element가 추가 될때는 가장 끝에 추가 된다 (Max heap 기준)
2. 새로운 element `18`이 parent node인 5와 비교하여 swap
3. 2번이 만족되지 않을때 까지 반복.

- worst case일때는 트리의 높이 (h = log n) 만큼 비교연산을 해야됨, 
- Time complexity: O(log n)의 시간복잡도를 가짐. 


<br/>

### Heapify all element: O(n)
 
Max_heapify를 이용하여 모든 element들을 heapify를 하게되면 시간 복잡도는 **O(n log(n))**가 된다.

하지만 모든 element들을 한번에 heapify 할 때, **O(n)** 으로 만족하는 알고리즘이 있는데 다음과 같다.

- heap의 레벨을 h (= log(n)) 이라 했을때, h-1 부터 (아래 그림에서 빨간 노드)부터 비교하면된다.
- h-1노드의 개수는 $$\frac{n}{4}$$ 이고 h 레벨과 비교하면 되기때문에 한번만 반복한다.
- h-2노드의 개수는 $$\frac{n}{8}$$ 이고 h-1, h 레벨과 비교해야 되기때문에 2번 반복한다.
... 반복.

위와같은 연산을 수행하였을때 시간복잡도는 O(n)이 된다.

힙이 **완전이진트리**라고 가정하였을때 연산량은 다음과 같다. (비교하는 연산을 한번이라 가정한다.)
    

$$ 2 * 1 \frac{n}{2^2} + 2 * 2 \frac{n}{2^3} + 2 * 3 \frac{n}{2^4} ... $$ 
$$ = 2 * \frac{n}{2^2} ( 1 + 2 \frac{1}{2}  + 3\frac{1}{4} ...) $$   
$$ = 2 * 2 * \frac{n}{4}$$  ( n이 무한대 일때, 수렴 값)  
$$ = n $$   
  

~~Big-O notation에 따라 무시되는 텀이 있을줄 알았는데 딱 맞아 떨어진다~~

|![heap insert](https://swha0105.github.io/assets/intro_algorithm/image/lec4_buildheap.PNG)   
|:--:| 
| [출처](https://ratsgo.github.io/data%20structure&algorithm/2017/09/27/heapsort/) |


<br/>

---

# Heap sort
- Heap 구조를 정렬해본다. (Heap은 정렬에 강점, Binary search tree은 탐색에 강점)


### heap sorting algorithm
1. unordered array에서 Max heap (A9를 만든다. O(n)
2. A에서 가장 큰 element (node)를 찾는다 (A[1]). O(1)
3. A[1]과 A[n]을 swap 한다. O(1)
4. 최대값인 A[n]을 제외하고 새로만든 sorted array에 넣는다. O(1)
5. A[n]이 제외된 A[1:n-1]를 가지고 max heap을 다시 만든다. 이떄 A[1]에 대해서만 연산 해주면 된다. O(log(n))
6. 2번으로 돌아간다.

위와같은 과정을 모든 node (element)에 대해 수행해야되기 때문에 전체 **시간복잡도는 O(n log(n))** 이 된다.

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




--- 

#### ref 

1. https://ict-nroo.tistory.com/55
2. https://ratsgo.github.io/data%20structure&algorithm/2017/09/27/heapsort/



<script>
MathJax.Hub.Queue(["Typeset",MathJax.Hub]);
</script>


<script>
MathJax = {
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']]
  },
  svg: {
    fontCache: 'global'
  }
};
</script>
<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-svg.js">
</script>
