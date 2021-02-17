---
layout: post
title:  "[프로그래머스] 기능개발"
subtitle:  "Programmers"
categories: code_practice
tags: code_practice
comments: true
---

![Problem](https://swha0105.github.io/assets/code_practice/P_develop.JPG)

<br/>

# 문제 개요

Progresses에는 각 기능의 완성도 담겨있고 speeds에는 각 기능의 개발속도가 담겨있다.  
각 기능의 완성도가 100이상이 되고 앞에 완성이 안된 기능이 없을 시 완료가 되고 배포가 된다.    
이때, 각 배포때 마다 몇개의 기능이 배포되는지 return 해라

<br/>

# 문제 풀이

- Progress에 speeds를 더해 맨 앞에 element가 100이 되는대로 pop한다. 
- 이때, 뒤에 있는 element들이 100인것도 pop한다. 다만, 100이 끊기는 순간 종료한다.
- 의외로 날짜 정보는 필요없다.

<br/>

# 코드 

```python 
def solution(progresses, speeds):
    import collections
    
    answer = []
    progresses = collections.deque(progresses)
    speeds = collections.deque(speeds)

    while progresses:

        progresses = collections.deque(list(map(sum,zip(progresses,speeds))))
        count = 0


        while progresses and progresses[0] >= 100 :
        # while progresses[0] >= 100 and progresses :
            progresses.popleft()
            speeds.popleft()
            count += 1


        if count != 0:
            answer.append(count)        
    
    return answer

```

<br/>

# Note

1. list element sum은 map을 통해 해야한다. ~~귀찮~~  

2. while의 조건문 순서에 따라 결과가 달라진다.
    - 주석처리 되어있는 조건으로 돌리면 progress가 존재하는지 확인하기 전에 0번째 요소를 체크하기 때문에 IndexError가 난다

