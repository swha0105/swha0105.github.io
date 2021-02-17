---
layout: post
title:  "[프로그래머스] 프린터"
subtitle:  "Programmers"
categories: code_practice
tags: code_practice
comments: true
---

![Problem](https://swha0105.github.io/assets/code_practice/P_printer.JPG)

<br/>

# 문제 개요

프린터 대기순서에서 문서를 하나씩 확인하여 우선 순위가 가장 높으면 출력, 그렇지 않은면 맨 마지막으로 보낸다. 이때 location에 해당되는 특정 문서가 몇번째로 출력되는지 확인  

<br/>

# 문제 풀이

- 우선순위큐를 구현하는 문제같지만 insert가 없다는 것에서 착안하여 heap구조가 아닌 python의 deque 구조를 사용한다. 

- 문서의 우선순위 (priorities)와 문서의 순서(location)를 한번에 저장해야 하기 때문에 hashing개념이 필요하다. 우선순위과 순서는 pair를 이루고 바뀌지 않기때문에 튜플로 hashing하였다. 

- dictionary는 unordered 구조기 때문에 tuple로 hashing하는게 맞다. [참조](https://swha0105.github.io/_posts/2020-12-24-programming-cs-lecture6.markdown)


<br/>

# 코드 

```python 

def solution(priorities, location):
   
    import collections
    
    hashing = []
    hashing = [(p,l) for l,p in enumerate(priorities)]
        
    deque = collections.deque(hashing)
    ref = deque.popleft()
    count = 0
    
    while True:
        
        for remains in deque:
            if remains[0] > ref[0]:
                deque.append(ref)
                break
        else:
            count += 1
            if ref[1] == location:
                return count
            
        ref = deque.popleft()            
    
    return count


```

<br/>

# Note

1. 생각보다 파이썬의 deque는 강력하다. stack과 queue의 특징을 한번에 사용할 수 있다.
2. 이 문제의 key는 priorities와 location을 hashing하는 개념인듯 하다.

