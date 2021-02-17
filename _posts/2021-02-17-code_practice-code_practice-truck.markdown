---
layout: post
title:  "[프로그래머스] 다리를 지나는 트럭"
subtitle:  "Programmers"
categories: code_practice
tags: code_practice
comments: true
---

![Problem](https://swha0105.github.io/assets/code_practice/P_truck.JPG)

<br/>

# 문제 개요

모든 트럭이 다리를 지나가는 가능한 가장 빠른 시간을 찾아라.  
이때, 트럭은 한번에 하나씩 출발할 수 있으며 1초에 1만큼 움직인다.  
또한 다리에 올라가있는 트럭의 총 무게는 다리의 하중을 넘기면 안된다.

bridge_length: 다리 길이
weight: 다리가 견딜 수 있는 하중
truck_weight: 트럭의 무게

<br/>

# 문제 풀이

1. bridge 객체를 만든다.
2. bridge에 무게가 허락하면 truck_weights에서 하나 가져온다.
3. bridge에 올라간 truck은 올라간 시간 + bridge_length과 함께 저장되기 때문에 정해진 시간이 되면 truck은 빠지게된다.

<br/>

# 코드 

```python 
def solution(bridge_length, weight, truck_weights):
    import collections

    time = 1
    bridge = collections.deque()
    truck_weights = collections.deque(truck_weights)

    bridge.append((truck_weights.popleft(),time+bridge_length))


    while bridge:
        time += 1


        if bridge[0][1] <= time:
            bridge.popleft()

        if truck_weights and truck_weights[0] + sum(list(map(lambda x:x[0],bridge))) <= weight:
            bridge.append( (truck_weights.popleft(),time+bridge_length))

        if not bridge:
            break

    return time   
```

<br/>

## 옛날 코드

```python 

def solution(bridge_length, weight, truck_weights):

    end = len(truck_weights)
    answer = 0
    time = 0
    truck_passing_weight = []
    truck_passing_time = []
    truck_passed = []


    while len(truck_passed) != end:

        if len(truck_passing_time) == 0:
            pass

        elif time - truck_passing_time[0] >= bridge_length:
            truck_passed.append(truck_passing_weight.pop(0))
            truck_passing_time.pop(0)


        if len(truck_weights) == 0:
            pass

        elif sum(truck_passing_weight) + truck_weights[0] <= weight:
            truck = truck_weights.pop(0)
            truck_passing_weight.append(truck)
            truck_passing_time.append(time)

        time = time + 1 
    answer = time
    return answer
```


<br/>

# Note

1. 옛날에 비해 코드가 늘긴했다.. 
2. hashing 개념을 이용해 트럭이 지나간 시간 정보를 같이 묶어서 사용했다. 
3. 처음에는 같이 갈 수 있는 트럭의 수를 구하는식으로 접근하였으나, 이러한 접근은 다리위에 있는 트럭이 하나 빠졌을때 그 다음 트럭이 다리위에 올라갈 가능성을 배제한다.
