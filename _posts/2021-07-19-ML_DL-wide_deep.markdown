---
layout: post
title:  "[ML Paper] Wide & Deep Learning for Recommender Systems"
subtitle:   "Recommendation"
categories: ml_dl
tags: ml_paper
comments: False
---

|![Title](https://swha0105.github.io/assets/ml/img/wide_title.png)  
|:--:| 
| [논문 링크](https://arxiv.org/abs/1606.07792) |  


추천시스템에서 딥러닝의 첫번째 주요 혁신 중 하나라고 소개된 논문이다.  
지금까지 알아본 결과, 이 논문을 기점으로 추천 딥러닝 후속 논문이 나오고 이 논문을 citation을 하고 있다.  
이 논문에서 사용된 시스템은 구글플레이 앱스토어에 앱 추천에 사용되고 있으며 추천하는데 있어 앱에 따라 각각 다른 feature들을 고려하여 linear and non-linear하게 분석하여 추천한다.  
추천 딥러닝을 조사하기 위해 첫번째로 이 논문을 선정하였다. 

---

<br/>


# Abstract 

**1. Linear model + Nonlinear feature transformation (`Wide`)**
  - **`Wide`** & sparse data set에 적용하여 Nonlinear feature를 찾기 위한 `Cross-product feature transformation`(아래에 기술된 wide & deep learning - wide component에 설명)은 feature간의 interaction을 `Memorization` 하는데 효과적이였다.
  - 하지만, `Cross-product feature transformation`을 통해 feature간 모든 interaction을 generalization 하는데는 feature engineering (manual)이 필요하다.

**2. Deep Neural Network (`Deep`)**
  - **`Deep`** Neural network는 `low-dimensional dense embeddings`을 통해 unseen feature combinations을 구할 수 있다.
  - 하지만, data가 sparse & high-rank일 경우 over-generalization 현상이 발생 (모든 feature interaction을 0으로 예측.)

따라서 이 논문에서는 **Wide(`Memorization`, `Linear`)** & **Deep(`Generalization`, `Nonlinear`)**의 각각의 장점을 뽑아 추천 시스템 구축하여 구글 플레이 스토어 앱 추천 등, 많은 상용화된 어플에 적용함  
  
이 논문에 나오는 예제는 google play store의 앱 추천을 기반으로 작성되어있음.

<br/>

---

# Intro

### Chanllenge: 

기존의 추천 모델은 `Memorization`와 `Generalization`을 동시에 만족시키기가 어려웠다. 

> **Memorization**: 과거의 데이터에서 item이나 feature끼리의 상호작용
- user가 기존에 상호작용했던 item과 비슷한 상품을 추천해주는 경향  

> **Generalization**: 과거에 일어나지 않은 새로운 correlation을 기반 
- 추천 상품의 다양성을 높이는 경향.


<br/>

### 선행연구

##### 1. Cross-product transformation (For linear model)

- Feature n1과 Feature n2의 연관성을 Manual하게 설정한다.   
(ex, installed_app_1 = netfilx, installed_app_2 = youtube,   
if installed_app_1 & installed_app_2 then return 1 )
- Feature가 의미하는바를 넓은 범위로 설정하면 어느정도 generalization도 구현이 되지만 manual feature engineering이 요구되는 점에서 한계가 명확하다.  
(ex, installed_app = videos)
- 또한, **관측되지 않았던 query-item feature pair는 고려하지 못한다.**

##### 2. Embedding-based model(For FM, DNN)

- query와 item을 각각 low-dimensional dense vector(embedding vector)로 변환, 연산하여 query-item pair의 **unseen feature**를 포착함.
- 하지만 query-item pair 대부분이 0인 데이터에서 dense vector는 모든 query-item pair를 0으로 예측한다. (over-generalization)
- 따라서, query-item matrix가 sparse하고 high-rank일 경우 (user가 specific한 prefence를 가지는 경우) 학습하기가 어려웠다.   


이 논문에서 소개하는 wide-deep model은 위와 같은 문제를 linear model과 joint learning을 통해 각각의 장점을 함께 구현하고자 하였다.

|![Fig 1](https://swha0105.github.io/assets/ml/img/wide_fig1.png)  
|:--:| 
| Fig 1|  
  

<br/>

---

# Recommender system overview

|![Fig 2](https://swha0105.github.io/assets/ml/img/wide_fig2.png)    
|:--:| 
| Fig 2|  
  

> **query**: user와 contextual feature를 담고있음. user가 app을 사용할때 만들어짐.  
> **retrieval**: query를 ML model or human-defined rule로 database안에서 query에 대응하는 data candidate pool을 줄임  
> **ranking system**: candidate로 뽑힌 data들에 **scores**를 계산. (ranking)
   - score: P(y $$\lvert$$ X) probability of a user action label y given the features X (X is including user,contextual,impression feature)

**모델이 학습하는건 Ranking system의 score P(y $$\lvert$$ X)**

<br/>

---

# WIDE & DEEP LEARNING

### 1. Wide component

- Wide component는 `Raw feature`와 `Transformed feature`를 포함

**For raw feature**: $$ y = w^{T}x + b $$  
**For transformed feature** (Cross-product Transformation): $$ \phi_{k}(X) = \Pi_{i=1}^{d} x_{i}^{c_{ki}} \quad  c_{ki} \subset {0, 1} $$   
Binary feature들끼리 상호작용을 캡쳐하고, Non-linear term을 추가하는 효과.

> x: feature (length d)  
> $$\phi_{k}$$: manual하게 설정한 k번째 feature들의 조합 (~~논문에 설명이 안되어있다~~)  
> $$c_{ki}$$: i-th feature가 $$\phi_{k}$$와 연관이 있을때 1, otherwise 0 (manual 하게 설정)


### 2. Deep component
`feed-forward` 기반의 neural network

- **high dimensional catgorical feature**를 low-dimensional and dense real-valued vector (embedding vector)로 변환. 

$$ a^{l+1} = f(W^{l} a^{l} + b^{l}) $$

> $$a^{l}$$: l-th activations  
> $$b^{l}$$: l-th bias  
> $$W^{l}$$: l-th model weight  


### 3. Joint Traning of Wide & deep Model

<details>
<summary> Ensemble vs Joint Traning </summary>
<div markdown="1">   
    
앙상블 모델은 하나의 앙상블안에 여러개의 모델이 존재하고 inference할때 model들의 prediction을 취합할 뿐, 학습과정에서는 모델끼리의 연관성은 전혀 없다. 따라서, 개별 모델이 어느정도 성능이 나와야함으로 모델 크기가 대체로 크다.

Joint training도 복수의 모델이 존재하는데, 학습과정에서 parameter들을 동시에 최적화 한다. 
이와 반대로, 하나의 모델의 다른 모델의 약점을 채워주는 구조라 모델 사이즈가 ensemble에 비해 작다.


</div>
</details>

- **`Wide model`** 과 **`Deep model`**에서 나온 back-propagating gradient를 simultanesoly 사용 하는게 목적
- Optimizer로 각각 Deep part - **AdaGrad**,  Wide part - **L1 regularization** 을 사용하는 **`Follow-The-Regularized-Leader` (FTRL)** 알고리즘 구현


**Follow-The-Regularized-Leader(FTRL):**

$$P(Y=1 \lvert X) = \sigma(w^{T}_{wide}(x,\phi(x)) + W^{T}_{deep} a^{l_{f}} + b)$$

> Y: binary class label  
> X: original features  
> $$\sigma$$: sigmoid function  
> $$\phi(X)$$: X에 대한 cross product transformation  
> b: bias   
> $$a_{l_{f}}$$: final activations of deep model  
> $$w_{wide}$$: wide model의 weight vector  
> $$w_{deep}$$: deep model의 weight vector 

- feed-forward과정 중에는 wide와 deep모델 각각 따로 학습 되지만, 마지막 activation 과정에서 activation function에 wide와 deep모델의 정보가 함께 들어가며 backpropagation에 사용되는 gradient를 구한다.

<br/>

---

# System Implementation

app을 추천하는 pipleline은 3가지로 구성된다:  
**Data generation**, **Model tranining**, **Model serving**

|![Fig 3](https://swha0105.github.io/assets/ml/img/wide_fig3.png)    
|:--:| 
| Fig 3|  
  
### 1. Data generation
- 한 주기(period)내, User와 app impression data를 이용해 training data 생성  
(app impression data: app age, historical statistics of an app)

**Categorical Feature:**
- 최소 발생 횟수가 넘은 categorical feature에 대해 Vocabularies 생성  
(Vocabularies: categorical feature string을 ID로 mapping한 table)

**Continous real-valued Feature:**
- feature value x에 대해 cumulative distribution function P(X $$\leq$$ x)을 구성 
- P(X $$\leq$$ x )을  quantile ($$n_{q}$$)로 나누어, normalized value = $$\frac{i-1}{n_{q}-1}$$ ([0 - 1]의 값을 가짐) 을 구성. 


### 2. Model training

|![Fig 3](https://swha0105.github.io/assets/ml/img/wide_fig4.png)    
|:--:| 
| Fig 4|  

- Input layer는 traning data와 vocabularies를 받아 `sparse`, `dense` feature들을 생성

- Wide component는 유저가 설치한 app의 cross-product transformation와 impression data으로 구성

- Deep component는 각각의 categorical feature를 학습하기 위해 32-dimensional embedding vector로 구성 및 학습

- 모든 dense feature들을 concatenate를 하여 dense vector구성.  (~1200 dimension)

- dens vector를 input으로, 3개의 ReLU layers 구성.

- 예전 모델의 가중치를 이용한 학습, `warm start`가 가능한 모델. (데이터가 업데이트 할때마다 새롭게 학습하는건 현실적으로 불가능.)


### 3. Model serving

- app retrieval system과 user feature들로 부터 app candidates를 받아 score를 매긴다.
- 이후 app은 highest -> lowest score로 정렬되며 유저에게 보여진다.
- 이 과정은 작은 batch를 multithreading 병렬화 하여 10ms 과정에 끝난다.

<br/>

---

# Experiment results
추천시스템의 효용성을 구하기 위해 `app acquisitions`과 `serving performance`를 계산하였다.

### 5.1 App acquisitions
3주간 A/B test 시행.

![table 1](https://swha0105.github.io/assets/ml/img/wide_table1.png)   


- 각 모델 linear(wide-only logistic regression) model, deep model(deep model), wide-deep model에 실험군으로 유저의 1%씩 randomly select하여 배정.

- 표와 같이, wide-deep model에 대한 app acquisition이 3.9% 높고 이는 통계적으로 유효하다


### 5.2 Serving Performance

![table 2](https://swha0105.github.io/assets/ml/img/wide_table2.png)   


- peak traffic일때, 실험에서 사용한 recommender server는 초당 1천만개의 score을 계산
- single threading일때 single batch를 이용해 모든 candidate에 scoring하는건 31ms가 걸렸음
- multithreading일때, batch를 작게 만들어 계산할 경우 14ms가 걸렸음.

<br/>

---

# Related work 

- linear model (with cross product feature transformation)과 deep neural network를 함께 사용하는 아이디어는 `Factorization Machine (FM)`논문 ([링크](https://swha0105.github.io/assets/ml/paper/Factorization_Machines.pdf)  ) 에서 가져옴

- FM과 다르게, model capacity를 Deep learning기반의 embedding vector의 highly nonlinear interaction을 이용해 향상시킴.

- joint training은 RNN에서 complexity를 줄이기 위해 사용했던 방법.  

# Conclusion

- Wide linear model은 sparse feature를 cross-product feature transformation을 통해 **`Memorize`**을 하는데 강점이 있다.
- Deep neural network는 low dimensional embedding을 통해 unseen feature를 **`generalize`** 하는데 강점이 있다.

- 제시한 `Wide-Deep learning framework`은 두가지 타입의 강점을 결합하였고 실제 데이터에서 좋은 모습을 보였다. 


<br/>

---


### Rerfence 

1. [https://towardsdatascience.com/modern-recommender-systems-a0c727609aa8]
2. https://soobarkbar.tistory.com/131

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
