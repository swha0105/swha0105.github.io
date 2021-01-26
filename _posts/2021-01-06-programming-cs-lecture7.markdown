---
layout: post
title:  "[Intro CS] Testing, Debugging, Exceptions, Assertions"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: cs
comments: true
---

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

# 7. Testing, Debugging, Exceptions, Assertions.
- Testing
- Debugging (exceptions, assertions)

<br/>

***

# Testing
- 프로그램이 의도한대로 작동했는지 확인하는 작업


### 테스트 종류에 따른 분류

1. Unit testing 
    - 각각의 기능, 함수들이 제대로 작동하고 있는지 독립적으로 테스트
2. Regression testing
    - 이전에 고쳐졌던 버그들이 재발생하는지 테스트
    - 따라서, 버그 fix log를 잘 정리하는게 중요하다.. ~~제발 좀 하자~~
3. Integration testing
    - Unit testing을 시행한 후, 이상이 없을 시 모든 기능, 함수들을 합쳐 전체 프로그램 테스트

### 테스트 접근방법에 따른 분류

1. Black box testing
     - 코드 내용을 모른채, Docstring 과 같은 Specification만 볼 수 있는 상태에서 행하는 테스트.

2. Glass box testing
     - 소스코드를 확인 할 수 있는 상태에서 행하는 테스트


### Guidelines for Glass box test

1. **Branches**

    **`Path-complete`** 와 **`Boundary condition`** 모두 확인하는 테스트 케이스를 구성
    ```python
    # Example code
    # 절대값을 리턴하는 함수
    def abs(x):
        if x<-1:
            return -x
        else:
            return x
    ```

    코드의 모든 구간을 통과하여 테스트를 거쳤을때 **`Path-complete`** 라고 한다

    위의 함수가 제대로 작동하는지 확인하기 위하여  $$x \subset [2,-2]$$ 를 구성하여 테스트를 해본다고 가정하자.  이 경우 if의 경우와 else의 경우를 모두 거치기 때문에 **`Path-complete`**했다고 할 수 있다. 

    하지만 $$x = -1$$ 일 경우는 함수값을 -1로 리턴하기 때문에 위의 함수가 제대로 작동한다고 할 수 없다. 따라서, if 문의 **`boundary condition`**을 포함한 테스트 케이스를 구성하여 테스트 할것을 추천한다.

2. **for loops**
    - Loop에 안들어 갈 경우
    - Body of Loop가 정확히 한번 돌아갈 경우
    - Body of Loop가 한번 이상 돌아갈 경우

    위의 경우 모두를 테스트 케이스에 포함해야한다.

3. **while loops**
    - for loop와 동일 하지만 exit하는 모든 경우를 테스트 케이스에 포함해야한다.
    (ex, if break, if return , while(statement 1 and statement 2 ) )

<br/>

***

# Debugging

개인적인 생각인데 `코드를 개발한다` 는 `디버깅 한다` 와 크게 다를바 없는거 같다..

- 테스트에서 이미 발견된 버그를 고치는 일련의 과정

## Type of Error (bug)

1. **Syntax & Static Semantics Error (Overt bug)**
    - 문법이나 잘못된 접근을해서 생기는 에러 (IndexError, TypeError, NameError, Syntax Error .. )  이러한 에러들은 언어에 익숙해지면 금방 해결한다. ~~솔직히 이런 에러뜨면 반가움~~
2. **Logic Error (Covert bug)** 
    - 코드가 정상적으로 작동했지만 결과값이 의도한바와 다를때 생기는 에러

    Solution for Logic Error

    1. 위의 Gulideline를 참고해 테스트 케이스를 다시 구성하고 가설을 다시 검증한다. 
    2. 알고리즘 전개도를 그리거나 개념을 그린다.
    3. ~~혼잣말하며 코드를 설명한다~~ (내가 가장 많이 사용하는 방법)
    4. ~~쉬고, 한숨 자고, 먹고 다시와라~~  (교수님이 말씀하셨지만 진짜 공감! 절대 오기부리지 말자)

## How to avoid bug by Python tools

1. **Try - Except** 

    ```python

    try: 
        a = int(input("Tell me one number: "))
        b = int(input("Tell me another number: "))

        print("a/b = ", a/b)
        print("a+b = ", a+b)
    except ValueError:
        print("Could not convert to a number.")
    except ZeroDivisionError:
        print("Can't divide by zero")
    except:
        print("Something went very wrong")
    ```

    위의 함수를 실행 시켰을때 a,b 를 적절한 수를 넣으면 에러가 안나고 print(a/b) print(a+b) 가 실행 될 것이다. 

    그런데 a,b 를 문자열로 넣는다던가, 분모를 0으로 넣으면 Try 구문이 실행되는 도중 Error가 발생할 것이다. 에러가 발생할 경우 **try구문을** 중지하고 에러에 따라 지정된 except문으로 넘어가게 된다. 

2. **assert**

    ~~계속 아는거 나와서 지겨워 하는 도중 득템함~~

    - 함수 body에서 처음이나 마지막에 보통 사용됨
    - 함수의 input이나 output의 형태를 체크 하는 용도로 사용되며 원하지 않는 형태일 경우 함수가 아닌 **프로그램 자체를 중지한다.**

    ```python
    def return_plus_10(N):
        assert type(N) == int, 'Input type is not integer'
        return N+10

    print(return_plus_10(10))
    print("N = 10 is fine")

    print(return_plus_10('a'))
    print("it will be not printed")
    ```

    위의 예시에서 마지막 print문은 실행 되지 않는다. 

    `assert type(N) == int` 가 False이기 때문에 옆에 있는 expression 구문을 호출하고 프로그램 자체가 종료된다.

    이러한 성질은 OOP할때 굉장히 유용할것으로 보인다

    OOP는 다음강의에.. 

~~점점 정리내용이 짧아지는건 기분탓인가?~~


<br/>

*** 

## Reference

[Lecture pdf](https://github.com/swha0105/swha0105.github.io/blob/gh-pages/assets/intro_cs/material/Lec7.pdf) 
