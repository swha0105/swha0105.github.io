---
layout: post
title:  "[Intro CS] Searching and Sorting Algorithms"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: cs
comments: False
---
# 12 Searching and Sorting Algorithms

- Searching Algorithms
- Sorting Algorithms
- 강의 마무리

<br/>

***

# Searching Algorithms

- Collections of items (Implicit, Explicit) 에서 특정한 item이나 Group of item을 찾는 방법

### Linear Search (Brute force, Exhaustive enumeration)

- 순차적으로 모든 item들을 검색하는 방법.
- Time Complexity: O(n)

### Bisection Search (Binary search)

- 알고리즘 내용 3장에서 정리
- **Sorting 된 collection of items에서만 사용가능**
- 일종의 Divide and Conquer 알고리즘
- Time Complexity: O(log n)
- example

    ```python
    def bisect_search(L,e):  # Wrapper function으로 argument를 간단하게 만들어줌
        def bisect_search_helper(L,e,low,high):
            if high == low:
                return L[low] == e
            mid = (low + high)//2
            if L[mid] == e:
                return True
            elif L[mid] > e:
                if low == mid:
                    return False
                else:
                    return bisect_search(L,E,low,mid-1)
            else:
                    return bisect_search(L,E,mid+1,high)

        if len(L) == 0:
            return False
        else:
            return bisect_search_helper(L,e,0,len(L)-1)
            
            
    ```

Bisect search가 강력한건 알겠지만 사용하러면 sorted 되어있어야 한다는 조건이 있다.

일반적으로 sorting하여 Bisect search를 하는 O(sort) + O(log n) 과정은 Linear search 하는 O(n) 보다 크다고 알려져있다. 

이렇게만 본다면 당연히 Linear search를 하는게 맞겠지만, 해당 collection (list)에서 k번 search를 해야 하는경우  o(sort) + k * O(log n) < k * O(n) 이 됨으로 k 가 커질수록 sorting한 다음 Bisect search하는게 더욱 효율적이다. 그럼 sorting algorithm들을 알아보자


<br/>

# Sort Algorithms

1. Monkey sort :
    - Bogo sort, Stupid sort, Slow sort, Permutation sort, Shotgun sort ~~진짜 샷건 칠듯~~
    - Randomly 모든 item들을 pick 하여 순서가 맞는지 확인하는 방법
    - Time complexity: **Undefined**
2. Bubble sort
    - Item들을 연속적인 pairs 로 구성 한다음, 비교하여 item들을 swap함
    - 모든 item들을 비교했으면 다시 처음부터 item pairs를 구성하여 비교하고 swap없을 경우 중단
    - Time complexity: O(n^2)
    - Example code

        ```python
        def bubble_sort(L):
            swap = False
            while not swap:
                swap = True
                for j in range(1, len(L)):
                    if L[j-1] > L[j]:
                        swap = False
                        temp = L[j]
                        L[j] = L[j-1]
                        L[j-1] = temp
        ```

3. Selection sort
    - 전체를 검색해 가장 작은(큰) item pick
    - 그 다음 다시 한번 전체를 검색해 2번째 작은(큰) item pick
    - 전체 item들이 정리될때 까지 반복
    - Time complexity: O(n^2)
    - Example code

        ```python
        def selection_sort(L):
            suffixSt = 0 
            while suffixSt != len(L):
                for i in range(suffixSt, len(L)):
                    if L[i] < L[suffixSt]:
                        L[suffixSt], L[i] = L[i], L[suffixSt]
                suffixSt += 1
        ```

4. **Merge sort**
    - List (Item of collections) 을 sublists들로 split 한다 (until len(sublist) <2)
    - 길이가 0또는 1인 sublists들로 부터 sort하고 sublists들끼리 merge한다
    - **Time complexity: O(n*log n)**
    - Example code

        ```python
        def merge(left,right):
            result = []
            i,j = 0,0 #이런 표현 기억하기!
            while i < len(left) and j < len(right): #이런 조건문도 기억하기
                if left[i] < right[j]:
                    result.append(left[i])
                    i+=1
                else:
                    result.append(right[j])
                    j+=1

            while i < len(left):
                result.append(left[i])
                i += 1
            while j < len(right):
                result.append(right[j])
                j += 1	
            return result

        def merge_sort(L):
            if len(L) < 2: #base case
                return L[:]
            else:
                middle = len(L)//2
                left = merge_sort(L[:middle])
                right = merge_sort(L[middle:])
                return merge(left, right)
        ```

<br/>

***

# 강의 마무리

- **Key topics**
    - Represent knowledge with data structures (5,6)
    - Iteration and recursion as computational metaphors (2,6)
    - Abstraction of procedures and data types (4)
    - Organize and modularize systems using object classes and methods (8,9)
    - Difference classes of algorithms, searching and sorting (12)
    - Complexity of algorithms (10,11)

- **Think computationally**  (3 A's)
    - Abstraction

        - Choosing the right abstractions (계층구조와 함수)

    - Automation

        - Think in terms of mechanizing our abstractions 

    - Algorithms

        - Language for describing automated processes

강의를 마무리하며..

한달전 쯤 코드를 연습할때 벽이 느껴지고 더 이상 발전하지 않는다는 느낌이 들었다. 고민을 좀 해봤는데 아무래도 근본 그러니깐 코스웤의 부재로 CS 필드에서 배우는 기초개념이나 논리들을 못따라간다는 느낌을 받았다. 

그리고 이 강의를 듣고 나서 역시 어느 분야든 기초가 중요하다고 다시한번 느껴진다. 

비록 이 강의는 2학년 과목이지만 내가 몰랐던 부분 혹은 알고 있었지만 부족했던 부분을 시원하게 긁어주었다.  아무리 분야가 다르더라도 코드 개발경력 5년이 넘었는데 2학년 과목을 듣는게  자존심도 많이 상했지만 자존심을 내세울때가 아니라 근본을 찾는게 먼저라고 생각했다.

정말 막무가내 개발을 했었구나를 느낀 동시에 어쩌면 앞으로 커리어의 전환점이 될 수도 있겠다는 느낌이 들정도로 유익했다. 다음강의는 MIT courseware의 Introduction to Algorithms이 될거 같다. 다음 강의도 배울게 많았으면 좋겠다

# **Think Computationally & Think Recursively**


<br/>

*** 

## Reference

[Lecture pdf](https://github.com/swha0105/swha0105.github.io/blob/gh-pages/assets/intro_cs/material/Lec11.pdf) 