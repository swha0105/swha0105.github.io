---
layout: post
title:  "[Intro Algorithms] Models of Computation, Document Distance"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: algorithms
comments: true
---

# Lecture 2. Models of Computation, Document Distance

- Model of computation
- Document distance

# Model of computation

- Input에서 Output을 계산하는 일련의 과정과 그 cost를 시뮬레이션하는 **수학 모델**

### Role

- 알고리즘에서 사용되는 연산정의
- 연산에 대한 Cost (Time, Space ...)정의
- Cost of algorithm = sum of operation osts

### **Computer ↔ Mathematics**

![Lecture%202%20Models%20of%20Computation,%20Document%20Distance%20bbe38b0719234f0f83b59497a1da83dc/Untitled.png](Lecture%202%20Models%20of%20Computation,%20Document%20Distance%20bbe38b0719234f0f83b59497a1da83dc/Untitled.png)

### Type of Model of computation

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

# Document Distance problem

- 문서들간의 거리를 정의하고 계산하는 문제.
- 문서들간 공통된 word를 count하고 이를 이용해 거리를 정의한다.

### Idea

![Lecture%202%20Models%20of%20Computation,%20Document%20Distance%20bbe38b0719234f0f83b59497a1da83dc/Untitled%201.png](Lecture%202%20Models%20of%20Computation,%20Document%20Distance%20bbe38b0719234f0f83b59497a1da83dc/Untitled%201.png)

D1 = "The cat" , D2 = "The dog"

**모든 Word에 대한 공간을 Span하고 공간상 두 벡터의 각도를 계산한다.**

d(D1,D2) = arccos(\frac{D_{1}D_{2}}{|D1||D2|}) 

이 Idea는 매우 naive하고 누구나 생각 할 수 있지만 모든 Word에 대해 공간을 Span하는게 굉장히 비현실적이다. 만약 서로 다른 word들이 100개가 있다면 100차원의 matrix를 구성해야 할 것이고 메모리는 물론이고 계산시간이 실용적이지 않을것이다.

### Algorithm

1. Split each document into words
2. count word frequencies (document vectors)
3. compute dot product 

### Solving methods

1 메가 바이트 정도되는 문서 두개 비교.

Method 1. 위에서 언급된 방법. 문서를 두 벡터로 표현하고 각도를 계산

1. split the text lines into words
2. count frequency of each word
3. sort words into alphabetic order
4. Inner product & Get angle

228.1

여기서 문서에 존재하는 word들을 list로 리턴하는 부분 

word_list = word_list + words_in_line ⇒ word_list.extend(words_in_line)

하면 164.7 초. 

1. Python의 re 모듈을 사용해 일치되는 모든 word를 찾은 뒤,  각각을 itertaion한다. 

    Time complexity: O(c^n)  **쓰지 말라고 강조  (228.1)**

    - Pseudo code

        ```python
        import re 
        re.findall (r“ w+”, doc)

        for char in doc:
        	if not alphanumeric
        		add previous word
        			(if any) to list
        	  start new word
        ```

    Lecture code link

     

2. Word들을 sorting하여 총 몇개가 일치하는지 계산.

    Time complexity: O( k * log k)  + O(k) , k is number of words  (164.7)

    - Pseudo code

        ```python
        sort word list

        for word in list:
        	if same as last word: 
        		increment counter
        	else: 
        		add last word and count to list 
        		reset counter to 0
        ```

    Lecture code link

3. doc1의 모든 word을 iteration할때 python의 in 기능을 활용하여 doc2에 일치하는 word가 있는지 확인한다.

    Time complexity: O(k1 * k2), k1 is number of words in doc1 (123.1)

    - Pseudo code

        ```python
        for word, count1 in doc1:
        	if word, count2 in doc2: 
        		total += count1 * count2
        ```

    Lecture code link

4. e  71.7
5. e 18.3
6. e 11.5
7. e 1.8
8. e 0.2

LSA 와 비슷.!! 

model of computation

- specifies
- what operation an algorithms is allowed

2 programming style 

2 model of computations

Random Access Machine (≠ Random Access memory)

Pointer Machine

 Random Access memory (giant array and constant access time whether point location is  )

len(L) is built-in function

ref 

[https://medium.com/@rabin_gaire/models-of-computation-document-distance-7be4a9850067](https://medium.com/@rabin_gaire/models-of-computation-document-distance-7be4a9850067)