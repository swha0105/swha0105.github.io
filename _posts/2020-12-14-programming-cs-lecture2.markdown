---
layout: post
title:  "[Intro CS] Branching and Iteration"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: cs
comments: False
---
# 2. Branching and Iteration
   - string object type
   - branching and conditionals
   - indentation 
   - iteration and loops

***

# About String
### Definition & Feature
   - Sequence of Character  (단순히 문자열이 아니다!)  
   ⇒ Character는 문자, 숫자를 포함한 어떤것이라도 될 수 있다. 

- Strings are **Immutable**
    
    <details> 
    <summary>example</summary>
    <div markdown="1">   

    s = "hello"    <br />  
    
    try 1: s[0] = 'y'   ⇒  string인 s를 바꾸려고 접근  
    TypeError: 'str' object does not support item assignment 
    <br />  
    try 2: s = 'y' +s[1:len(s)]  ⇒ 기존 s와 "hello" 의 바인딩을 버리고 새로운 객체를 생성해서 바인딩  
    
    ![schematic](https://swha0105.github.io/assets/intro_cs/image/lec_2_Untitled.png)

   </div>
    </details>


### Operator
   1. str + str : Concatenate strings ( `"a" + "b"` ⇒ `"ab"` )
   2. str * int: Repeat strings (`"a"*5` ⇒ `"aaaaa"`)  
   3. str > str: Ascii code 넘버로 비교 (`"a" < "b"` ⇒ `True`, `"A" > "a"`  ⇒ `False`)

        <details> 
        <summary> TMI: Python2 문법</summary>  
         <div markdown="1">   

        Python2에서는  `'4' < 3` 의 결과는 `False`  였다.  Python3에서는 위와같은 표현은 `TypeError` 가 나오며 앞선 강의에서 나온 Static semantic이 틀렸다고 할 수 있다. 
        교과서에 말하기를 이러한 애매한 문법, 즉 Semantics가 좋지 않은 표현들은 현대 컴퓨터 언어에서 퇴출되고 있는 추세라고 한다.

         </div>
        </details>

---

# Branching

### Straight-line programs

- 분기점이 없이 하나의 Flow를 따라가는 코드.

### Branching programs

- Conditional 과 같은 branching statement가 존재하여 두개 이상의 Flow가 존재.
  
![Flow chart for conditional statement ](https://swha0105.github.io/assets/intro_cs/image/lec_2_Untitled_1.png)

---

# Iteration

### Definition

   - 같은것을 많이 반복한다.  So simple!  ( Looping이라고도 불림.)

### For loop

   - Iteration 횟수를 알고 있을때 유용하다.
   - Flow control을 위해 counter를 사용하지만 내재되어있다.
   - For loop로 만들 수 있는 코드는 while loop로도 만들 수 있다.

### While loop

- Iteration이 얼마나 될지 모를때 유용하다.
- Counter를 initialize해줘야한다.
- while loop로 만들 수 있는 코드는 for loop로 표현 못할 수도 있다.

<br/>
<br/>

***

## Reference
[Lecture pdf](https://github.com/swha0105/swha0105.github.io/blob/gh-pages/assets/intro_cs/material/Lec2.pdf)  
