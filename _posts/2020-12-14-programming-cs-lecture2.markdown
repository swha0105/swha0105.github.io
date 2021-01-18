---
layout: post
title:  "[Intro CS] Branching and Iteration"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: cs
comments: true
---

# 2. Branching and Iteration
# Contents

- string object type
- branching and conditionals
- indentation 
- iteration and loops


# About String
### Definition & Feature
 
- Sequence of Character  (단순히 문자열이 아니다!)
⇒ Character는 문자, 숫자를 포함한 어떤것이라도 될 수 있다. 

- Strings are **Immutable**
    
    <details> 
    <summary>example</summary>
    s = "hello"  <br />  <br />  
    
    try 1: s[0] = 'y'   ⇒  string인 s를 바꾸려고 접근  
    TypeError: 'str' object does not support item assignment 
    <br />  
    try 2: s = 'y' +s[1:len(s)]  ⇒ 기존 s와 "hello" 의 바인딩을 버리고 새로운 객체를 생성해서 바인딩  
    No error   <br />  

    
    ![schematic](https://swha0105.github.io/assets/intro_cs/image/lec_2_Untitled.png)
    </details>

### Operator
1. str + str : Concatenate strings ( `"a" + "b"` ⇒ `"ab"` )
2. str * int: Repeat strings (`"a"*5` ⇒ `"aaaaa"`)  
3. str > str: Ascii code 넘버로 비교 (`"a" < "b"` ⇒ `True`, `"A" > "a"`  ⇒ `False`)

    <details> 
    <summary> TMI: Python2 문법</summary>  
    Python2에서는 `'4' < 3` 의 결과는 `False`  였다. Python3에서는 위와같은 표현은 `TypeError` 가 나오며 앞선 강의에서 나온 Static semantic이 틀렸다고 할 수 있다. 

    교과서에 말하기를 이러한 애매한 문법, 즉 Semantics가 좋지 않은 표현들은 현대 컴퓨터 언어에서 퇴출되고 있는 추세라고 한다.
    </details>


# Branching

### Straight-line programs

- 분기점이 없이 하나의 Flow를 따라가는 코드.

### Branching programs

- Conditional 과 같은 branching statement가 존재하여 두개 이상의 Flow가 존재.
  
![Branching_schematic](https://swha0105.github.io/assets/intro_cs/image/lec_2_Untitled_1.png)



Flow chart for conditional statement 

이러한 branching programs의 장점은 시간이 줄어든다는 것이다. 
만약, Straight-line programs이 n개의 line으로 코드가 구성되어있다고 하면 runtime은 n*factor로 이루어질것이다. 하지만 Branching이 코드가 n개의 line으로 구성되어있다 해도, runtime은  n*factor보다 작거나 같을것이다. (교과서에 설명 되어있는 문장. 아마 Linear & Ideal 한 상황 가정인듯) 

# Iteration

### Definition

- 같은것을 많이 반복한다.  So simple!  ( Looping이라고도 불림.)

### For loop

- Iteration 횟수를 알고 있을때 유용하다. 
for i in range(start,stop,step)   ⇒ iteration until stop - 1
for i in array (= [0,1,2,3]) ⇒ Iteration all items
- Flow control을 위해 counter를 사용하지만 내재되어있다.
- For loop로 만들 수 있는 코드는 while loop로도 만들 수 있다.

### While loop

- Iteration이 얼마나 될지 모를때 유용하다.
- Counter를 initialize해줘야한다.
- while loop로 만들 수 있는 코드는 for loop로 표현 못할 수도 있다.