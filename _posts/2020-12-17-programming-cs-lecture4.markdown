---
layout: post
title:  "[Intro CS] Decomposition, Abstraction, Functions"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: cs
comments: true
---
# 4. Decomposition, Abstraction, Functions
- Function feature (decomposition, abstraction)
- Function component (Parameters)
- Parameter Passing
- Scope
- Lambda function

  (강의,인터넷,교과서 내용 취합하여 컨텐츠 목록 바꿈)

# Function features

내가 학부 1학년때 기초 프로그래밍 수업을 드랍한 이유는 프로그래밍을 왜 배우는지, 배워서 어디에 쓰는지 전혀 몰랐기 때문이다. ~~이게 직업이 될줄이야~~ 그 당시 수업내용으로 함수를 배웠는데 "아니 그냥 코딩하면되는데 왜 굳이??" 라는 생각을 했었다. 과거의 나처럼 되지않기 위해 함수를 왜쓰는지 확실하게 알고가자..

예를 들어, 자동차를 만든다고 치자. 설계도 던져주고 알아서 만들어라고 ~~대학원생~~ 할 수 있겠지만 좀 더 스마트한 방법이 있을것이다. 먼저, 기능이 다른 각 부품들을 따로 만들것이다. 엔진, 바퀴, 브레이크는 역할이 다르기 때문에 서로의 상세 기능들을 몰라도 각자 알아서 작동할 수 있다. 함수도 이러한 특성을 가지고 있는데 이러한 특성을 **Decomposition** 이라 한다. 따라서 엔진, 바퀴, 브레이크를 만드는 하청업체(함수)를 만들고 계약을 할 것이다.  

부품이 다 모아져서 조립을 해야 한다고 치자. 이 조립하여 자동차를 만드는걸 **메인프로그램을 만든다고** 표현해도 될 것이다. 그런데 조립하는 입장에서 엔진을 만들때 필요한 유체역학, 열역학 까지 알아야될까? 
물론 엔진을 테스트는 해보아야할 것이다. 코드에서도 함수를 구성하면 **무조건** 확인하는 절차가 필요하다. 
하지만 엔진을 만드는데 있어 필요한 세부지식까지 알아야 된다면 자동차 산업은 이렇게 발전하기 힘들었을것이다. 따라서, 엔진 안이 블랙박스처럼 어떻게 작동하는지는 몰라도 차를 조립하는데 문제는 없다. 이러한 특성을 **Abstraction** 이라한다.

~~차 점검받아야하는데~~

<br/>

***

# Function components

```python
def testFunc(x): # x: **formal parameter,** example_function: **function name**
	# """ This is **docstring** and will be shown when call help(example_function) 
  # or testFunc.__doc__ in main program """

	x = x + 1                         #function body       
	return x                          #return 

x = 3
z = testFunc(x)  # x **actual parameter**
```

### Function name

- PEP8 style에 의하면 함수 이름은 소문자이름을 추천한다. ([More about PEP8](https://www.python.org/dev/peps/pep-0008/#function-and-variable-names))
- PEP8 style에 의하면 두 단어로 이루어진 함수 이름은 두번째 단어는 대문자로 시작 (testFunc)
- 시작하는 문자는 영어와 언더스코어(_) 만 허락한다. ~~규칙 못찾아서 내가 해봄~~

### Docstring (optional)

- 함수를 설명하는 공간. Triple quotation mark를 사용한다.
- Docstring을 설정하면 함수에 __doc__ 속성이 추가된다.
- help(함수이름) 이나 함수이름.__doc__ 으로  Docstring을 불러낼 수 있다.

### Function body & Return

- Function body에서는 함수에서 실행시키고 싶은 연산을 구성한다.
- return 값이 없으면 함수는 언제나 `None`을 리턴한다

### Parameters

- **Formal Parameters:** 함수를 정의하는 부분에서 선언되는 parameter (위의 예시에서 첫번째 줄 x)
- **Actual Parameters:** 함수를 Call하는 부분에서 실제로 들어가는 변수 (위의 예시에서 마지막 줄 x)
**(function arguments)**
- 함수 Call(invocation) 을 하면 Formal Parameters는 Actual parameter와 **결합(bound)** 한다.
    1. Positional bound: 
        -   Actual parameter와 Formal parameter의 순서대로 대응하여 결합
    2. Keyword argument bound: 
        -  Actual parameter가 들어갈 부분에 `Formal parameter = Value`  와 같이 선언
        -  이와같은 방법으로 parameter를 bound할 경우, 함수 내에서 default값이 지정됨.
        - Example code

            ```python
            # refer to textbook
            def printName(firstName, lastName, reverse = False):
            	if reverse:
            		print(lastName + ', ' + firstName)
            	else:
            		print(firstName, lastName)

            printName('Olga', 'Puchmajerova')   # Default로 reverse는 false
            printName('Olga', 'Puchmajerova', True) # potisional bound로 다시 지정
            printName('Olga', 'Puchmajerova', reverse = True) #keyword argument

            ### 결과 
            Olga Puchmajerova
            Puchmajerova, Olga
            Puchmajerova, Olga
            ```

<br/>

***

# **Parameter Passing** (bound)

앞서 언급한 bound의 개념은 좀 더 정확한 용어로 **Parameter passing**이라 부른다.
Parameter passing은 actual parameter가 formal parameter에게 값을 전달 해주는 방법인데 
파이썬은 넘겨지는 객체 종류에 따라 **Call by assignment**와 **Call by value**가 정해진다. 

**Call by value**     : Actual이 Formal에 복사되어 local 변수처럼 사용됨

**Call by reference** : Formal에서 Actual의 주소값을 전달받아 사용 

이렇게 객체의 종류에 따라 bound 방법이 달라지는 형태를 **Call by assignment**라고 한다.
(다른 언어의 방법은 이 [블로그를](https://yunmap.tistory.com/entry/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%EC%96%B8%EC%96%B4-Formal-parameter-Actual-parameter-%EA%B7%B8%EB%A6%AC%EA%B3%A0-parameter-passing) 참조하면 좋겠다)

따라서 파이썬은 Call (pass) by assignment를 따르고 자세한 예시는 다음과 같다. 

- **Call by value**: 단일 값을 가지거나, static속성을 가지는 **Immutable object** (int,float,tuples,str)은 call by value를 사용하고 이 경우, actual parameter의 값을 함수내에서 복사해서 사용하기 때문에 원래의 객체에는 영향을 미치지 못한다
- **Call by reference**: list, dict, set와 같은 **mutable object는** actual parameter에 주소값을 가져오는 call by reference를 이용하기 때문에 원래의 객체가 수정이 된다

아마 추론컨데 Immutable에 해당되는 객체들은 용량이 대체로 작아 빠르고 안전한 **call by value**을 사용하고,  Mutable 객체들은 용량이 클 수 있어 복사하기 부담이 되니 주소값을 가져오는 **call by reference** 를 사용하는듯 하다.

이에 대한 자세한 설명은 scope와 함께 서술되어있다.

<br/>

***

# Stack & Scope

## Stack

많은 이유로 함수의 변수가 존재하는 공간은 따로 설정이 되어있는데 이러한 공간을 프로그램 영역에서는 **Scope**라 하고 실제 메모리에서는 **Stack**에 저장이된다. 

그럼 먼저 함수의 변수가 실제로 존재는 메모리 구조 **Stack**에 대해 알아보자

```python
# func2 stack frame
def func2(x):
	return x
# func2 scope

# func1 stack frame
def func1(x1):
	x = func2(x1)
	return x
# func1 stack frame

# main stack frame
a = 1
b = func1(a)
# main stack frame
```

|![Stack의 구조](https://swha0105.github.io/assets/intro_cs/image/lec_4_Untitled.png)
|:--:| 
| Stack의 구조 [출처](http://www.tcpschool.com/c/c_memory_stackframe) |


**Stack** 이라는 자료구조를 배우면 알겠지만, Stack이란 차곡차곡 쌓이고 나중에 쌓인게 먼저 나가는 (LIFO)형식의 자료구조를 뜻한다. 그 Stack이 메모리 구조에도 같은 개념으로 존재한다. 

먼저 main을 실행 시켜 메모리에 존재하는 Stack영역에 main()이라는 stack frame을 형성한다. 이 stack frame은 메모리안에 물리적으로 존재한다. Main stack frame안에 변수 a,b 가 존재하며 이것을 프로그램에서는 **Scope**라 부른다

위의 코드는 `main → func1 → func2` 로 호출을 하기때문에 stack frame도 main, func1, func2로 형성이 된다.  함수를 다 읽고 return을 만나면 return값을 앞선 stack frame에 전달 해주고 해당 stack frame은 종료가 되어 메모리를 비운다. 따라서 Stack frame이 없어지는 순서는 `func2 -> func1 -> main` 이고 이는 Stack의 구조와 개념과 일치한다.

참고로 이러한 형성과정중에 컴퓨터가 허용하는 stack메모리를 넘어가면 **stack overflow** 에러가 난다. ~~홈페이지 이름이 아니다~~ 

Stack frame를 통해 함수의 변수들이 어떻게 실제 메모리에 저장되는지 알아봤으니 이제 프로그램에서 어떠한 일이 발생하는지 알아보자. 

<br/>

## Scope

Scope는 파이썬의 객체들이 유효한 범위를 이야기한다. 한국 돈을가지고 미국에 가서 쓰려고하면 안되듯이 각 객체는 객체에 맞는 범위가 존재하는데 그 범위에 대해 알아보자

```python
# from Bell's slide

## global scope

# f scope
def f(x):
	x = x + 1
	print('in f(x): x =', x)
	return x
# f scope

x = 3 
z = f(x) 

## global scope
```

|![Scope의 구조](https://swha0105.github.io/assets/intro_cs/image/lec_4_Untitled_1.png)
|:--:| 
| Scope의 구조 |

|![Scope의 구조](https://swha0105.github.io/assets/intro_cs/image/lec_4_Untitled_2.png)
|:--:| 
| Scope의 구조 |


위의 그림은 위의 코드가 함수 f 직전까지 실행 되었을 scope구조를 도식화 한 것이고 아래 그림은 코드가 끝났을때 Scope구조 이다. 

위의 코드를 보자면 `global scope` 에서 f,x,z가 선언이 되었고 `f scope`에서 x가 선언되었다. 
당연하게도 컴퓨터는 `global scope`의 x와 `f scope`의 x를 **전혀 다른것으로 인식** 할텐데 그 이유를 알아보자.

코드를 디테일하게 설명하자면 `int` 형태인 Actual parameter인 `global scope의 x` 가 Formal parameter와 결합 (call by value)하여 `f scope의 x`에게 값을 전달하였다. 

위에서 설명했듯이, **call by value**는 **값 자체를 복사**해서 새로운 객체를 만든다. 따라서 새로우 객체  `f scope의 x` 는 메모리에서는 **`f의 stack frame` 에 저장**되기 때문에 `global scope의 x`와는 우리눈에만 같지 전혀 다른 변수가 된다.

그렇다면 **call by reference**를 하는 객체들은 어떻게 될까? 
call by reference를 할 경우, Formal parameter는 Actual parameter의 **주소값**이 되고 **이 경우 원래의 값인 actual parameter에 접근 할 수 있게된다**.! 따라서 append,remove,pop과 같이 객체에 영향을 주는 연산을 함수 내에서 하게 된다면 **return값과 관계없이 객체에 영향을 줄 수 있다**.

**따라서 함수의 actual parameter로 mutable한 객체 (list,set,dict)과 같은 객체는 적합하지 않으며 사용할때 주의를 요구한다.**

이와같은 성질을 이용하여 메인프로그램에 있는 변수들을 보호 하며 필요한 결과만 메인 프로그램에 return 할 수 있다. (**Abstraction**). 하지만 모든일이 그렇듯 규칙을 깨는 **global variable**이라는 놈이 존재하는데 이 친구는 많이 쓰면 코드가 굉장히 지저분해지고 강력히 비추한다. ~~따라서 자세한 설명은 생략한다~~

<br/>

***

# Lambda function

~~아이고 많다 함수 같이 중요한걸 한 강의로 속성을 때리다니~~

이 내용도 수업 내용에 없었지만 실제로 코딩할때 굉장히 많이 사용하므로 함수 내용과 같이 정리하겠다. 

Lambda function은 Anonymous function(익명 함수)라고도 불리는데 함수를 정의할때 function name이 필요 없기 때문이다. 파이썬은 iterable객체를 잘 다룰수 있는 많은 내장함수들이 존재한다. 그러한 내장함수들과 Lambda function을 잘 이용하면 코드를 굉장히 깔끔히 그리고 pythonic하게 짤 수 있다.

아래의 예졔를 보자. 

```python
# function
def plus_ten(x):
	return x + 10

print(plus_ten(1)) # result is 11

# lamda function
plus_ten = lambda x: x + 10
print(plus_ten(1)) # result is 11 
```



|![lambda 함수수](https://swha0105.github.io/assets/intro_cs/image/lec_4_Untitled_3.png)
|:--:| 
| lambda 함수  [출처](https://dojang.io/mod/page/view.php?id=2359) |


함수를 구성하는 formal parameter, return이 다있지만 `plus_ten` 라는 함수의 이름을 선언하는 부분이 빠졌다. x를 받아 10을 더해주는 함수는 lambda라는 이름으로 퉁쳐버렸다.

개념은 쉬운데 왜 이런 함수를 사용할까?? 

앞서 말했듯이 파이썬은 `iterable 객체`를 `iterator 객체` 로 다루는데 굉장히 효율적이다. 
`iterable 객체` 란 list,dict,set,str와 같이 **반복이 가능한 객체**
`iterator 객체` 란 iterable 객체를 **차레대로 꺼낼 수 있는 객체**
 

이 Lambda함수를 `iterable 객체` 로 만들어 파이썬의 `내장 iterator`와 함께 사용 할 수 있다.

예를들어, 리스트 각 요소에 특정 값 보다 큰 값만 반환하는 연산을 **`filter`**, **`map`** 함수를 통해 계산해보자

```python
list_test = [1,50,20,80]
refer_value = 30

# method 1
answer = []
for element in list_test:
	if element > ref_value:
		answer.append(element)

print(answer)

# method 2
def value_finder(list_test,ref_value)
	
	return 

value_finder(list_test,ref_value)

# method 3
answer = list(filter(lambda x:x > ref_value,list_list))

#코드 출처 https://dojang.io/mod/page/view.php?id=2359
```

`map` 함수 뿐만 아닌   `zip` , `reduce` 과 같은 함수와 함께 쓰면 굉장히 강력한 기능이 될 수 있다.


<br/>

*** 

## Reference


1. [http://www.tcpschool.com/c/c_memory_structure](http://www.tcpschool.com/c/c_memory_structure)
2. [https://yunmap.tistory.com/entry/프로그래밍언어-Formal-parameter-Actual-parameter-그리고-parameter-passing](https://yunmap.tistory.com/entry/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%EC%96%B8%EC%96%B4-Formal-parameter-Actual-parameter-%EA%B7%B8%EB%A6%AC%EA%B3%A0-parameter-passing)
3. [https://code13.tistory.com/214](https://code13.tistory.com/214)
4. [https://stackoverflow.com/questions/986006/how-do-i-pass-a-variable-by-reference](https://stackoverflow.com/questions/986006/how-do-i-pass-a-variable-by-reference)