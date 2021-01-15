---
layout: post
title:  "[Intro CS] What is computation"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: cs
comments: true
---


[Lecture pdf](https://github.com/swha0105/swha0105.github.io/blob/gh-pages/assets/intro_cs/material/Lec1.pdf)
 

## Topics

이 렉쳐의 전반적인 Topic들을 소개한다.

1. **Represent knowledge with data structures / iteration and recursion as computational metaphors**

    ⇒ objects and knowledges,  Non-linear program flow control,
        컴퓨터 구조와 컴퓨터 언어의 전반적인 통찰

2. **Abstraction of procedures and data types / organize and modularize systems using object classes and methods**
⇒ 코드를 readable, re-understanding 하기 쉽게 구성하는 법.
3. **Different classes of algorithms, searching and sorting  / complexity of algorithms**
⇒ 어느 코드가 더 좋은지 평가 하는 법. 

위와 같이 구성되고 이 강의는 당연히 1번에 해당된다.  


---

# Computer basics

## What Does a computer do

1. Performs calculations.
2. Remember results.

 ~~게임 에뮬레이터가 아니다~~

## **Type of Knowledge**

1. **Declarative knowledge**   
    - Statements of fact
    - a = 3, b =3 , Coronavirus is dangerous 와 같은 정확한 정보 (지식)
2. **Imperative knowledge**
    - Series of "How to" or Recipe
    - 코로나를 피하는 방법 (1. 손씻기 2. 마스크 착용 3. 어쩌구저쩌구..)
    - 이러한 Imperative knowledge 중에 
    - **간단한 몇가지 단계로 구성되어 있으며,
    - 각 단계들이 실행될 시기를 지정하는 Flow of control의 유무
    - 언제 끝날지 알려주는 수단의 존재**
    가 있을때 이러한 knowledge series를 **Algorithm**이라 부른다  

</br>

## Basic primitives

Alan Turing은 6가지 원칙만(Primitives) 있으면 어떠한 계산도 할 수 있다고 증명했다. 그 6가지 원칙은 아래와 같다.

Move left / Move right / Read / Write / Scan / Do nothing 

현대의 프로그래밍 언어들은 위와같은 원칙 외에 더 좋은 원칙들을 가지고 있고, 심지어 프로그래머들이 코드내에서 원칙들을 만들 수도 있다. (Inheritance of Abstract Method)

하지만 이 원칙으로 부터 우리는 다음과 같은 사실을 알 수 있는데

> 컴퓨터 언어로 쓰여진 어떠한 알고리즘이라도 다른 컴퓨터 언어로 똑같이 구현될 수 있다

너무 당연하게 생각했지만 이렇게 정리가 되어있다는게 수학 공리 같은 느낌이다.

## Aspect of languages

언어의 기본적인 역할은 정보를 전달하는데 있다. 우리가 쓰는 언어는 사람들간의 소통, 컴퓨터 언어는 사람과 컴퓨터와의 소통하는데 그 목적이 있다. 하지만 컴퓨터는 융통성이 없기 때문에 ~~멍청이~~ 사람의 언어와는 다르게 까다로운면이 존재한다. 아래 표는 사람의 언어와 컴퓨터 언어의 차이점을 비교한 표이다. 

| |Human languages|Computer language|
|------|---|---|
|Primitive Constructs |단어들|Numbers, Strings, Simple operators|
|Syntax|cat dog boy (명사, 명사, 명사)  ⇒ Not syntactically valid|"Hi"5 (operand,operand) ⇒ Not Syntactically valid|
|Static Semantics|I are hungry (명사 동사 명사) ⇒ Syntactically valid, but static semantic error | 	3+"hi" (operand, operator, operand) ⇒ Syntactically valid, but static semantic error
Semantics|같은 문장이라도 다양한 뜻 존재.| 오직 한가지 뜻이 존재. |  

---  

## Languages type

- Low level vs High level: 프로그램의 지시사항들의 머신에 가까운지, 유저에 가까운지 
⇒ C,C++은 변수에 접근하기위헤 메모리 주소값을 이용하지만 파이썬은 그러지 않는다. (해도 되지만)
- General vs targeted to application domain: 프로그램 언어가 다양한 곳에서 사용 될 수 있는지?

    ⇒ C,C++은 거의 모든곳에서 사용 될 수 있지만 내가 대학원생때 사용한 Fortran은 계산밖에 못한다

- Interpreted vs Compiled

    ⇒ Source code를 컴퓨터가 알아 들을수 있게 Compile을 하는 언어 (C,Fortran), Interpreter를 통해 결과를 실시간으로 피드백 해주는 언어 (Python,MATLAB) 

    - 📋 TMI: Compiler언어 vs Interpreted 언어

        두 언어의 차이는 `컴파일 시점` 이다. 런타임 전에 컴파일을 하면 compiler 언어, 그렇지 않으면 interpreted언어 

        **Compiler 예시** 
        `코드` --(컴파일러)-->> `어셈블리어` --(어셈블러)-->> `기계어` --(링커)-->> `실행파일` -->> `런타임` 

        - 컴파일 언어의 소스코드는 운영체제별 특징이 있어 운영체제마다 **별도의 라이브러리** 필요
        - 어셈블리어는 CPU에 의존적이기 때문에, 프로세스를 옮겨다니며 사용할 수 없음.

        **Interpreted 예시** 

        `코드` --> 실행 & 런타임 시작 --> (컴파일) --> `바이트코드` --> VM --> `기계어`

        - 런타임중 프로그램을 Line by Line으로 해석하며 실행
        - 바이트코드는 가상머신에서 돌아가기 때문에 CPU에 의존적이지 않다
        - 운영체제마다 별도의 라이브러리가 필요 없지만 Python을 설치할때 OS를 선택하는 이유는 바이트코드에서 기계어로 넘어갈때 어떤 기계어로 바꿀지 결정하기 위해서.

        출처: [링크](https://wayhome25.github.io/cs/2017/04/13/cs-14/)

## What is program

코드의 모든것들은 Object로 이루어져 있고, 이러한 Object들을 다루는것들을 Program이라 한다.

## Type of Object

Scalar: 나눌 수 없는 Object들 (Integer, real number, Boolean)

non-scalar: Object안에 접근할 수 있는 내부구조가 존재하여 Object들을 저장 (List, dictionary..)
