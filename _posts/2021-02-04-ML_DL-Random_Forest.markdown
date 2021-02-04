---
layout: post
title:  "[ML Algorithms] Decision Tree & Random Forest"
subtitle:   "supervised Learning"
categories: ml_dl
tags: ml
comments: true
---
# 들어가기 전 

Recision Tree와 Random forest에 대해 알고 있는것들을 구체적으로 정리하기 위해 포스팅을 한다.  
Review paper나 책을 찾아보았지만 깔끔하게 정리된것을 못찾아서 발품(?)팔아서 여기저기 찾으며 혼자 정리 해보았다.  
주로 [김성범 교수님 강의를](https://www.youtube.com/watch?v=xki7zQDf74I) 참고하였다.

- Decision Tree
- Random Forest

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

[decision tree](https://swha0105.github.io/assets/ml/img/DecisionTree.JPG)  

<br/>

## Mathematical Principle

### Data set
$$X_{i} = [x_{1},x_{2},x_{3} ... x_{p}] $$ , $$ Y_{i}$$  
> $$X_{i}$$: 데이터 포인트  
> $$Y_{i}$$: 데이터 포인트에 해당되는 Label   
> $$[x_{1},x_{2} .. x_{p}]$$ Feautre 들을 의미하고 p는 Feature의 갯수 


$$ f(X) = \sum_{m=1}^{M} c_{m} I (x \in R_{m})$$

> $$f_(x)$$: X에 대한 예측 값.  
> $$R_{m}$$: 분류된 데이터의 부분 집합.  각 **Ternimal node**에는 분류된 부분 집합 존재.  
> $$I (x \in R_{m})$$: Indicator Function. 괄호 안에 내용이 True면 1, False면 0을 반환  

**데이터 X 의 Feature x 들이 부분집합 $$R_{m}$$ 에 속하면 Cost function인 $$c_{m}$$을 리턴, 아니라면 0 을 리턴.**

### Cost function

$$\min_{c_{m}} \sum_{i=1}^{N}  (y_{i} - f(x_{i}))^2 $$   

$$= \min_{c_{m}} \sum_{i=1}^{N} (y_{i} - \sum_{m=1}^{M} c_{m} I (x \in R_{m}) )^2$$  

<br/>

**이때, $$c_{m}$$은 각 부분집합에($$R_{m}$$) 속해 있는 y값들의 평균으로 예측할 때 cost function 이 최소가 된다**   
따라서 **최적의 $$c_{m}$$** 은 다음과 같다

$$\overset{-}{c_{m}} = ave(y_{i}|x_{i} \in R_{m})$$

### 분할변수와 분할점.

그렇다면 어떤 변수를 어떤 값을 기준으로 Branch를 치는게 좋을까?  
또한 어떤 변수를 분할하는데 먼저 사용할지, 순서도 중요한 문제가 될 것이다.  
여기서 말하는 어떤 변수는 **분할 변수 $$x_{j}$$** 이고 어떤 값은 **분할 점 `s`** 이다.

만약 2개의 집합 즉, 2개의 terminal node만 있다고 하자.  
각 터미널 노드에는 해당되는 부분 집합 $$R_{1}, R_{2}$$ 가 존재 한다.  

terminal node위 Intermediate node에서는 최적의 **분할 변수**와 **분할 점**을 찾기 위해 다음과 같은 연산을 한다. 

$$ argmin_{j,s} [ \min_{c_{1}} \sum_{x_{i} \in R_{1}(j,s)} (y_{i} - c_{1})^2  + \min_{c_{2}} \sum_{x_{i} \in R_{2}(j,s)} (y_{i} - c_{2})^2]$$

이 함수를 최소화 하는 j,s의 값을 찾는 것이다.  
우리는 앞선 수식을 통해 $$c_{m}$$ 의 최적값을 알아 냈고 이를 적용하면 다음과 같다.  

$$ argmin_{j,s} [ \sum_{x_{i} \in R_{1}(j,s)} (y_{i} - \overset{-}{c_{1}})^2  +  \sum_{x_{i} \in R_{2}(j,s)} (y_{i} - \overset{-}{c_{2}})^2]$$

$$\overset{-}{c_{1}} = ave(y_{i}|x_{i} \in R_{1}) \quad \overset{-}{c_{2}} = ave(y_{i}|x_{i} \in R_{2})$$

$$R_{1}(j,s) = [x|x_{j} \leq s] \quad  R_{2}(j,s) = [x|x_{j} \geq s]$$

위 수식을 해석하자면  
- 3번째 수식: j,s 를 **grid search** 를 통해 모든 값을 바꿔가면서 정답의 부분 집합 $$R_{m}$$을 계산한다.  
- 2번째 수식: $$R_{m}$$ 이 바뀌면서 그 부분집합에 속하는 데이터의 Label의 평균 값이 바뀜 따라서 최적의 $$c_{m}$$ 즉, $$\overset{-}{c_{m}}$$ 도 값이 바뀔것이다.
- 1번째 수식: 바뀐 $$R_{m}$$에 속해있는 데이터의 label $$y_{i} (y_{i} \in R_{m})$$ 값과,    label $$y_{i} (y_{i} \in R_{m})$$ 전체의 평균값인 $$\overset{-}{c_{m}}$$ 과의 차이를 계산 해보았을때, 최소가 되는 j,s 를 return 한다.


<br/>


---


## Reference



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
