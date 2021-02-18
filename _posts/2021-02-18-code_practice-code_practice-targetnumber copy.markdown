---
layout: post
title:  "[프로그래머스] 타겟넘버"
subtitle:  "Programmers"
categories: code_practice
tags: code_practice
comments: true
---

![Problem](https://swha0105.github.io/assets/code_practice/P_targetnumber.JPG)

<br/>

# 문제 개요

n개의 음의 아닌 정수를 더하거나 빼는 모든 경우의수를 고려해 특정한 수가 되는 경우의 수를 찾아라.


numbers: 음이 아닌 정수가 담겨 있는 리스트  
target: 찾아야되는 특정한 수  

return: target이 될 수 있는 모든 경우의 수
  
<br/>

# 문제 풀이

1. DFS/BFS 로 분류되어 있다. 굳이 따지자면 모든 경우의 수를 찾아야되기 때문에 DFS에 가깝다.
2. numbers에서 하나의 수를 뽑아 더하고 빼는 작업을 반복적으로 연산을 해야되는걸 알 수 있다. **Think recursively**


<br/>

# 코드 

```python 

def solution(numbers, target):
    
    import collections

    numbers = collections.deque(numbers)
    num_list = []

    def recursive_function(numbers,num_list):

        if not numbers:
            return num_list
        else:

            num = numbers.popleft()
            positive_sum = [a + num for a in num_list]
            negative_sum = [a - num for a in num_list]

            num_list = positive_sum + negative_sum

            return recursive_function(numbers,num_list)

    first_num = numbers.popleft() 
    num_list.extend([first_num,-first_num])
    answer_list = recursive_function(numbers,num_list)

    count = 0 
    ans = [count + 1 for a in answer_list if a == target]

    return len(ans)
    
```

<br/>

# Note

1. numbers에서 하나씩 숫자를 뽑아 더하고 빼는 작업을 재귀함수를 구성하여 연산하였다.
2. num_list에서 더할 때, 바로 target이 되는 숫자를 찾고 싶었으나 재귀함수 이해가 아직 완벽하지 않은 탓인지 구현하기가 어려웠다. 따라서 모든 경우의 수를 다 찾은 뒤 다시 target number를 찾는 비효율적인 코드를 구성할 수 밖에 없었다.

3. 다른 사람이 2번을 구현 해놓은 코드가 있어 같이 첨부한다. 시간날때 조금 더 보고 이해해야겠다. 

```python
def solution(numbers, target):
    if not numbers and target == 0 :
        return 1
    elif not numbers:
        return 0
    else:
        return solution(numbers[1:], target-numbers[0]) + solution(numbers[1:], target+numbers[0])
```