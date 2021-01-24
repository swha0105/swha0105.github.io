---
layout: post
title:  "[Intro CS] Understanding Program Efficiency"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: cs
comments: true
---

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

# 10 & 11. Understanding Program Efficiency
- Estimation of algorithm efficiency
- Big-O notation
- Common Pattern in Big-Oh notation

사족  
나는 대학원을 물리학으로 나왔지만 석사학위하는 동안 병렬화와 슈퍼컴퓨터에서의 Scaling 과 씨름 했었다. 특히 응용수학적 기법이 Computation cost가 너무 커 병렬화가 제대로 되지않으면 의미있는 데이터 뽑기가 너무 힘들었다. 따라서 코드를 돌릴때 Efficiency가 중요한건 깨닫고 있었지만 경험적으로, 구글링하여 알아냈지 체계적이게 알지 못했는데 이번 기회로 알아가게 되었다.

<br/>

***

# Estimation of algorithm efficiency

1. **Use timing program**

    Python의 Import 모듈과 같은 코드내의 시간 측정 모듈을 사용한다 
    (MPI에서는 MPI_WTIME())

    - Good: Algorithm 끼리 비교가능.
    - Bad  : Implementation에 따라 다를 수 있음
    - Bad  : Computer hardware에 따라 다를 수 있음 (Hardware dependency)
    - Bad  : Input 사이즈, 특히 작은 사이즈일때 시간관계가 명확하지 않음.

        ⇒ **Input사이즈 와 Running time의 대한 상관관계 표현 불가.**

2. **Counting operation**

    컴퓨터의 기본 연산들이 같은 시간 소요를 한다고 가정 한 뒤 연산의 횟수를 구함.

    - Good: Algorithm 끼리 비교가능.
    - Bad  : Implementation에 따라 다를 수 있음
    - Good: Computer hardware와 상관없음
    - Bad  : 기본 연산에 대한 정확한 정의 부족

        **⇒ Input 사이즈와 Running time에 대한 상관관계를 근사적으로 표현 가능**

        - example code

            ```python
            def mysum(x):
            	total = 0            # Operation = 1
            	for i in range(x+1): # Operation = x 
            		total += i         # Operation = 2x     
            	return total         # Operation = 1

            # Total number of operation 3x + 2  
            # Assume that Assign, Arithmetic Operation, Accessing to memory are having same time
            ```

    무작위 데이터에서 exhaustive search를 통해 특정값을 찾고 return하는 코드 처럼, 연산횟수가 랜덤성에 의해서도 결정되는 경우가 있다. 이 경우 가장 Worst case를 선택하여 코드의 효율성을 논한다.
    Best case: 모든 가능성 중에 가장 빠른 케이스
    Average case: (Best + Worst)/2 이며 Practical measurement이다
    **Worst case: 모든 가능성 중 가장 느린 케이스이며 아래에 설명할 Big-O notation에서 사용된다.**

<br/>

***

# Big-O notation

- O(n)과 같이 Input 데이터 사이즈 n과 Running time의 상관관계를 나타냄

### Orders of Growth

**Big-O notation은 이 Order of Growth를 표현하는 방법이다** 

Order of Growth는 다음의 성질을 만족한다.

- Input 사이즈가 매우 크고, Worst case일때를 가정한다.
- Input 사이즈와 Running time과의 상관관계, 즉 Order만 보는것이다.
- Order에 가장 영향을 크게 미치는 Factor만 고려할 것이다.

예를들어, Input 데이터의 크기가 n이라 하고 이것에 대해 exhaustive search를하여 모든 데이터들을 찾아본다고 하였을때, 이 코드의 **Counting operation**는 n + something이 될 것이다. 이때 **Order of Growth**은 n 이 될 것이고 **Big-O notation**으로 표현하면 **O(n)** 될 것이다. 그리고 이때 **O(n)은** 코드의 **Time complexity (시간 복잡도)** 가 된다. 

Example table

Time Complexity           |  Counting opreation   
:-------------------------:|:-------------------------:
$$O(n^2)$$   | $$n^2 + 2n + 2$$
$$O(n^2)$$   | $$n^2 + 10000n + 3^1000$$
$$O(n)$$     | $$log{(n)} + n + 4$$
$$O(n log{(n)})$$   | 0.0001 n log{(n)} + 300n
$$O(3^n)$$   | $$ 2 n^{30} + 3^n $$

<br/>


### Law of Addition & Multiplication

1. Law of addition:
- Sequential 하게 이어져있는 코드의 시간복잡도 계산방법. 가장 order가 큰 부분이 코드의 시간복잡도가 된다.  

    ```python
    for i in range(n):
        print('a')
    for i in range(n*n):
        print('b')
    ```

    위의 Loop 는 O(n)의 시간복잡도 이고 아래의 Loop 는 O(n^2)의 시간복잡도를 가진다. 이 경우 총 코드의 시간 복잡도는 **O(n) + O(n^2)** 이고 이것은 Big-O notation의 정의에 따라 **O(n^2)**이 된다. 

2. Law of Multiplication: 
- Nested하게 묶여있는 코드의 시간 복잡도 계산방법. 두개를 곱하여 산정한다.

    ```python
    for i in range(n):
        for j in range(n):
            print('a')
    ```

    O(n)의 Loop안에 O(n)이 또 있다. 이경우 O(n) * O(n)이 되고 전체 시간 복잡도는 O(n^2) 된다.

### Typical cases of Big - O notation

- O(1): Input 데이터와 관계없는 시간 복잡도
- O(log n): Input 데이터가 늘어날때 **logarithmic**하게 시간이 증가한다.
    > - Bisection Search  
    > - Binary Search of a list
- O(n): Input 데이터가 늘어날때 **Linear**하게 시간이 증가한다.
    >- Iterative Loops
    >- Recursive Calls
- O(n*log n): Input 데이터가 늘어날때 **Log-Linear**하게 시간이 증가한다.
    > - **Many practical algorithms**
- O(n^c): Input 데이터가 늘어날때 **Polynomial** (Typically, c = 2 **Quadratic**) 하게 시간이 증가한다. 
    > - Nested Loops
    > - Recursive Function (O(n)) Calls
- O(c^n): Input 데이터가 늘어날때 **Exponential** 하게 시간이 증가한다
    > - 2 more than Recursive Function calls  (Hanoi's Tower)