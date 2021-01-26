---
layout: post
title:  "[Intro Algorithms]  Algorithmic thinking, peak finding"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: algorithms
comments: true
---

# 1. Algorithmic thinking, peak finding
   - Course overview
   - 1D peak-finding
   - 2D peak-finding

<br/>

***

# Course overview

   - Efficient procedures for solving large scale problems
   - Scalability
   - Classic data structures  (binary search tree, hash) & classical algorithms
   - Real implementation in python

<br/>

***

# 1D peak-finding

### **Problems**:

1D numbers (array)에서 peak을 찾자.  (만약 있다면)

![1d_problem](https://swha0105.github.io/assets/intro_algorithm/image/lec1_Untitled.png)  

**Peak**: b is a peak if and only if b ≥ a and b ≥ c. (b가 a와 c보다 크거나 같으면 b는 peak이다.)
> **if and only if(iff)** : 필요충분조건

<br/>

### Solution 1: Straightforward Algorithm

**Steps:**

1. 왼쪽이나 오른쪽에서 시작하여 Edge를 제외한 모든 Point에서 양옆의 숫자와 비교한다.
2. 양옆의 숫자와 비교하여 해당 숫자가 가장 크면 Peak.

**Time complexity**: O(n)

<br/>

---

### Solution 2: Divide and Conquer Strategy

**Steps:**

1. n/2부터 비교를 시작한다. 

    **Recursive call**

    - If n/2 < (n/2-1) 이면 0~(n/2-1)에 존재하는 숫자만 본다.
    - else If n/2 > (n/2-1)  이면 (n/2+1) ~ n 에 존재하는 숫자만 본다.
    - else이면 n/2는 peak이다.

**Time complexity**: O(log n)

<br/>

# 2D peak-finding

### **Problems**:

2D numbers (array)에서 peak을 찾자.  (만약 있다면)

![2d_problem](https://swha0105.github.io/assets/intro_algorithm/image/lec1_Untitled_1.png)  

**Peak**: a is a 2D-peak iff a ≥ b, a ≥ d, a ≥ c, a ≥ e
   
<br/>

### Solution 1: Straightforward Algorithm

**Steps:**

1. 한쪽 끝에서부터 시작하여 사방의 총 4개의 숫자와 비교한다.
2. Peak의 조건이 만족하면 그 point를 peak이라 정의한다

**Time complexity**: O(m*n)

<br/>

### Solution 2: Straightforward Algorithm

**Steps:**

1. Column 중 middle point (j = m/2) 를 pick 한다.
2. j = m/2 에서 i에 대해 global maximum을 찾는다
3. Global maximum (i,j) 와 (i,j-1),(i,j+1) 을 비교한다.

    **Recursive call**

    - If j가 < (j-1) 이면 (:,0)~(:,j-1)에 존재하는 숫자만 본다.
    - else if j가 > (j+1) 이면 (:,j+1) ~ (:,n)에 존재하는 숫자만 본다.
    - else이면 break 한다.
4. 3번에서 나온 마지막 column에서 global maximum을 찾는다
5. 4번에서 찾는 global maximum이 **peak**

**Time complexity**: O(n * log m )

<br/>


## Reference
[Lecture pdf](https://swha0105.github.io/assets/intro_algorithm/material/lec.png)  

<br/>
<br/>

#### 사족  

첫 강의 들었는데 MIT 학생들은 질문하는걸 두려워 하지않는거 같다. 저번 강의도 그랬지만 특히 이번 강의는 질문한다고 눈치주는 사람도 없을 뿐더러 질문하면서 교수와 Interaction 하는 모습이 굉장히 인상깊었다..