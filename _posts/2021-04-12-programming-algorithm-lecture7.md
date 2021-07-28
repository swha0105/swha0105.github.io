---
layout: post
title:  "[Intro Algorithms] Linear-Time Sorting"
subtitle:   "Introduction to Algorithms"
categories: programming
tags: algorithms
comments: true
---

# 7. Linear-Time Sorting
- Comparision sort
- Count sort
- Radix sort

<br/>

---

# Comparison model of Computation
`model of computation` (Lec 2) 다양한 Sort 연산을 비교할 수 있다.  
이때, `model of computation`은 다음과 같은 가정을 가진다.
- 모든 인풋은 black box. (Abstract Data Types)
- 비교 연산만 가능.
- time cost = number of comparison 

<br/>

---

# Comparision sort (Decision Tree model)
두개 요소값을 반복적으로 비교하여 정렬하는 sort 방법.   
`Comparision sort`는 모든 결과, 비교, 정답을 구성하는 트리로 시각화 가능하다.

|![BF](https://swha0105.github.io/assets/intro_algorithm/image/lec_7_comparision_sort.png)   
|:--:| 
| comparsion sort 예제 |

**Array A**에서 A[1] 값을 정렬한다고 하자, `Comparision sort`는 두개의 요소들을 반복적으로 비교하기 때문에 binary tree로 위와 그림과 같이 표현이 가능하다.  

<!-- |  | Ridge | Lasso |
|---|:---:|---:|
| 변수(feature)선택 유무 | X | O |
| Analytic solution | O | X |
| 좋은 예측성능 | 변수 간 상관관계 많을 때  | 변수 간 상관관계 적을 때 | -->


| **Decision tree model** | **algorithm** |
|---|---:|
| internal node |  binary decision(comparisons) |
| leaf  |  answer |
| root-leaf path  |  algorithm execution |
| length of path  |  running time |
| height of tree  |  worst case running time |

`Decision Tree model`과 알고리즘의 비교

### Time complexity
데이터의 갯수가 n, 트리의 높이가 h라 가정하면 말단노드의 최대갯수는 n!이 된다.  
이때 Comparision sort의 시간복잡도는 다음과 같다.  

number of leaves >= number of answer 을 만족해야 되기때문에,  
$$2^{h} \geq n! $$ 을 만족한다. 따라서, $$h \geq log(n!)$$

$$h \geq log(n!) \; = \; \log(1 2 3 ... n) = \log(1) + \log(2) + \log(3) + \log(n)$$  
$$= \sum_{i=1}^{n} \log(i)
\; \geq \; \sum_{i=\frac{n}{2}}^{n} \log(i)
\; \geq \; \sum_{i=\frac{n}{2}}^{n} \log(\frac{n}{2}) $$   
$$ = \frac{n}{2}\log(n) - \frac{n}{2} $$   
$$= O(n \log(n))$$

시간복잡도를 증명하는 다른 방법으로는 `Stirling's Formula`을 이용하여 증명도 가능하다.

Stirling's Formula: $$ n! \approx \sqrt{2 \pi n} (\frac{n}{e})^{n} $$ (when n is very large)

$$ h \geq log(n!) \; \approx \; \log(\sqrt(2 \pi n)(\frac{n}{e})^{n})$$  
$$ = O(n \log(n))$$

<br/>

---

# Count sort
Array A의 모든 요소의 빈도를 세어 sorting 하는 방법.

Array A = [1,2,3,4, ... k-1] 일때,
1. k만큼 갯수를 가지는 Array B를 생성 (len(B) = k)
2. Array B에 해당되는 데이터와 빈도수를 저장한다. L[key(A[j])].append(A[j]) for j in range(n):
3. 2를 다 수행한다음, B를 데이터에 따라 빈도수 만큼 출력하면 sort된 형태가 된다. 

### Time complexity
위의 순서중 1번은 O(k), 2번은 O(n)의 시간복잡도를 가진다.
`Big-O notation`에 의하면 시간복잡도는 **O(n)**이다.    
  
하지만, 이때 k값은 array의 맥스값으로 특정한 경우에 비효율적인 알고리즘이 구성된다.  

예를 들어, A = [1,2,1,2,10000] 있다고 가정할 때, 마지막 요소인 10000을 위해 L의 크기는 10000이 되어야하고, 시간복잡도가 증가하게 된다.  
  
이러한 단점을 극복하기 위해 `Radix sort`가 나오게 된다.

<br>

---

# Radix sort
모든 숫자 (데이터) 들을, 지수법으로 표현할 수 있다고 가정한다.  
이때, 가장 작은 자리수 부터 가장 큰 자리수 까지 sorting하여 정렬한다. 

|![BF](https://swha0105.github.io/assets/intro_algorithm/image/lec_7_radix_sort.png)   
|:--:| 
| comparsion sort 예제 |

A = [329,457,657,839,436,720,355] 이 있을때, 가장 마지막 자리수인 0의 자리수 부터 100의 자리수 까지 정렬한다.  
이를 수학적으로 표현하자면

$$ d = \log_{b}k$$
> b: 자릿수 (=100)  
> k: 데이터 

이때, b가 100이 되고, d값의 마지막 자리 수는 [9,7,7,9,6,0,5]가 되고 이를 따라 먼저 sorting한다.  
이후, d값의 다음 값은 [2,5,3,5,5,2,3]이 되고 이미 먼저 sorting된 값을 기준으로 다시 sorting한다.
...

이러한 알고리즘을 통해 정렬을 한다.

### time complexity 

O(n $$log_{n}k$$) $$ \quad k leq n^{c}$$  
= $$O(n)$$

~~유도 과정 생략~~

<br/>

Count sort와 Radix sort은 선형시간에 가깝게 정렬할 수 있기 때문에 **`linear-time sort`** 라고 한다

<br/>
<br/>

### 사담
강의하시는 에릭다이어는 위키백과에도 나올만큼 어렸을적 부터 신동으로 유명했다고 한다.  
그래서 그런지 완전 천재식 설명이라 알아듣기가 힘들다 ㅜㅜ 흐름잡기 힘들어 [이 블로그](https://ratsgo.github.io/data%20structure&algorithm/2017/10/16/countingsort/)를 포함한 다른 블로그들을 많이 참조하는중.

---
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
