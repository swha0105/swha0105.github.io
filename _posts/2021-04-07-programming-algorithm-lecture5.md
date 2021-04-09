---
layout: post
title:  "[Intro Algorithms] Scheduling, Binary Search Trees, tree traversal"
subtitle:   "Introduction to Algorithms"
categories: programming
tags: cs
comments: true
---

# 5. Scheduling, Binary Search Treesd
- Runway reservation system
- Binary Search Trees

<br/>

---

# Runway reservation system
- 기본적으로 scheidling 알고리즘, 공항에 활주로가 하나 있다고 가정하고 `미래에 착륙하는` 비행기에 대한 예약 스케쥴 시스템.

### Algorithm
1. set R (활주로에 착륙하는 비행기에 대해 landing time 집합)을 설정.
2. 특정 time `t` 에 착륙 요청이 들어온다. 이때, t 사이로 k 만큼 착륙 스케쥴이 없으면 허가, 아니면 에러. (K minute Check)
3. 착륙 후, set R에 있는 해당되는 착륙 시간은 삭제 되어야함

위 문제를 풀때, 가장 효율적으로 착륙요청을 넣고 착륙시간을 지우는 데이터 구조는?

**A. Unsorted list / array:** 
- K minute check: O(n) 
- Insert, Delete: O(1)

**B: Sorted array**
- K minute check: O(lon g) (Binary search 가능), 
- Insert, Delete: O(n) (because its sorted array, so needed shifting)

**C: Heap**
- K minute check: O(n) (모든 원소 체크 필요.)
- Insert, Delete: O(log n)

**D: Binary search Tree**
- 모든 operations in time O(h)  **(log n < h < n)**

<br/>

---

# Binary search Tree

- Binary search와 Linked list을 결합한 자료구조
- Binary search의 효율적인 탐색과 data insert/delete를 효율적으로 하기 위한 자료구조

Binary Search Tree (BST) 예시
![BST](https://swha0105.github.io/assets/intro_algorithm/image/lec5_bst.png)  
- node x에 대해 key(x), left(x), right(x), parent(x) 가 존재한다.
- 이때, parent(x), left(x), right(x) 는 실제 값이 아닌, Pointer를 의미 한다. (Heap 과 다른점)
- node x을 기준으로 왼쪽아래에 존재하는 subtree에 모든 key값 y에 대해 key(y) < key(x)는 항상 만족한다.

## Operation

**1. Insert:**

|![BST_insert](https://swha0105.github.io/assets/intro_algorithm/image/lec5_bst_insert.png)  
|:--:| 
| [출처](https://ratsgo.github.io/data%20structure&algorithm/2017/10/22/bst/) |

위 그림에서 새로운 key 값 4를 추가 한다고 가정하자. 7가 3사이에 4를 추가해도 BST 속성을 만족하지만 Subtree를 다시 정의해야함.   
따라서 BST insert연산은 항상 마지막 노드(잎새노드) 에서 이루어져야함.  

Time Complexity: O(h)  **(log n < h < n)**

<br/>


**2. Delete:** (강의에 안나옴)

|![BST_delete](https://swha0105.github.io/assets/intro_algorithm/image/lec5_bst_delete.png)  
|:--:| 
| [출처](https://ratsgo.github.io/data%20structure&algorithm/2017/10/22/bst/) |

위 그림에서 key값이 20인 노드를 삭제 한다고 가정해보자.  
이때, Tree 구조를 만족하러면 아래에 있는 특정 값이 삭제된 위치를 대체하여야한다. 어떤 값이 좋을까?  
  
먼저, Node(key = 20) 왼쪽에 있는 노드들 중에 가장 큰 값을 `predecessor`라고 정의한다.  
또, 오른쪽에 있는 노드들 중 가장 작은 값을 `successor` 라고 정의한다.  

이 두개의 값들 중 아무 값이나 선택하여 삭제된 노드의 위치를 대체하면 된다. 아래 그림은 `successor`인 Node(key =20)이 대체한 그림이다.

|![BST_delete 2](https://swha0105.github.io/assets/intro_algorithm/image/lec5_bst_delete_2.png)  
|:--:| 
| [출처](https://ratsgo.github.io/data%20structure&algorithm/2017/10/22/bst/) |  
  
Time Complexity: O(h)  **(log n < h < n)**  

<br/>

**3. find value and find_min:**

Heap과 다르게 index가 정의 되어있지 않다. 따라서 root node에서 부터 left(x) pointer를 따라 트리의 높이까지 따라 내려가야한다.  
Time Complexity: O(h)  **(log n < h < n)**  

<br/>

### BST 의 한계점
Balanced Tree가 아니기 때문에 트리의 높이인 h값이 n값과 같을 수 있다.  
이러한 한계점을 극복하기 위해 다음 렉쳐에서 `AVL Tree`를 배운다고 한다.


<br/>

---

# Tree traversal
이 내용은 강의에 포함되어있지 않지만 중요한 내용이라 이 [블로그](https://www.google.com/search?q=traversal+order&rlz=1C1SQJL_koKR840KR840&oq=traversal+order&aqs=chrome..69i57.140j0j7&sourceid=chrome&ie=UTF-8)를 참조 하여서 정리한다.

- Traversal(순회)는 트리의 모든 노드들을 visit하는 것을 의미.
- Preorder(전위), Inorder(중위), postorder(후위)가 있다. 


|![Tree_ordering](https://swha0105.github.io/assets/intro_algorithm/image/lec_5_order.png)  
|:--:| 
| Tree 예제 [출처](https://m.blog.naver.com/PostView.nhn?blogId=4717010&logNo=60209908735&proxyReferer=https:%2F%2Fwww.google.com%2F) |  

1. Preorder
   1. 노드 x가 NULL이 아니라면 **노드 x에 접근** (NULL일 경우 Base case, Return None)
   2. 노드 x의 왼쪽 subtree root note (left child)을 기준으로 다시 1번을 call
   3. 노드 x의 오른쪽 subtree root note (right) child)을 기준으로 다시 1번을 call

위와같이 recursive하게 호출이 되며 예제 그림에서 호출되는 순서는 다음과 같다.  
**A->B->D->H->I->E->C->F->G** 

2. Inorder 
   1. 노드 x가 NULL이 아니라면 (NULL일 경우 Base case, Return None)
   2. 노드 x의 왼쪽 subtree root note (left child)을 기준으로 다시 1번을 call
   3. **노드 x를 접근**
   4. 노드 x의 오른쪽 subtree root note (right) child)을 기준으로 다시 1번을 call

예제 그림에서 호출되는 순서는 다음과 같다.   
**H->D->I->B->E->A->F->C->G** 

3. Postorder 
   1. 노드 x가 NULL이 아니라면 (NULL일 경우 알고리즘 종료)
   2. 노드 x의 왼쪽 subtree root note (left child)을 기준으로 다시 1번을 call
   3. 노드 x의 오른쪽 subtree root note (right) child)을 기준으로 다시 1번을 call
   4. 노드 x를 방문


예제 그림에서 호출되는 순서는 다음과 같다.   
**H->I->D->E->B->F->G->C->A** 



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
