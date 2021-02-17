---
layout: post
title:  "[ML/DL Lecture] Kmeans"
subtitle:   "unsupervised learning"
categories: ml_dl
tags: ml
comments: true
---

**들어가기전..**  
KNN 알고리즘은 예전에 [Filament project](https://swha0105.github.io/projects/2021/01/24/projects-projects-filament/) 에서 사용해보았다. 비록 원하던대로 결과가 나오지 않아 결과에는 포함이 되지않았지만 ~~온갖 알고리즘 다 써봄~~ 이것저것 해보면서 수학적인 직관이 왔었다. 이번 기회를 통해 직관을 정리해보려고 한다. [김성범 교수님 강의를](https://www.youtube.com/watch?v=8zB-_LrAraw&t=1658s) 참고하였다.

<br/>

# KNN 특징 및 알고리즘 

- ###  Classification & Prediction 두가지 모두 다 가능한 모델 

## KNN 특징 

**1. Instance based**  
각각의 관측치(instance)만 이용하여 테스팅 데이터에 대한 예측   

**2. Memory based**  
모든 학습 데이터를 메모리에 저장.  

**3. Lazy learning**
테스팅 데이터가 들어와야 작동.  

> ### Model based vs Instance based
1. model based learning (데이터로부터 모델을 생성)
  - regression (linear, logistic )
  - Neural network
  - decision tree & random forest
  - SVM
2. instance-based learning (모델 생성없이 인접 데이터 분류)
  - KNN
  - Locally weighted regression


## Algorithm without mathematics

1. 테스팅 데이터에서 k개의 학습 데이터 탐색
2. k개의 데이터의 평균으로 리턴.

> - 기본 KNN에 대해서만 평균으로 리턴, 변형되는 weighted KNN 같은 경우 다름.

<br/>

---

# Hyperparameter

- KNN은 Instance based model로서 parameter는 없다.
- KNN은 K, distance를  Hyperparameter로 가진다.

## 1. k
**테스팅 데이터에서 거리상 근접한 고려해야될 데이터 갯수 k**
- k 가 작을수록 local한 boundary를 구성. (overfitting)
- k 가 클수록 global한 boundary를 구성. (underfitting)

#### 최적의 k 산출 방법
- **metric을 구성하여 grid search**  ~~노가다~~

  1. classification model metric  
misclasserror = $$ \frac{1}{k} \sum_{i=1}^{k} I(c_{i} \neq \hat{c_{i}})$$
> $$c_{i}$$ 실제값 $$\quad $$ $$\hat{c_{i}}$$ 예측값

  2. prediction model metric  
$$SSE_{k}$$ (Sum of square Error) = $$ \sum_{i=1}^{k} (y_{i} - \hat{y_{i}}^2)$$

<br/>

## 2. distance (1-similarity)
- 데이터 끼리의 거리 측정하는 측도.
- similarity와 반비례 하는 특성을 가짐.  
- **변수(feature)끼리 다른 범위, 분산을 가지고 있기 때문에 데이터 정규화 or 표준화가 필수**

**1. Euclidean distance**  

$$ d_{x,y} = \sqrt{ \sum_{i=1}^{p} (x_{i} - y_{i})^2  } $$ 

- L2 norm와 같은말.

**2. manhattan distance**  

$$ d_{x,y} = \sum_{i=1}^{n} |x_{i}-y_{i}|$$

- 격자형태로 움직일수 있는 체스판에서의 거리.

**3. Mahalanobis distance**  

$$ d_{X,Y} \sqrt{ (X - Y)^{T} \Sigma^{-1} (X -Y)}$$

> $$  \Sigma^{-1} $$: inverse of covariance matrix 

- Mahalanobis distance의 수식은 타원의 방정식과 같다. 

**증명**

let 
$$X = \begin{pmatrix}x_{1} \\ x_{2} \end{pmatrix} \quad \quad Y = \begin{pmatrix}y_{1} \\ y_{2} \end{pmatrix}  \quad \quad \Sigma^{-1} = \begin{pmatrix}s_{11}^{-1} & s_{21}^{-1} \\ s_{12}^{1} & s_{22}^{-1} \end{pmatrix}$$

$$ d_{X,Y}^2 = (X-Y)^{T} \Sigma^{-1} (X-Y)$$  
= $$ (x_{1}-y_{1})^{2} s_{11}^{-1} + 2(x_{1}-y_{1})(x_{2}-y_{2})s_{12}^{-1} + (x_{2}-y_{2})^{2} s_{22}^{-1} \quad \quad (s_{12}^{-1} = s_{21}^{-1}) $$ 

Mahalanobis distance를 풀어쓰면 위와같이 된다.  

여기서 편의를 위해 비교할 대상은 원점이라 하자. $$ Y = \begin{pmatrix} 0 \\ 0 \end{pmatrix}$$ 을 위 수식에 대입하면

$$ x_{1}^{2} s_{11}^{-1} + 2x_{1}x_{2}s_{12}^{-1} + x_{2}^{2} s_{22}^{-1} = c^2$$

이와 같은 수식으로 변형이 된다. 이 수식은 $$x_{1}$$와 $$x_{2}$$에 대한 타원의 방정식이다.


이 수식을 해석하자면 **x1,x2 각각에 대해 분산을 고려하여 거리를 계산** 한다는 의미이다. s는 x1,x2의 관계를 담은 공분산 (covariance)이다.


![Mahalanobis distance를](https://swha0105.github.io/assets/ml/img/mahalanobis.JPG)  

<br/>


**4. pearson correlation distance**

$$ d_{X,Y} = 1-r$$  

r = x,y에 대한 correlation $$(-1 \leq r \geq 1 )$$


**5. spearman rank correlation distacne** 

$$ d_{X,Y} = 1 - \rho$$  

$$ \rho = 1 - \frac{6 \sum_{i=1}^n (rank(x_{i}) - rank(y_{i}))^2 }{n (n^{2}-1)}$$

$$rank(x_{i})$$ 는 데이터 집합 x에 대해 data의 값으로 sorting을 하였을때 $$x_{i}$$ 의 위치 (순서)

<br>

---

# KNN 장단점 & weighted KNN 

### 장점 
- noise에 robust (특히 mahalanobis distance)

### 단점
- Hyperparameter(K, distance)를 설정하기 어려움
- Instance based & lazy learning 특성때문에 테스팅 시간이 오래걸림.
- 고차원 데이터엔 distance 정의가 힘들어 잘 작동 안된다고 알려져있음

<br/>

## weighted KNN
앞서 KNN을 사용할때, 테스팅 데이터 근처 K 데이터값을 평균내어 예측 및 분류를 한다고 하였다. 하지만 상식적으로 생각해보면 근처 K값들이 각기 다르고 테스팅데이터와의 거리와 같지않다. 이를 종합해보면 K에 대해 평균 내는건 unfair하다고 생각을 할 수 있다.

따라서 거리와 값에대해 weighting을 주는 weighted KNN 개념이 등장한다.

**1. prediction** 

$$ \hat{y_{new}} = \frac{\sum_{i=1}^{k} w_{i}y_{i}}{\sum_{i=1}^{k} w_{i}}   \quad \quad w_{i} = \frac{1}{d_{new,x_{i}}^2 }$$

**2. classification**

$$ \hat{c_{new}} = \underset{c}{argmax} \sum_{i=1}^{k} w_{i} I(w_{i} \subset c ) \quad \quad w_{i} = \frac{1}{d_{new,x_{i}}^2}$$


<script>
MathJax.Hub.Queue(["Typeset",MathJax.Hub]);
</script>

<script>
MathJax = {
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']]
  },
  svg: {
    fontCache: 'global'
  }
};
</script>
<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-svg.js">
</script>
