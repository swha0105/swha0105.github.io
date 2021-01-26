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


# Label spreading 개요 및 경험
Label spreading 알고리즘은 **Semi-Supervised Learning 알고리즘** 중 하나로써 개인적으로 [Filament Project](https://swha0105.github.io/projects/2021/01/24/projects-projects-filament/) 를 할때 사용하였다.  
이 프로젝트는  총 4개의 구조를 가지는 데이터에서 정량적으로 구분이 되는 2개의 구조와 (`Cluster`, `Void`) 그렇지 않은 구조 특정 구조를 2개를 (`Filament`, `Wall`)구분하여 분류하는 프로젝트였다. 

이 두개의 구조는 (`Filament`,`Wall`) 3차원 공간 데이터상에 존재하며 기하학적 정보와 물리학적 정보를 함께 보며 판단해야하고 또한 주변의 환경들도 문맥적으로 고려해야한다.

우리의 가설은 `Filament`는 `Cluster`와, `Wall`은  `Void`와 4차원 공간상에서 좀 더 가까이 있을꺼라 판단하였다. 4차원 공간은 물리 데이터 3개 (온도, 밀도, X-ray)와 기하학적 정도를 나타내는 데이터 (Shape Strength)를 계산하여 4차원 데이터 공간을 구성하였다.

따라서, 정량적인 지표로 구분이 가능한 `Cluster`와 `Void`를 **Label**로 가정하고 **Unlabel**데이터 인 `Filament`와 `Wall`이 4차원 공간상에서 어디와 더 가까운지 판단하려고 했다.

<br/>

# Mathematical Principle
Label spreading은 모든 데이터의 거리를 계산하고 Normalized Graph Laplacian을 이용하여 Unlabel 데이터에 Label을 지정해주며 Convergency가 될때까지 Iteration하는 알고리즘이다.

이게 무슨말인가?? 아래 수학수식들을 보자  

데이터 셋 (Data set) $$X = [ x_{1},x_{2},...x_{l},x_{l+1},...,x_{n} ]$$ 안에   
레이블 존재하는 지점의 데이터 (Label point) $$x_{L} $$  $$(1 \leq L \leq l)$$ 와   
레이블 존재하지 않는 지점의 데이터 (Unlabel point) $$x_{U} $$ $$(l+1 \leq U \leq n)$$  가 있다고 가정하자.

그리고 데이터 셋에 대응되는 레이블 데이터 $$Y = [y_{1},y_{2}, ... y_{l}, y_{l+1}, ... ,y_{n} ]$$ 는  
레이블이 존재하면 $$y_{i} \subset [-1,1]$$ $$(1	\leq L \leq l)$$  이고  
레이블이 없으면 $$y_{i} = 0$$ $$(l+1 \leq U \leq n)$$  이다

그리고 알고리즘의 주인공 Normalized Graph Laplacian을 보자.  
$$S = D^{-\frac{1}{2}}WD^{-\frac{1}{2}}$$ (D는 [Degree Matrix](https://en.wikipedia.org/wiki/Degree_matrix))



<details>
<summary> Notation 관련 </summary>
<div markdown="1">   
원래 Normalized Graph Laplacian은 $$L^{sym} = D^{-\frac{1}{2}}LD^{-\frac{1}{2}}$$ 이다.  
L = D - A 이고 A는 adjacency matrix 즉, 우리가 사용하고 있는 W와 같은 개념이다. 
이 정의를 위 수식에 대입하면 $$L^{sym} = I - D^{-\frac{1}{2}}AD^{-\frac{1}{2}}$$가 되어야 한다.  
왜 Label spreading에서 Normalized Graph Laplacian을 $$S = D^{-\frac{1}{2}}WD^{-\frac{1}{2}}$$ 이렇게 구성하는지는 잘 모르겠다. 엄밀한 정의와 다르니 L 대신 S notation을 쓰는게 아닌가 추측해본다.

</div>
</details>

이제 수학적 정의 부분은 끝났다. 알고리즘을 보자.  

1. W (adjacency matrix)를 계산 한다. 
    - W는 모든 포인트들끼리의 거리 정보를 담고 있다.
    - W의 거리 정보는는 knn, rbf 등 여러 형태로 계산 될 수 있다.
2. S (Normalized Graph Laplacian Matrix) 를 계산한다.
    - Noramlized하지 않고 사용하는 알고리즘은 Label propagation
    - Noramlized할 경우 대칭성??
    - 의미

  As mentioned
above, normalization has been commonly practiced and appears to be useful, but there hasn’t been
any solid theoretical justification on why it should be useful
3. $$Y^{t+1} = \alpha S Y^{t} + (1-\alpha)Y^{0}$$ 이 Convengence 가 일어날때 까지 Iteration한다.
    - Alpha는 Hyperparameter로써, Initial 정보를 얼마나 간직할것인지 결정한다. 
    - Alpha가 작을수록 Initial정보를 바꾸지않고 계산한다.


# 한줄 요약

# 실제 사례
내 예시


## Reference



[1]. Rie Johnson, Tong Zhang, On the Effectiveness of Laplacian Normalization for Graph
Semi-supervised Learning, , JMLR, 2007, [Paper link](https://www.jmlr.org/papers/volume8/johnson07a/johnson07a.pdf)  
[1]. Learning with Local and Global Consistency (paper)  

[1]. Mastering machine learning algorithms : expert techniques to implement 
popular machine learning algorithms and fine-tune your models (book)
[]
논문, Scikit learn