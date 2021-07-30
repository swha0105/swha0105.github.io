---
layout: post
title:  "[Intro Algorithms] Models of Computation, Document Distance"
subtitle:   "Introduction to Algorithms"
categories: programming
tags: algorithms
comments: False
---

# Lecture 2. Models of Computation, Document Distance

- Model of computation
- Document distance

<br/>

---

# Model of computation

- Input에서 Output을 계산하는 일련의 과정과 그 cost를 시뮬레이션하는 **수학 모델**

### Role

- 알고리즘에서 사용되는 연산정의
- 연산에 대한 Cost (Time, Space ...)정의
- Cost of algorithm = sum of operation osts

### **Computer ↔ Mathematics**


![Computer & Mathematics](https://swha0105.github.io/assets/intro_algorithm/image/lec2_1.png)  


## Type of Model of computation

1. **Random Access Machine (RAM, ≠ Random Access Memory)**
    - Object들은 정렬된 메모리 Array에 위치해있다.
    - Memory에 접근하고 저장하는 시간과 Simple operation (+,=,*,/) 은 1 time step을 소비한다.
    - Instruction은 한번에 하나만 실행 될 수 있다.

2. **Pointer Machine**
    - Object들은 Dynamically allocated 되어있다 (linked list, directed graph)
    - 연산을 할 수 없고 input을 읽을수만 있다.
    - Object들을 Pointer (Reference) 로 다룬다

### **Python model**

- Python은 RAM과 Pointer Machine 개념이 모두 공존한다.
- Object들의 type과 연산에 따라 유용한 개념을 따라 사용된다.

1. List: (tuple.string)
    - 기본적으로 RAM의 개념을 따른다.

        a. L[i] = L[j] + 5 ⇒ ( O(1) )

        b. L = L1 + L2 (  O(L1) + O(L2) )

    - Pointer Machine의 개념을 따르는 연산도 있다.

        a. L.attribute()   ( O(1) ) 

위의 개념들과 Dict, Heapq, Long에 대해서는 짧게 있다는것만 소개하고 넘어감. 강의 후반부에 다시 배운다고 언급됨.

<br/>

---


# Document Distance problem

- 문서들간의 거리를 정의하고 계산하는 문제.
- 문서들간 공통된 word를 count하고 이를 이용해 거리를 정의한다.

### Idea

![Computer & Mathematics](https://swha0105.github.io/assets/intro_algorithm/image/lec2_2.png)  


D1 = "The cat" , D2 = "The dog" 일때 가장 기본적인 Idea는 다음과 같다.   
**모든 Word에 대한 공간을 Span하고 공간상 두 벡터의 각도를 계산한다.**

$$d(D1,D2) = \arccos(\frac{D_{1} \cdot D_{2}}{|D1||D2|})$$



## Solving methods

강의에서는 위와 같은 Idea를 구현하는 여러 방법을 소개하였다.  
각 방법에 대해 1 메가 바이트 정도되는 문서 두개 비교할때 걸리는 시간을 측정하였다.   


### Method 1. 위에서 언급된 방법. 문서를 두 벡터로 표현하고 각도를 계산 
[Code link](https://swha0105.github.io/assets/intro_algorithm/material/docdist1.py)   228.1(s)
> 1. split the text lines into words
> 2. count frequency of each word
> 3. sort words into alphabetic order
> 4. Inner product & Get angle

### Method 1.1 List extend 사용
 
- Method 1 코드의 **문서에 존재하는 word들을 list로 리턴하는 부분** 을 다음과 같이 바꾸었다. 

> word_list = word_list + words_in_line (변경전)  
> word_list.extend(words_in_line)  (변경후)  

 [Code link](https://swha0105.github.io/assets/intro_algorithm/material/docdist2.py)   164.7(s)

<details>    
<summary> list의 append와 extend </summary>
<div markdown="1">   

![Computer & Mathematics](https://swha0105.github.io/assets/intro_algorithm/image/lec2_3.png)  
append는 x 그 자체를 원소로 넣고 extend는 iterable의 각 항목들을 넣음

[출처](https://m.blog.naver.com/wideeyed/221541104629)

</div>
</details>

<br/>

### Method 2. Method 1을 Dictionary로 구현. 

알고리즘은 Method 1과 동일하다.  
- Dictionary를 구성하여 단어와 빈도수를 `key`와 `value`로 구성한다.  

[code link](https://swha0105.github.io/assets/intro_algorithm/material/docdist4.py) 71.7(s)

hash 개념인거 같은데 왜 빠른지는 좀 더 알아봐야한다..

### Method 2.1 String 내장 함수

- Method 2의 대문자를 소문자로 바꾸는 함수에서 string class의 method인 **maketrans** **translate**를 사용하였다.
- 위와같은 method를 사용하면 string을 쉽게 치환을 할 수 있다.

[code link](https://swha0105.github.io/assets/intro_algorithm/material/docdist5.py) 18.3(s)

### Method 2.2 Merge sort

- Method 2.1에 대해 insert sort대신 **merge sort**를 사용하였다. (다음 강의 주제)

[code link](https://swha0105.github.io/assets/intro_algorithm/material/docdist6.py) 11.5(s)

### Method 2.3 Treat whole file as a single "line"

- 지금까지는 line by line으로 word들을 분석하였지만 이 방법은 문서를 통째로 한 line으로 인식한다.

[code link](https://swha0105.github.io/assets/intro_algorithm/material/docdist8.py) 0.2(s)

<br/>
<br/>

**Method 1** 과 **Method 2.3**은 약 1000배 차이가 난다.  
이렇듯 같은 알고리즘을 구현하는 데도 있어 효율을 생각하며 코딩을 해야한다..

#### ref 

[1. https://medium.com/@rabin_gaire/models-of-computation-document-distance-7be4a9850067](https://medium.com/@rabin_gaire/models-of-computation-document-distance-7be4a9850067)



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
