---
layout: post
title:  "[Intro CS] Tuples, Lists, Aliasing, Mutability, Cloning"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: cs
comments: true
---
# 5. Tuples, Lists, Aliasing, Mutability, Cloning
- Tuple
- List

<br/>

***

# Tuple

### Features

- Sequence of  anything   (String is Sequence of Character)
- Cannot change element values, **immutable** 
⇒ 튜플에 element를 추가하는건 가능, 삭제하거나 수정 불가능
- 튜플에 element가 단단 하나 있을 때 element뒤에 `,` 가 붙음  **(length 1인 string과 구분하기 위해)**  
 a = (), a += (1),  print(a) ⇒ (1,)  


### Operator

- 기본적으로 **String operator**와 매우 비슷한 연산형태
- `+` : Concatenation of tuple   
- `*` : Repeat tuple
- `-` , `/` 은 안됨.

### Usage

- **Swap variable values**

    ```python
    # not recommanded
    temp = x
    x = y 
    y = temp

    **# Good**
    **(x,y) = (y,x)**

    ```

- **Return more than one value from a function**

    ```python
    def function_example(x,y):
    	q = x//y
    	r = x%y
    	**return (q,r)**

    (q,r) = function_example(4,5)
    ```

- **Pairs with different type variables**

    서로 다른 타입의 변수들이 묶여있을때 각각 다른 연산을 처리 하기 좋음.

    ```python
    data= ((2014,"Harry"),(2014,"Harry"),(2012,"jake"),(2010,"Taylor"),(2008,"Joe") )

    def tuple_handle(aTuple):
    	nums = ()
    	words = ()
    	for t in aTuple:
    	    
    	    nums = nums + (t[0],)  # tuple의 요소가 하나 있을때는 , 필수!
    	    if t[1] not in words: # t0과 t1에 대한 다른연산 시도
    	        words = words + (t[1],)
    	
    	    min_n = min(nums)
    	    max_n = max(nums)
    	    unique_words = len(words)

    	return (min_n,max_n,unique_words )   # 함수에서 리턴값이 하나 이상일땐 튜플로!

    (min_year,max_year,name) = tuple_handle(data)
    ```

<br/>

***

# List

### Features

- Sequence of information, accessible by index
- Element can be changed, Mutable

### Operator

1. 새로운 리스트를 만들어내는 연산 (기존의 리스트에 영향 없음)

    - `+` : Concatenation of list

    - `*` : Repeat list

    - `sorted(array)`: sorting array

2. 기존의 리스트를 변형하는 연산  (mutable operation)

    - `.` means Call methods or functions of list object (class)

    - `.extend`: Concatenation of lists

    - `.append`: add element to list

    - `array.sort()`: sorting array

    - `del array[index]`: remove element by index **`Time complexity: O(N)`**

    - `array.remove(value)`: remove element by value **`Time complexity: O(N)`** 

    - `array.pop()`: remove last element **`Time complexity: O(1)`**   

<br/>

<details>    
<summary> TMI: list와 시간복잡도. </summary>

코딩을 하다보면 list의 요소들을 제거해줘야되는 연산을 많이 하게 될 것이다. list는 기본적으로 stack의 구조를 (LIFO) 따라 pop을 하게 될 경우 가장 마지막 요소가 빠지게 되고 시간 복잡도는 O(1)이게 된다. 하지만 가장 처음에 들어온 요소를 제거 하기 위한 pop(0), pop(1)과 같은 연산들은 O(N)의 시간복잡도를 가지고 이럴 경우 굉장히 비효율적인 연산이 된다.

이러한 연산이 필요한 경우 list가 아닌 파이썬의 built-in 모듈 collections의 deque를 사용하여 popleft, appendleft와 같은 연산을 활용하자. 
</details>

<br/>

### Aliases  
- 리스트의 변수 이름은 **Object**를 가르키는 **Pointer** 이다. 따라서 **Mutable operation**을 할때,

    1.  Object를 가르키는 **비슷한 변수 이름이 많거나**
    2.  **Iteration**연산을 통한 mutable operation을 하면

예상과 다르게 문제가 발생할 확률이 **매우 높다**. 특히 함수 안에서 List를 Mutable operation할때 실수할 확률이 높다. ~~경험담~~

```python
#exmaple 1.

warm = ['red','yellow','orange']
hot = warm  # hot과 warm은 변수이름만 다를뿐, 가르키는 object는 같다!! 
hot.append('pink')

if hot == warm:
	print("aliases") # it will be printed out

#####

#example 2.
L1 = [1,2,3,4]
L2 = [1,2,5,6]

for e in L1: #iteraion
	if e in L2:
		L1.remove(e) #mutable operation 

print(L1) = [2,3,4]  #only element 1 was deleted
```

**1번 예제:**

원래의 object는 같고 변수이름만 다를때, mutable 연산을 하면 원래의 object가 바뀌게 되고 그 object에 해당하는 모든 변수이름들이 가지는 값들이 변하게 된다.

**2번 예제:**

파이썬의 for loop 에는 내장되어있는 counter가 존재하고 counter가 현재 iteration되고 있는 list 길이에 도달하게 되면 loop는 꺼진다. 

내장된 counter는 L1이 변하지 않았따면 1부터 시작하여 4까지 연산해야 하지만 루프를 돌면서 L1 리스트의 길이가 mutable 연산을 통해 바뀌었고 바뀐 리스트의 길이 만큼 counter가 도달하게 되면 loop는 끝나게 된다

### Solution to avoid aliases

1. : Cloning 
    - L1_copy = L1[:] 을 하게 되면 L1에 있는 모든 element에 접근하여 새로운 객체 (L1_copy)를 만들게 된다.
2. : copy.deepcopy
    - import copy후 , L1_copy = copy.deepcopy(L1)을 하게 되면 새로운 객체를 형성하게 된다.

이 두개의 차이점은 아직 알아보지 못하였다. 시간날때 알아보도록...