---
layout: post
title:  "[Intro CS] Guess-and-Check, Approximation, Bisection"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: cs
comments: true
---

<!-- <script type="text/javascript" 
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML">
</script> -->

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

# 3. Guess-and-Check, Approximation, Bisection
# Contents

- ~~string manipulation~~ (2단원에 같이 정리함)
- guess and check algorithms (exhaustive enumeration)
- approximate solutions
- bisection method

***

# Exhaustive enumeration (Guess and Check)

- Example code 3.1:
 세제곱근 (cube root) 찾기

    ```python
    #reference to fig 3.3 in text book
    x = int(input('Enter an integer: '))  #remember return of input type is str
    ans = 0
    while ans**3 < abs(x): #remember we don't know number of iteration => use while
    	ans = ans + 1
    if ans**3 != abs(x):
    	print(x , 'is not a perfect cube')
    else:
    	if x < 0:
    		ans = -ans
    	print('Cube root of', x,'is', ans) #remember comma makes space! not concat 
    ```

- 답이 될 수 있는 집합내의 모든 원소 검색.   
⇒ 위의 예제에서는 0 ≤  ans < abs(x) 에 속하는 모든 정수
- 다음과 같은 성질을 만족하는 `Decrementing function` 를 구성해야한다.
    1. 일련의 프로그램 변수를 정수로 변환해야한다.
    2. Loop가 시작될때 function의 값은 음수이면 안된다. ( ≥ 0)
    3. Loop가 끝날때 function의 값은 양수이면 안된다.  (≤ 0)
    4. 매번의 iteration마다 function의 값은 감소해야한다. 

     ⇒ 위의 예제에서는 **abs(x) - ans^3**

- Exhaustive enumeration은 N에 선형적으로 수렴한다.

<br/>

***

# Approximation solutions

Q: 위의 exhaustive enumeration으로 $$\sqrt{2}$$ 를 구해보자.

당연히 답이 안나올것이다. exhaustive enumeration의 제한 조건 중 하나가 ans는 답이 될 수 있는 집합내의 모든 원소인데 $$\sqrt{2}$$ 는 무리수 이기 때문에 정수집합에 속하지 않기 때문이다. 

무리수는 정확한 값을 알 수 없다. 사람이야 정확한 값을 몰라도 표기법으로 $$\sqrt{2}$$ 라고 쓰지만 ~~멍청한~~ 컴퓨터한테 이런식으로 말해주면 알아듣지 못한다 ( computers only know what you tell them )

따라서 필연적으로 근사를 해야 할텐데 이번 강의는 이에 대한 기본적인 알고리즘들을 배운다




<br/>

Example code 3.2: 세제곱근 (cube root) 찾기

```python
#reference to fig 4.5 text book
x = 25
epsilon = 0.01
step = epsilon**2
numGuesses = 0
ans = 0.0

while abs(ans**2 - x) >= epsilon and ans < x:
    ans += step
    numGuesses += 1 
print('numGuesses =', numGuesses)

if abs(ans**2 - x) >= epsilon:
    print('Failed on square root of', x)
else:
    print(ans, 'is close to square root of', x)
```

위의 예제코드는 exhaustive enumeration을 approximation 방법으로 풀어낸 것이다. 
코드 3.1과는 다르게 무리수도 찾아낼 수 있음을 보인다. 하지만 계산속도에서 좋은 성능을 보이지 못하는데 예를 들어,  
처음 추론이 실제값과 먼 위치에 존재하거나, 매우 정확한 값을 요구해 epsilon이 작고, step이 작으면 iteration 횟수는 기하급수적으로 증가 할 것이다. 따라서 조금 더 현명한 방법을 고려할 필요가 있다. 


<details>
<summary>TMI: Epsilon-delta 증명 </summary>
    

자연대/공대생이라면 학부 1학년때 배우는 미적분학에서 Epsilon-delta증명을 배웠을것이다. 그 증명의 개념을 활용해 이 예제에서 epsilon을 극한을 이용해 무한소로 보내면 ans가 5에 일치 하다는걸 알 수 있다. 이 Approximation solution도 똑같은 개념이지만 컴퓨터에서 극한을 이용해 무한소로 보내면 무한루프가 돌것이기 때문에 어느정도 Tolerance 범위 안에서 찾게 한다.
</details>

<br/>

<details>
<summary>TMI: 현대 응용수학과 근사법 </summary>

대학원생때 ~~신물나게~~ 했던 프로젝트 중 유한차분법 (FDM, Finite difference Method)의 한 방법인 WENO(Weighted Essentially Non-Oscillatory)를 사용해서 충격파, 플라즈마 불안정성과 같은 비선형적인 물리현상을 묘사하는 일이 있었다.  
이런 WENO를 포함한 현대 계산수학과 계산물리에 최전선에 사용되고 있는 응용수학 방법들도 모두 근사방법을 따른다. 해석학적인 해가 없는 비선형적 모델을 푸는 최적의 방법이 수많은 iteration을 통한 근사방법이다.

</details>

<br/>
<br/>

***

# Bisection Method

~~술 자리에서 하는 업엔다운 게임이다~~

솔직히 취소선을 쳤지만 이거보다 더 잘 설명 할수가 없다.. 

![Bisection search](https://swha0105.github.io/assets/intro_cs/image/lec_3_Untitled.png)

<br/>


0부터 100까지 숫자가 있다고 하자. 그리고 정답은 67이라고 가정하자. 

처음 추론할때 가장 합리적인 선택은 정답이 될 수 있는 집합에서 가운데 있는 50이다. 

그 뒤, 정답이 추론값 보다 높다고 알려주면 그 다음 합리적인 선택은 75이다. (50 ~ 100에서 선택) 
.. 이런식으로 반복하다 보면 정답을 찾을 수 있다. 

이렇게 정답이 될 수 있는 **집합 (search space)**를 반토막 내며 추론 하는 방법이 `Bisection Method` 혹은 `Binary search Method` 이다.

- **Bisection search는  $$\log_{2}{N}$$ 에 수렴한다.**

따라서 0과 100에서 특정한 숫자를 찾는 알고리즘은 $$\log_{2}{101} \approx 6.658$$ 즉, 7번안에 무조건 찾아진다는걸 알 수 있다.   

- Example code 3.3:
Bisection search

    ```python
    #Ana bell's slide
    cube = 27
    epsilon = 0.01
    num_guesses = 0
    low = 0
    high = cube
    guess = (high + low)/2.0
    while abs(guess**3 - cube) >= epsilon:
    	if guess**3 < cube :
    		low = guess
    	else:
    		high = guess
    		guess = (high + low)/2.0
    		num_guesses += 1
    print 'num_guesses =', num_guesses
    print guess, 'is close to the cube root of', cube
    ```

위의 예제는 추론하려는 숫자가 1보다 작거나 음수일때 작동하지 않는다. 그 이유를 교수님이 찾아보고 코드를 짜보라고 간단한 숙제를 내주셨기에 1보다 작을때 경우 코드를 직접 짜보았다. 

- Example code 3.4:
Bisection search  x < 1

    ```python
    cube = 0.5
    epsilon = 0.01
    num_guesses = 0
    low = 0
    high = 1
    guess = (high + low)/2.0

    while abs(guess**3 - cube) >= epsilon:
        if guess**(3) > cube:
            high = guess
        else:
            low = guess

        guess = (high + low)/2.0
        num_guesses += 1
        print(guess,high,low,guess**(3)-cube)
    ```

<br/>

***

# About Floats

이 챕터는 교수님이 강의를 안하셨지만 교과서 reading material에 나와있고 읽어보니 유용해서 정리한다.

코드를 직접 짜본 사람이라면 이러한 경우를 경험해보았을 것이다. 

```python
#ref text book
x = 0.0
for i in range(10):
	x = x + 0.1
if x == 1.0:
	print(x, '= 1.0')
else:
	print(x, 'is not 1.0')

>>> 0.9999999999999999 is not 1.0 
```

띠용 이게 무슨소리인가 **0.1를 10번 더한게 1.0과 같지 않다니**. 코드를 처음 접하는 사람들은 이런 결과를 보면 "컴터가 나랑 장난친다"  라고 생각 할 것이다. ~~내가 그랬거든~~

사실 지금까지도 원인을 완벽히 이해를 못했는데 이번 챕터를 읽고 이해를 했다. ~~드디어~~

왜 이런 결과가 나오는지 컴퓨터 입장도 들어보자. 
다들 알다시피, 컴퓨터놓는 Binary number밖에 모른다. 즉 0,1로 모든 숫자를 표현하고 이러한 0,1 를 bits라고 부른다. 또한, 숫자를 표현하는 방법 중 가장 많이쓰이는 방법은 유효숫자 (significant number)과 거듭제곱(exponent)의 개념을 이용해서 표현하는 것이고 컴퓨터도 당연히 이 방법을 따른다. 이러한 표기법을 이용해 1.949를 표기하면 (1949,-3) 이 된다. (1.949 = 1949*10^-3)

이를 이용해서 0.1을 10번 더하는 예제를 이진법을 통해 알아보자. 
0.1을 4개의 유효숫자를 가진 2진법으로 표현하면 (0011,-101)이다. 이를 풀어 쓰자면 아래와 같이 된다
  $(1*2^{1} + 1*2^{0}) * (2^-(1*2^{2} + 1*2^{0}) = \frac{3}{32} = 0.09375$   

아니 너무 당연한 0.1이 4자리 유효숫자를 가진 2진법으로 표현하면 0.09375가 되버리네??
이를 10번 더하면 0.9375가 되지 1.0이 되지 않는다. 0.0625라는 생각보다 큰 오차가 발생한다.

물론 많은 계산에서 4개의 유효숫자를 사용하지는 않는다. 그래도 유효숫자의 개수를 무한대로 높이지 않는이상 정확히 1이 될순 없다. 

따라서 이렇게 **Float끼리 비교할때는 항상 유념하고** round 함수를 쓰자. ~~샷건 치지말고~~


<br/>

***

## Reference
[Lecture pdf](https://github.com/swha0105/swha0105.github.io/blob/gh-pages/assets/intro_cs/material/Lec2.pdf) 