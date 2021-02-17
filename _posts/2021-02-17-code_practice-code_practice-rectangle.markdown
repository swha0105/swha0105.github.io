---
layout: post
title:  "[프로그래머스] 멀쩡한 사각형"
subtitle:  "Legal TOTO"
categories: code_practice
tags: code_practice
comments: true
---



![Signature visualization](https://swha0105.github.io/assets/code_practice/P_rectangle.JPG)

<br/>

# 문제 개요

가로 세로 1cm인 정사각형으로 이루어진 직사각형을 대각선을 따라 잘라냈을때 멀쩡하게 살아있는 사각형의 갯수를 구하여라 

<br/>

# 문제 풀이

1. 최소 공배수를 구한다.
    - 그림을 자세히 보게되면 대각선의 모서리가 닿는 부분이 총 4번있다.   
    - 4의 숫자는 가로세로의 최소공배수와 같다는건 충분히 직관으로 알 수 있다.  

2. 가로세로를 더한 값에 최소공배수를 뺀다. 
    - 흰색 정사각형을 가로,세로에 대해 정사영 시켜보자
    - 가로세로 더한 값에 총 4개가 덜 잘렸고 이는 최소공배수와 같다.

3. 전체 정사각형 갯수에 2번을 뺀다.

<br/>

# 코드 

```python 
def solution(w,h):

    def GCD(w,h):
        return h if w == 0 else GCD(h%w,w)

    return w*h - (w+h) + GCD(w,h)
```

<br/>

# Note

1. 위의 코드는 다른사람 코드를 참고하였다. 내 코드와 알고리즘은 같지만 훨씬 간결하여 참고하였다.
2. 최소공배수 구하는 코드 정도는 외워두자
3. 처음엔 w,h의 ratio로 접근하는 방법을 생각했지만 코드가 너무 더러워져서 갈아엎었다.
