---
layout: post
title:  "[Intro CS] Recursion and Dictionaries"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: cs
comments: true
---

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

# 6. Recursion and Dictionaries
- **Recursion**
- Dictionaries

<br/>

***

# Recursion

### Definition

> - **Algorithmically:** 문제를 작은 문제로 나누는 divide-conquer 으로 접근 하는 방법
> - **Semantically**: 어떤 함수에서 본인을 (함수) 콜하는 프로그래밍 기술.

### Structure
> **Recursion** 코드는 **Recursive step**과 **base case**로 나누어져 있다. 
> - **Recursive step:** 작은(간단한) 버전의 같은 문제에서도 적용 되는 알고리즘이 기술된 코드 부분
> - **Base case**: 어떤 케이스에 대한 답이 기술된 코드 부분. (재귀 함수의 탈출 부분 기술)

### **Scope & flow of control**
> - Recursive call은 함수 자신만의 own scope를 만든다.
> - Recursive call이 끝에 다달아서 값을 return 해주면 역방향으로 만들어진 scope의 역방향으로 전달해줌

<br/>

### Recursion Example & Scope

```python
# Example of recursion 

def fact(n):
	if n==1:   # base case 
		return 1
	else:  #recursive step
		return n*fact(n-1)
```

<br/>

**In Scope view** 
![In Scope view](https://swha0105.github.io/assets/intro_cs/image/lec_6_Untitled_3.png)
왼쪽에서 오른쪽으로 함수를 call 할것이고 이에따라 순서대로 scope(stack)가 열린다. 아직 값을 return받지 못했기에 함수가 실행되지는 않는다.

<br/>

**In Value view** 
![In Scope view](https://swha0105.github.io/assets/intro_cs/image/lec_6_Untitled_4.png)


맨 오른쪽 함수에서 값을 return 받았고, 그 값을 왼쪽으로 전달해준다. 그리고 맨 오른쪽 scope(stack)은 메모리에서 사라진다. 


<br/>

***

# Hanoi's Tower

~~이 내용으로 이틀 공부한거 실화냐~~

이 예제는 recursion에서 가장 유명한 예시이고 생각보다 짧게 해주고 넘어갔는데 MIT 학생들은 이정도 듣고 다 이해하나 보다.. 나는 그러지 못해 이 [블로그를](https://shoark7.github.io/programming/algorithm/tower-of-hanoi) 참고해서 공부 한뒤에 내용을 정리한다. 여기 있는 그림은 블로그에서 가져온것이다. 

<br/>

## Description

![Hanoi's](https://swha0105.github.io/assets/intro_cs/image/lec_6_Untitled_5.png)


원판 타워를 아래 2가지 조건을 만족시키며 A에서 C로 옮기는 것이다 (B도 가능)

1. 한번에 가장 위에 있는 하나의 원판만 움직일 수 있다.
2. 어떤 원반 위에 그보다 더 큰 원반을 쌓을 수 없다.

<br/>

## Recursive Thinking

처음 문제를 보았을때 가장 아래에 있는 원판을 움직이는게 목표가 된다는걸 알 수 있다. 따라서 

`A에 있는 5번 원판을 C로 옮기고 싶다` 라는 생각을 할 수 있다. 그럼 그 뒤에

→ `A에 있는 4번 원판을 C로` → `A에 있는 3번 원판을 C로`

와 같이 같은 행동을 반복적으로 사용해야 하는걸 알 수 있다.  이를 통해 이 문제는 **recursive** 문제임을 알 수 있다.

![Hanoi's sequence](https://swha0105.github.io/assets/intro_cs/image/lec_6_Untitled_6.png)
Ref: [https://shoark7.github.io/programming/algorithm/tower-of-hanoi](https://shoark7.github.io/programming/algorithm/tower-of-hanoi)


<br/>

## Concrete thinking

편의를 위해 `1, A->B->C` 는 A에 있는 1번 원판을 B를 통해 C로 옮겻다고 표현한다.

3번 원판을 옮기기 위해 2번을 움직여야 하고 2번을 옮기기 위해 1번을 움직여야 한다. 따라서 Recursive Thinking의 끝은 `1, A->B->C` 임을 알 수 있다. 이 표현은 바로 실행 될 수 있다.

그렇다면 `2, A→C→B`를 하기 위해 위의 그림에서 start에서 3번 그림까지 과정을 살펴보자. 여기서 B,C의 순서가 다른건 넘어가도록 하자.

`1, A->B->C` → **`2, A->C->B`** → `1, C->A->B`

여기서 잠시, **`2, A->C->B`** 앞 뒤로 있는 저것들은 1번 원판을 옮기기 위해 **사용된 표현과 비슷하다는걸 알 수 있다.**

이 사실을 알고 한번만 더 해보자. `3, A→B→C` 을 위의 전체 그림을 참고해서 그대로 표현하면,

`1, A->B->C` → `2, A->C->B` → `1, C->A->B`  → **`3, A->B->C`**→ `1,B->C->A` → `2,B->A->C` →`1, A->B->C`  가 된다.

여기서 **`3, A->B->C`** 가 나오기 전 앞선 일련의 표현들은 **`2, A->C->B`** 을 수행하기 위한 표현과 완벽히 일치하고, 그 후에 나오는 일련의 표현들은 출발,경유,도착 순서만 바뀐걸 알 수 있다

자 이제 함수로 정의 해보자.

move(1,a,b,c)는 `1, A→B→C` 라고 하자

move(2,a,c,b)는 `1, A->B->C` → **`2, A->C->B`** → `1, C->A->B` 의 과정을 수행하게 되고 이것은
- `move(1,a,b,c) -> move(2,a,c,b) -> move(1,c,a,b)` 로 표현할 수 있다.  

move(3,a,b,c)는 `1, A->B->C` → `2, A->C->B` → `1, C->A->B`  → **`3, A->B->C`**→ `1,B->C->A` → `2,B->A->C` →`1, A->B->C` 의 과정을 수행하고 이것은 
- `move(2,a,c,b) → move(3,a,b,c) → move(2,b,a,c)` 로 표현 할 수 있다.

떙땡 드디어 끝났다. 이제 규칙이 보이지 않는가?

move(N,a,b,c)를 하기 위해 move(N-1,a,c,b)를 앞에서, move(N-1,b,a,c)를 뒤에서 수행 하면 된다. 그리고 N=1일때 까지 이 재귀함수를 계속 유지하면 끝이다. 

이제 코드를 보자.

```python
def move(n,start,via,des):
    if n == 1:   #base case
        print(n," ",start,'->', via, '->', des)  
    else: #recursive step
        move(n-1,start,des,via)   #N번 원판을 옮기기 전 
        print(n," ",start,'->', via, '->', des) #N번 원판을 옮김
        move(n-1,via,start,des) #N번 원판을 옮긴 후
```

코드는 ~~허무할 정도로~~ 굉장히 짧다. 이 예제를 통해 배우는건 recursion의 간단명료함 인듯하다. 물론 잘못 쓰면 무한루프가 돌거나 의도하지 않았던대로 작동할 수 있지만 Prof. Eric Grimson 에 따르면 익숙해지면 recursion이 for-loop이나 while보다 훨씬 보기 편하다고 말씀하셨다.

솔직히 나도 이 문제를 처음접하면 while로 접근 했을듯하다. 그리고 아직 recursion에 대해 완벽히 이해하지 못한듯 하다. 앞으로 문제풀때 recursion에 대해 한번 더 고찰 해봐야겠다. 

<br/>

***

# Dictionaries

### Features
> - key - value가  pair로 있는 집합.
> - **순서가 없다.  unordered**

### Key
> - must be unique
> - Immutable type (int, string, tuple, bool..)  
> (모든 Immutable type은 hashable하다!)

### Value
> - Key와 연동되는 값
> - immutable, mutable 과 관계없이 아무 변수타입이나 가능.

### Unordered example

```python
grades = {'Ana':'B', 'John':'A+', 'Denise':'A', 'Katy':'A'}

grades.keys() # returns ['Denise','Katy','John','Ana']
grades.values() à returns ['A', 'A', 'A+', 'B']
```

**key와  value의 연동은 풀리지 않지만 iteration 되는 순서는 guarantee되지 않는다!**

<br/>

### Dynamic programming & Recursion & Dictionary

먼저 피보나치 수열을 구하는 Recursion 예시를 보자.

```python
def fib(n):
	if n == 1:
		return 1
	elif n == 2:
		return 2
	else:
		return fib(n-1) + fib(n-2)
```

이 예시는 피보나치 수열을 구하는 코드이고 recursion하게 잘 작동한다.
하지만 이러한 recursion 코드는 총 $$2^{n-2}$$ (n>2) 의 계산 횟수가 필요로 하다.  

그런데 만약 fib(3),fib(4) ... fib(9)를 컴퓨터에서 기억하고 있다면 계산 횟수를 획기적으로 줄일 수 있을것이다. 이렇게 계산된 값을 기억하고 (memoization) 다른 계산예 사용하는 테크닉을 **Dynamic programming** 이라 한다

자 그럼 dictionary가 어떻게 dynamic programming 에 사용될 수 있을지 다음 예시를 통해 알아보자.

```python
def fib_efficient(n, d):
	if n in d:
		return d[n]
	else:
		ans = fib_efficient(n-1, d) + fib_efficient(n-2, d)
	d[n] = ans
	return ans
d = {1:1, 2:2}
```

똑같이 피보나치 수열을 계산하는 예제이지만 fib_efficient 변수에 key값으로 n값, value값으로 수열에 해당하는 값을 지정했다. 이럴 경우 recursion에서 해당 n값을 필요로 할때 recursion을 통해 base case까지 갈 필요 없이 return이 됨으로 굉장히 효과적으로 계산이 가능하다.

~~이번 렉쳐는 하노이의 탑에서 에너지를 다써서 더이상 정리 못하겠다~~

<br/>

*** 

## Reference

[Lecture pdf](https://github.com/swha0105/swha0105.github.io/blob/gh-pages/assets/intro_cs/material/Lec6.pdf) 

<br/>

### 코스 절반쯤에서 쓰는 후기


이번 강의를 들음으로서 이 코스의 반을 들었다.

들으면서 느끼는건데 여기 교수님들은 강의에 굉장히 진심인거 같다... ~~한국과는 다르다~~

준비도 굉장히 많이 한게 느껴지고 유치할수도 있는 비유를 하면서 학생들을 어떻게든 이해시키려는 모습이 조금 감동이였다..  이번 강의부터 ~~목소리가 예쁜~~ Dr. Ana 대신 Prof. Eric Grimson이 가르치는데 말이 너무 빨라 첨에는 알아듣기 힘들었지만 듣다보니 수업을 체계적으로 잘한다는 느낌을 받았다.  그리고 이 강의를 끝까지 듣고 다음 advanced 코스까지 들으면 나의 코드에도 **근본**이 생길꺼같다는 희망을 느끼고있다.


![개그](https://swha0105.github.io/assets/intro_cs/image/lec_6_humor.png)

