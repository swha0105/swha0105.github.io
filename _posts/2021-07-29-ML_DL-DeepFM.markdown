---
layout: post
title:  "[ML Paper] DeepFM"
subtitle:   "Recommendation"
categories: ml_dl
tags: ml_paper
comments: true
---

|![Title](https://swha0105.github.io/assets/ml/img/DeepFM_title.png)  
|:--:| 
| [논문 링크](https://github.com/swha0105/swha0105.github.io/blob/gh-pages/assets/ml/paper/DeepFM.pdf) |  

<br/>

참조하는 [포스팅](https://towardsdatascience.com/modern-recommender-systems-a0c727609aa8)에서 DeepFM에 대해 정보를 얻었다. FM의 neural net version이라하고 Wide & Deep moidel의 발전된 버전이라고 한다. 딥러닝을 이용한 추천시스템 문제에 첫 번째 주요 돌파구였다고 한다. 

<br/>

---

# Abstract

- 복잡한 feature interaction을 학습하는건 CTR recommender system의 핵심이다.
- 그럼에도 불구하고, 기존의 있는 방법들은 low(or high)-order interaction의 strong bias를 가지고 있거나 전문적인 feature extraction이 필요로 했다.
- 본 논문의 주제인 `DeepFM`은 FM의 장점과 Deep learning의 장점을 결합하였다.
-  feature engineering이 필요한 기존의 모델 [Wide & Deep model](https://swha0105.github.io/ml_dl/2021/07/19/ML_DL-wide_deep/)과는 달리 DeepFM은 raw feature 처리외에 필요하지 않다.

<br/> 

---

# Introduction

- **Recommender system의 목적은 `CTR (Click-through rate)`을 최대화 하는것**이고 이를 통해, Item들을 유저에게 계산된 CTR로 ranking을 매길 수 있다. 

- CTR prediction 문제는 user click behavior에 숨겨져 있는 `implicit feature interaction`을 학습하는것이 중요하다. (시간, 성별, 나이, ...)

- 기존의 Wide & Deep model로 알 수 있었던건 low와 high feature간의 interaction은 하나씩만 고려하였을때 보다 성능 향상을 가져왔다.

- CTR을 위한 recommender system 모델링의 가장 어려운점은 feature interaction에 관한것이다. 어떤 feature interaction은 필드 전문가에 의해 설계되지만 그렇지 않은 feature interaction (`priori`) 는 machine learning에 의해서만 캡쳐된다.

- FM은 feature간의 latent vector의 내적을 통해 매우 성공적인 결과를 냈지만 모델의 complexity때문에 order-2 feature interaction 이상은 현실적으로 하기 힘들었다.

- 딥러닝은 CTR prediction문제를 잘 다룰수 있는 포텐셜을 가지고 있다.  
  - CNN-based 모델은 neighboring feature들 간의 interaction에 대해 biased 되어있다.
  - 이에 반해 RNN-based ahepfdms sequential dependency가 있는 click data에 적합하였다. 
  - Factorization-machine supported Neural Network (FNN) 은 DNN에 적용하기 전에 FM을 미리 학습하여 FM의 capability를 제한한다. 
  - Product-based Neural Network (PNN)은 embedding layer와 fully-connected layer사이 product layer를 도입하여 feature interaction에 대해 이해했다.

- 이 논문에서는 **low or high order feature intercation에 대해 biased 되어있지 않고, feature engineering이 필요하지 않은 ene-to-end 모델**을 소개한다

- 메인 contribution은 다음과 같다.
  - FM과 DNN의 arcchitecture를 합쳐서 low-order interaction은 FM처럼, high-order interaction은 DNN처럼 한다. DeepFM은 Wide & Deep model처럼 feature engineering이 필요하지 않다.
  - wide와 deep part가 input과 embedding vector를 공유한다. 
  - benchmark와 commercial data에 적용하였다. 


|![Fig 1](https://swha0105.github.io/assets/ml/img/DeepFM_fig1.png)    
|:--:| 
| Fig1. Wide & Deep architecture of DeepFM|  

- wide and deep component가 input raw feature vector를 공유한다. 이를 통해 low and high order feature interaction을 동시에 가능하게 한다. 

<br/>

---

# Our Approach

- 데이터 구조는 다음과 같다
  - traning data $$(\chi,y)$$는 총 n개로 구성되어있다.
    > $$\chi$$: m-fields data. (user-item pair에 대한 정보가 기록)  
    > y: label indicating user click (y = 1 유저가 해당아이템 클릭)

- $$\chi$$에는 categorical field와 continuous field가 존재한다.
  - catergorical field는 ont-hot encondinig한다.
  - continuous field의 값은 그대로 사용하거나 discretization후 ont-hot encoding한다.


## DeepFM

- Figure 1과 같이, DeepFM은 **`FM component`**와 **`Deep Component`**로 나누어진다. 

- feature i에 대해 $$w_{i}$$는 order-1 feature interaction, latent vector $$V_{i}$$는 order-2 feature interaction을 나타낸다. $$V_{i}$$는 deep component의 input으로도 사용된다.

- 모든 parameter ($$w_{i},V_{i}, ...$$$)들은 jointly trained 된다. (wide & deep model 참조)

$$\hat{y} = sigmoid(y_{FM} + y_{DNN})$$


**FM component**

|![Fig 2](https://swha0105.github.io/assets/ml/img/DeepFM_fig2.png)    
|:--:| 
| Fig2. The architecture of FM|  

FM에 대한 자세한 설명은 [이전 포스팅](https://swha0105.github.io/ml_dl/2021/07/27/ML_DL-FM/)에 하였고 이와 똑같은 모델을 쓰는거 같아 넘어가겠다

**Deep component**

|![Fig 3](https://swha0105.github.io/assets/ml/img/DeepFM_fig3.png)    
|:--:| 
| Fig3. The architecture of DNN|  

- 기존에 이미지나 오디오처리를 위한 연속적이고 dense한 DL 모델과 달리, CTR 예측을 위한 DL model은 **highly sparse, super high-dimensional, catergorical-continuous mixed, grouped in fields**를 포함한 **raw feature를 input vector**로 받기 때문에 새로운 architecture 디자인이 필요로 했다

- 이를 위해, **`embedding layer`**는 input vector를 `low dimensional dense real-value vector`로 바꾸어 hidden layer로 전달한다.


|![Fig 4](https://swha0105.github.io/assets/ml/img/DeepFM_fig4.png)    
|:--:| 
| Fig4. The structure of the embedding layer|  

**Embeddindg layer**의 특징은 다음과 같다
- Input field vector의 길이는 다를 수 있지만 embedding은 모두 k-size로 당일하다 
- FM에서 나온 latent vector (V)는 weight로 사용된다.
  - 기존에 있던 모델들은 FM의 latent feature vector를 이용하여 network initialize 하였지만 DeepFM은 FM을 포함하여 학습하기 때문에 end-to-end 를 달성할 수 있었다. 


**Embedding layer의 output**은 다음과 같다.

$$ a^{0} = (e_{1},e_{2}, ..., e_{m}) $$
> $$e_{i}$$: i-th field에 대한 embedding   
> m: number of fields  

$$ a^{l+1} = \sigma(W^{l}a^{l} + b^{l})$$
> a^{l+1}: l-th layer output
> $$\sigma$$: activation function
> W^{l}: l-th layer weight
> b^{l}: l-th bias

최종적으로 Deep component의 prediction은 다음과 같다.

$$ y_{DNN} = \sigma(W^{ \lvert H \lvert + 1} \cdot a^{H} + b^{\lvert H \lvert+1})$$
> $$\lvert H \lvert$$: number of hidden layers

- FM component와 Deep component가 rkxdms feature embedding을 사용하는데 있어 장점은 다음과 같다
   - raw feature에서 low and high order feature interaction을 학습할 수 있다.
   - 전문가에 의한 feature engineering이 필요하지 않다.

<br/>

## Relationship with the other Neural Networks


|![Fig 5](https://swha0105.github.io/assets/ml/img/DeepFM_fig5.png)    
|:--:| 
| Fig5. FNN,PNN,Wide & Deep model architectures (for CTR Prediction)|  

**1. FNN** (Zhang et al., 2016)
- FNN은 neural network를 FM을 통해 initialized 시킨 모델이다
- FNN은 3가지 약점이 존재하는데
   - embedding parameter가 FM에 의해 과영향을 받는다.
   - pre-training stage가 존재함으로 efficiency가 감소한다.
   - high-order feature interaction 밖에 캡쳐하지 못한다.

- DeepFM은 pre-traning도 필요없으며 high-low order feature interaction이 가능하다.

**2. PNN** (Qu et al., 2016)
- high-order feature interaction을 캡쳐하기 위해 embedding layer와 first hidden layer사이에 product layer를 도입하였다.
- product operation에 따라 IPNN,OPNN,PNN*으로 나누어진다
   - IPNN: vector의 내적에 기반
   - OPNN: vector의 외적에 기반
   - PNN*: vector의 내,외적에 기반

   - 이러한 product operation을 효율적으로 하기위해 내적은 몇개의 뉴런을 삭제하고 계산하였으며 외적은 feature vector를 압축하였다.

- 하지만 외적은 계산결과가 unstable하였고, 내적은 product layer와 hidden layer가 fully connected에서 나오는 time complexity가 문제였다.

- 또한, low-order feature interaction은 무시되었다.


**3. Wide & Deep** (Cheng et al., 2016)

자세한 내용은 [이전 포스팅](https://swha0105.github.io/ml_dl/2021/07/19/ML_DL-wide_deep/)에 작성하였고 이와 비교하여 DeepFM의 장점만 기술하면 다음과 같다

- Wide & Deep model에서 Wide part에서 필요한 전문가에 의한 feature engineering이 필요하지 않다
- Wide & Deep model과 달리 feature를 두 모듈에서 동시에 학습하기 때문에 좀 더 정확한 학습이 가능하다.

<br/>

---

# Experiments


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
