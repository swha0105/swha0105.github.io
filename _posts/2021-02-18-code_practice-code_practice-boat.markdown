---
layout: post
title:  "[프로그래머스] 구명보트"
subtitle:  "Programmers"
categories: dev
tags: code
comments: false
---

![Problem](https://swha0105.github.io/assets/code_practice/P_boat.JPG)

<br/>

# 문제 개요

구명보트의 하중을 넘기지 않는 선에서 모든 사람들을 태울수 있는 최소의 구명보트 수를 찾아라.


people: 사람들의 무게가 저장되어있는 리스트
limit: 구명보트의 하중 

return: 필요한 구명보트의 최소의 수
  
<br/>

# 문제 풀이

1. greedy로 분류되어있다. 가장 무거운사람에 대해 가장 가벼운사람이 같이 타는걸 고려하는게 최선 (local optima = global optima)

2. 만약 최대 2명밖에 못탄다는 가정이 없었으면 문제가 조금 더 재밌었을듯 하다. ~~그래도 쉬울꺼 같음~~


<br/>

# 코드 

```python 

def solution(people, limit):
    import collections
    
    people.sort(reverse=True)
    people = collections.deque(people)

    answer = 0

    while len(people) > 1:

        heavy = people.popleft()

        if heavy >= limit:
            answer += 1
        else:
            if heavy + people[-1] > limit:
                answer += 1
            else:
                people.pop()   
                answer += 1
    
    if people:
        answer += 1
    return answer
```

<br/>

# Note

1. 문제가 너무 쉬워서 혹시 sorting을 쓰지말라는건가 한참 고민을 했었다. 하지만 sorting 안쓰고는 무게 정보를 O(1)으로 알 수 없어 sorting을 쓸수밖에 없었다.
2. 특별한 개념이 들어간것도 아니고 다른 풀이도 필요없을듯하다.. 