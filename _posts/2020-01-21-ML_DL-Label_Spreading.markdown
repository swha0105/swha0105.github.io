---
layout: post
title:  "[ML Algorithms] Label Spreading"
subtitle:   "Semi-supervised Learning"
categories: ml_dl
tags: ml
comments: true
---

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>


# Label spreading 요약

**`Label spreading`** 에 대해 한줄 요약하자면, 알려진 Label로 부터 모르는 Unlabel에 대해 `특정 공간상의 상대적인 거리`를 구하여 가장 가까운 알려진 Label로 지정하는 알고리즘이다 

이 알고리즘은 데이터 셋이 Label과 Unlabel이 섞여있고, Unlabel에 대한 정보를 Label로 부터 추측하고 싶을때 사용할 수 있고 **`Semi-supervised learning`**중 하나이다. 

<br/>

---


# Mathematical Principle
위의 요약을 수학적으로 말하자면  
>Label spreading은 모든 데이터의 거리를 계산하고 **Normalized Graph Laplacian**을 이용하여 Unlabel 데이터에 Label을 지정해주며 수렴할 때까지 iteration하는 알고리즘이다.

이게 무슨말인가?? 라고 생각들면 당연한거다. 천천히 아래 수식을 따라가보자. 

## Preparation
데이터 셋 (Data set) $$X = [ x_{1},x_{2},...x_{l},x_{l+1},...,x_{n} ]$$ 안에   
레이블 존재하는 지점의 데이터 Label point는 $$x_{L} $$  $$(1 \leq L \leq l)$$ 이고  
레이블 존재하지 않는 지점의 데이터 Unlabel point은 $$x_{U} $$ $$(l+1 \leq U \leq n)$$  가 있다고 가정하자.

그리고 데이터 셋에 대응되는 레이블 데이터 $$Y = [y_{1},y_{2}, ... y_{l}, y_{l+1}, ... ,y_{n} ]$$ 는  
레이블이 존재하면 $$y_{i} \subset [-1,1]$$ $$(1	\leq L \leq l)$$  이고  
레이블이 없으면 $$y_{i} = 0$$ $$(l+1 \leq U \leq n)$$  이다

그리고 알고리즘의 주인공 Normalized Graph Laplacian은  
**$$S = D^{-\frac{1}{2}}WD^{-\frac{1}{2}}$$** (D는 [Degree Matrix](https://en.wikipedia.org/wiki/Degree_matrix))  라고 정의 한다.  
이제 수학적 정의 부분은 끝났다. 알고리즘을 보자.  

<details>
<summary> Notation 관련 의문</summary>
<div markdown="1">   
원래 Normalized Graph Laplacian은 $$L^{sym} = D^{-\frac{1}{2}}LD^{-\frac{1}{2}}$$ 이다.  
L = D - A 이고 A는 adjacency matrix 즉, 우리가 사용하고 있는 W와 같은 개념이다. 
이 정의를 위 수식에 대입하면 $$L^{sym} = I - D^{-\frac{1}{2}}AD^{-\frac{1}{2}}$$가 되어야 한다.  
왜 Label spreading에서 Normalized Graph Laplacian을 $$S = D^{-\frac{1}{2}}WD^{-\frac{1}{2}}$$ 이렇게 구성하는지는 잘 모르겠다. 엄밀한 정의와 다르니 L 대신 S notation을 쓰는게 아닌가 추측해본다.

</div>
</details>

<br/>

## Algorithm

1. W (adjacency matrix)를 계산 한다. 
    - W는 모든 포인트들끼리의 거리 정보를 담고 있다.
    - W의 거리 정보는는 knn, rbf 등 여러 형태로 계산 될 수 있다.
2. S (Normalized Graph Laplacian Matrix) 를 계산한다.
    - Noramlized하지 않고 사용하는 알고리즘은 Label propagation
    - Noramlized의 정확한 역할은 [Ref 2 ](https://swha0105.github.io/assets/ml/paper/Learning_with_Local_and_Global_Consistency.pdf) 있는데 너무 어려워서 정리 못함..    
 
3. $$Y^{t+1} = \alpha S Y^{t} + (1-\alpha)Y^{0}$$ 이 수렴할 때까지 Iteration한다.
    - Alpha는 Hyperparameter로써, Initial 정보를 얼마나 간직할것인지 결정한다. 
    - Alpha가 작을수록 Initial정보를 바꾸지않고 계산한다.


~~Label spreading을 처음 소개한 논문은 이렇게 친절하지 않다~~

<br/>

---

# 사용 사례

나는 **`Label spreading`** 알고리즘을 이 [프로젝트](https://swha0105.github.io/projects/2021/01/24/projects-projects-filament/)  에서 사용했다. 

프로젝트는  총 4개의 구조를 가지는 데이터에서 정량적으로 구분이 되는 2개의 구조와 (`Cluster`, `Void`) 그렇지 않은 구조 특정 구조를 2개를 (`Filament`, `Wall`)구분하여 분류하는 프로젝트였다. 

이 두개의 구조는 (`Filament`,`Wall`) 3차원 공간 데이터상에 존재하며 기하학적 정보와 물리학적 정보를 함께 보며 판단해야하고 또한 주변의 환경들도 문맥적으로 고려해야한다.

우리의 가설은 `Filament`는 `Cluster`와, `Wall`은  `Void`와 4차원 공간상에서 좀 더 가까이 있을꺼라 판단하였다. 4차원 공간은 물리 데이터 3개 (온도, 밀도, X-ray)와 기하학적 정도를 나타내는 데이터 (Shape Strength)를 계산하여 4차원 데이터 공간을 구성하였다.

따라서, 정량적인 지표로 구분이 가능한 `Cluster`와 `Void`를 **Label**로 가정하고 **Unlabel**데이터 인 `Filament`와 `Wall`이 4차원 공간상에서 어디와 더 가까운지 판단하려고 했다.

자세한 결과는 [링크](https://swha0105.github.io/projects/2021/01/24/projects-projects-filament/)에 있지만 간단하게 말하자면 우리의 가설대로 4차원 공간상에서 두개의 구조를 충분히 구분할 수 있었다. 하지만 계산량이 데이터에 기하급수적으로 비례하기 때문에 높은 해상도의 데이터를 사용하지는 못하였다.  
코드는 Scikit-learn에 있는 모듈을 사용하였고 비교적 간단히 전처리만 하면 사용할 수 있기에 기억하고 있으면 유용한 알고리즘인거 같다.


<br/>

---


## Reference

[1]. Rie Johnson, Tong Zhang  On the Effectiveness of Laplacian Normalization for Graph Semi-supervised Learning, , JMLR, 2007, [Paper link](https://swha0105.github.io/assets/ml/paper/On_the_Effectiveness_of_Laplacian_Normalization_for_Graph.pdf) 

[2].Dengyong Zhou, Olivier Bousquet, Thomas Navin Lal,
Jason Weston, and Bernhard Scholkopf   
Learning with Local and Global Consistency [Paper link](https://swha0105.github.io/assets/ml/paper/Learning_with_Local_and_Global_Consistency.pdf) 

[1]. Mastering machine learning algorithms : expert techniques to implement 
popular machine learning algorithms and fine-tune your models (book)
[]
논문, Scikit learn