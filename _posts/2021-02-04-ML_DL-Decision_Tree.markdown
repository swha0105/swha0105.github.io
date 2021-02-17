---
layout: post
title:  "[ML/DL Lecture] Decision Tree"
subtitle:   "supervised Learning"
categories: ml_dl
tags: ml
comments: true
---

**들어가기전..**  
Random forest를 정리하기전 Decision Tree에 대해 알고 있는것들을 구체적으로 정리하기 위해 포스팅을 한다.  
Review paper나 책을 찾아보았지만 깔끔하게 정리된것을 못찾아서 발품(?)팔아서 여기저기 찾으며 혼자 정리 해보았다.  
주로 [김성범 교수님 강의를](https://www.youtube.com/watch?v=xki7zQDf74I) 참고하였다.

<br/>

Regression, Classification model에 대해  
- Model 구조
- Cost function
- 분할 변수와 분할 점

<br/>

---

# Decision Tree 

- Data에 내재되어 있는 패턴을 변수의 조합으로 나타내는 예측/분류 모델을 나무의 형태로 만드는것
- 나무의 Branch처럼, IF문의 Branch를 나무처럼 수없이 많이 만들어 정답이 될 수 있는 후보 집합을 좁혀나간다.  
- 이떄 Branch는 데이터를 2개이상의 **균일한** 부분집합으로 분할 하는 조건이여야 한다.   
   
$$ 균일한 = \begin{cases} 비슷한 \; 범주 \quad (분류 문제) \\ 비슷한 \; 수치  \quad (예측 문제)\end{cases}$$

- 모델을 구성하였을때, Branch가 뻗은 모양이 나무를 뒤집어 놓은거 같다고 하여 Tree라고 한다..

~~스무고개 게임과 같다~~  

<br/>

## Model 구조

- **Root node, Intermediate node, Terminal node** 로 이루어져 있다.

**`Root node`** (뿌리 마디): Branch가 일어나기전 가장 처음 조건. 하나만 존재한다.   
**`Intermediate node`**(중간 마디): Branch가 생기는 마디. 데이터가 이 노드를 통과할때 마다 최소 2개 이상의 집합으로 분류가 된다.   
**`Terminal node`**(끝 마디): Branch가 더이상 생기지 않고 끝나는 마디.

![decision tree](https://swha0105.github.io/assets/ml/img/DecisionTree.JPG)  

<br/>

---


# Mathematical Principle

## Data set
$$X_{i} = [x_{1},x_{2},x_{3} ... x_{p}] $$ , $$ Y_{i}$$  
> $$X_{i}$$: 데이터 포인트  
> $$[x_{1},x_{2} .. x_{p}]$$ Feautre 들을 의미하고 p는 Feature의 갯수   

> $$Y_{i}$$: 데이터 포인트에 해당되는 Label    
> 수치형 문제: 예상 값  
> 범주형 문제: 예상되는 범주(Category).    $$y_{i} \in [1,2,3... k]$$

<br/>

## Model equation

### - Regression Tree

<!-- $$ f(X) = \sum_{m=1}^{M} a_{m} I (x \in R_{m})$$ -->

$$ f(X) = \sum_{m=1}^{M} c_{m} I (x \in R_{m})$$

> $$f_(x)$$: X에 대한 예측 값  
> $$R_{m}$$: 분류된 데이터의 부분 집합.  각 **Ternimal node**에는 분류된 부분 집합 존재.  
> $$I (x \in R_{m})$$: Indicator Function. 괄호 안에 내용이 True면 1, False면 0을 반환  

**Terminal node에서 분류된 데이터 부분 집합 $$R_{m}$$에 대해서 데이터 X가 부분집합 $$R_{m}$$ 에 속하면 $$c_{m}$$을 리턴, 아니라면 0 을 리턴.**


### - Classification Tree

$$f(X) = \sum_{m=1}^{M} k_{m} I (x \in R_{m})$$  

$$k(m) = \underset{k}{argmax} (\overset{-}{p_{mk}}) \quad \quad \overset{-}{p_{mk}} = \frac{1}{N_{m}} \ sum_{x_{i} \in R_{m}} I(y_{i} = k)$$

> $$k_{m}$$: $$R_{m}$$ 에 대해 $$\overset{-}{p_{mk}}$$ 가 가장 큰 값을 가지는 k 값  
> $$\overset{-}{p_{mk}}$$: Terminal node에 속하는 데이터 집합 $$R_{m}$$ 에서 $$y_{i} = k$$ 를 만족하는 데이터의 비율.  

**Terminal node에서 분류된 데이터 부분집합 $$R_{m}$$에서 가장 많은 비율을 가지는 class K 를 리턴해주는 함수.**

<br/>

## Cost function

### - Regression Tree

$$y_{i}$$가 Numerical value로 지정이 된다.  

$$\min_{c_{m}} \sum_{i=1}^{N}  (y_{i} - f(x_{i}))^2 \quad = \quad  \min_{c_{m}} \sum_{i=1}^{N} (y_{i} - \sum_{m=1}^{M} c_{m} I (x \in R_{m}) )^2$$  

<br/>

**$$c_{m}$$은 각 부분집합에($$R_{m}$$) 속해 있는 y값들의 평균으로 예측할 때 cost function 이 최소가 된다**   

**따라서 Regression Tree의 cost function을 최소화 시키기 위한 최적의 $$c_{m}$$ 은 다음과 같다**

$$\overset{-}{c_{m}} = ave(y_{i}|x_{i} \in R_{m})$$

<br/>

### - Classification Tree

$$y_{i}$$가 Discretized value로 지정이 된다.  $$y_{i} \in [1,2,3... k]$$

분류 문제에서는 $$Y_{i}$$ 값이 Discretized 되어 있기 때문에 MSE와 같은 직관적인 함수를 사용하지 못한다. 사용되는 **cost function**은 다음과 같다. 

- Misclassification rate  

$$ \frac{1}{N_{m}} \sum_{i \subset R_{m}} I(y_{i} \neq k(m)) = 1 -  \overset{-}{p_{mk}}_{m}$$

- **Gini Index**  

$$ \sum_{k \neq k^{'}} \overset{-}{p_{mk}} \overset{-}{p_{mk^{'}}} = \sum_{k=1}^K \overset{-}{p_{mk}} (1 -\overset{-}{p_{mk}} )$$

- **Cross-Entropy**

$$ -\sum_{k=1}^{K} \overset{-}{p_{mk}} log \overset{-}{p_{mk}}$$

위의 **cost function**들은 데이터 분류에 대한 **불순도**(impurity) 를 측정하는 방법들이다.  
내 개인적인 경험으로는 Gini index와 Cross-Entropy를 많이 쓴다.

<br/>

## 분할변수와 분할점.

그렇다면 어떤 변수를 어떤 값을 기준으로 Branch를 치는게 좋을까?  
또한 어떤 변수를 분할하는데 먼저 사용할지, 순서도 중요한 문제가 될 것이다.  
여기서 말하는 어떤 변수는 **분할 변수 $$x_{j}$$** 이고 어떤 값은 **분할 점 `s`** 이다.

만약 2개의 집합 즉, 2개의 terminal node만 있다고 하자.  
각 터미널 노드에는 해당되는 부분 집합 $$R_{1}, R_{2}$$ 가 존재 한다.  

$$R_{1}(j,s) = [x|x_{j} \leq s] \quad  R_{2}(j,s) = [x|x_{j} \geq s]$$

이때, $$x_{i}$$에 해당되는 $$y_{i}$$의 값은 Regression일 경우 Numeric value, Classification일 경우  Discretized value이다. 

terminal node위 Intermediate node에서는 최적의 **분할 변수**와 **분할 점**을 찾기 위해 트리 종류에 따라 다음과 같은 연산을 한다. 

### - Regression Tree

$$ argmin_{j,s} [ \sum_{x_{i} \in R_{1}(j,s)} (y_{i} - \overset{-}{c_{1}})^2  +  \sum_{x_{i} \in R_{2}(j,s)} (y_{i} - \overset{-}{c_{2}})^2]$$

$$\overset{-}{c_{1}} = ave(y_{i}|x_{i} \in R_{1}) \quad \overset{-}{c_{2}} = ave(y_{i}|x_{i} \in R_{2})$$

위 수식을 해석하자면  
- 3번째 수식: j,s 를 **grid search** 를 통해 모든 값을 바꿔가면서 정답의 부분 집합 $$R_{m}$$을 계산한다.  
- 2번째 수식: $$R_{m}$$ 이 바뀌면서 그 부분집합에 속하는 데이터의 Label의 평균 값이 바뀜 따라서 최적의 $$c_{m}$$ 즉, $$\overset{-}{c_{m}}$$ 도 값이 바뀔것이다.
- 1번째 수식: 바뀐 $$R_{m}$$에 속해있는 데이터의 label $$y_{i} (y_{i} \in R_{m})$$ 값과,    label $$y_{i} (y_{i} \in R_{m})$$ 전체의 평균값인 $$\overset{-}{c_{m}}$$ 과의 차이를 계산 해보았을때, 최소가 되는 j,s 를 return 한다.

<br/>

### - Classification Tree

- Cross-Entropy Cost function 

$$argmin_{j,s}[-\sum_{k=1}^{K}  \overset{-}{p_{mk}} log  \overset{-}{p_{mk}} ]$$

내가 물리학 전공할때 배운 엔트로피는 두가지 정도로 정의를 했었다.  
통계역학: $$S = k_{B} * ln \Omega$$  
열역학: $$dS = \frac{dQ_{rev}}{T}$$   

지금 생각해보면 여기서 나오는 엔트로피의 개념은 통계역학에서 배운 엔트로피 개념과 매우 흡사하다.  통계역학에서 나온 $$ ln \Omega$$는 어떤 계 가 가질수 있는 모든 앙상블(상태) 의 수를 의미한다. 계가 취할 수 있는 상태의 수가 적으면 엔트로피는 낮게 계산이 된다. 

위의 개념을 생각하면서 현재 엔트로피 개념으로 돌아와보자.   
Cross-Entropy Cost function은 Entropy를 최대한 낮게 가져가도록 수식이 쓰여져 있고, 낮게 가져간다는건 가질수 있는 모든 상태의 수가 적은 계, 즉 분류가 잘되어 가능한 상태의 수가 적은 쪽으로 계산한다는 것이다. 

수 많은 분할변수 중 가장 분류가 잘되는 변수와 분할 점을 선택해 entropy를 낮추는게 핵심 개념이다. 


정리하다가 문득 깨달았는데 이와같은 Decision Tree는 찾은 값이 Local optimal로 빠질 확률이 매우 높다는 생각이 든다.

~~덕분에 몇년만에 물리책 꺼냄~~


<br/>




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
