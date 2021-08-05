---
layout: post
title:  "[ML Paper] Modeling User Exposure in Recommendation"
subtitle:   "Recommendation"
categories: ml_dl
tags: ml_paper
comments: False
---

|![Title](https://swha0105.github.io/assets/ml/img/expose_MF.png)  
|:--:| 
| [논문 링크](https://github.com/swha0105/swha0105.github.io/blob/gh-pages/assets/ml/paper/Modeling User Exposure in Recommendation.pdf) |  

<br/>

---

# Abtract

- `Collaborative filtering`은 비슷한 유저들의 선호를 바탕으로 아이템을 추천해주는 기법으로 널리 사용된다. 

- Collaborative filetering **`Implicit Feedback setting`**에서는 user와 모든 item의 관계를 고려한다. 하지만 이는 user와 전혀 관련없거나 인지하지 못한 item도 고려하기에 상식에 맞지 않다.

- **`인과 분석 (Causal anaylsis)`**에서는 Assignment mechanism(특정 item이 user에 노출되는 것) 은 user/item의 조합에 따라 바뀔 수 있는 **`Latent variable`**이라 본다.

- 본 논문에서는 Item이 User에 노출되는 것을(User exposure) collaborative filetering에 직접적으로 고려하는 새로운 확률적 접근을 제시한다.

<!-- - User exposure은 latent variable로 모델이 데이터로 부터 추론하는 값이다. -->

<br/>

---

# Introduction

- 전통적으로, 추천 시스템은 두가지 모드가 존재한다. **`Explicit data`**, **`Implicit data`** 

- `Explicit data`는 유저가 직접 item에 대해 평가를 내린 데이터로 이를 이용해 missing data를 예측하는 시스템을 구축한다. 하지만 보통 explicit data는 구하기가 까다롭다.

- `Implicit data`는 실제 사용에 좀 더 많이 사용된다. 클릭이나, 과거 구매여부로 간접적으로 유저의 성향을 담고 있는 데이터를 의미한다.
   - Implicit data는 다음과 같은 한계점을 가진다.
    1. 유저의 성향을 binary로 추론한다 (클릭여부, 구매여부, 기타등등..)
    2. 기존에 많은 implicit method들이 사용하는 unclicked item을 선호하지 않는 아이템이라 가정은 large-sacle setting에서 불합리하다. (단지, item이 user에 expouse되지 않았을 뿐인데 선호하지 않는다고 가정.)


- 현존하는 모델은 uncliecked item에 대한 weight를 낮추는 방향으로 접근한다.
   - 아래의 방법은 효과적이였지만 data에 대한 heuritsic alteration이 필요하다.
    1. unclicked item에 Gasussian random variable을 통해 lower confiendce를 부여. (Hu et al, 2008)
    2. unclicked item의 subsample rate를 낮추는 방향 (Rendle et al, 2015)


- 본 논문은 이러한 문제들을 **`user에 해당 item이 expose되었는지`, `user가 해당 아이템을 클릭하기로 결정했는지` 각각 따로 캡쳐하는 `ExpoMF`를 소개한다**

- ExpoMF는 유저 선호도와 item에 대한 노츨에 대해 iteration하여 자연스럽게 unclicked item에 대한 weight를 낮춘다. (expose가 적게된 unclicked item일수록 weight 낮춤)

- Item과 User의 additional information (날짜, 위치.. 기타 등등)으로 exposure을 조절한다. 

<br/>

# Background

**1. Matrix Factorization for Collaborative filetering**

- User-item의 선호도를 나타낸 Implicit(Explicit) 데이터는 user-item matrix로 **`Encoding`**될 수 있고 본 논문에서는 `click matrix` 또는 `consumption matrix`라 부른다

- Matrix Factorization model은 click-matrix factorization을 통해 user의 해당 item 선호도를 추론 한다.

- Generative modeling의 시각으로는, 이는 user와 item의 latent factor를 내적한 값을 parameter로 사용하는 specific distribution(ex: possion or gaussian)을 추론하는 것이다.


$$ \theta_{u} \sim N(0, \lambda_{\theta}^{-1}, I_{K})$$  
$$ \beta_{i} \sim N(0, \lambda_{\theta}^{-1}, I_{K})$$  
$$ y_{ui} \sim N(\theta_{u}^{T}\beta_{i}, \lambda_{y}^{-1})$$

> $$\theta_{u}$$: user u의 잠재 선호도 (latent prefences)  
> $$\beta_{i}$$: item i의 잠재 선호도
> $$\lambda$$: hyperparameter  
> $$I_{k}$$: identity matrix of dimension K 


**2. Collaborative filtering for implicit data**

- `Weighted Matrix Factorization` (WMF, 또는 one-class collaborative filtering)은 implicit data를 사용하는 factorization model의 표준이며, 선택적으로 click matrix의 unobserved item의 weight를 낮춰준다.

$$ y_{ui} \sim N(\theta_{u}^{T} \beta_{i}, c_{y_{ui}}^{-1})$$

> c: confidence $$(c_{1} \gt c_{0})$$ 
>   - $$c_{1},c_{0}$$에 대한 설명은 언급되지 않는다. 아마 c1은 observed item, c0은 unobserved item이라 생각 된다.


- WMF는 binarized click matrix를 사용하기에 regression problem에 약했다.
- Bayesian Personalized Ranking (BPR)은 unobserved data를 observed data에 비해 낮은 순위를 rank하는 접근을 취한다.

<br/>

# Exposure Matrix Factorization

## Model Description

|![ExpoMF schematic](https://swha0105.github.io/assets/ml/img/exposeMF_fig1.png)  
|:--:| 
|  |  

$$ \theta_{u} \sim N(0,\lambda_{\theta}^{-1},I_{K})$$  
$$ \beta_{i} \sim N(0,\lambda_{\beta}^{-1},I_{K})$$  
$$ a_{ui} \sim Bernoulli(\mu_{ui})$$  
$$ y_{ui} \lvert a_{ui} = 1 \sim B(\theta_{u}^{T}\beta_{i},\lambda_{y}^{-1})$$  
$$ y_{ui} \lvert a_{ui} = 0 \sim \delta_{0} \quad (P(y_{ui} = 0 \lvert a_{ui} = 0) \rightarrow \delta_{0} = 1 )$$

> u: user (u $$\in$$ 1,2,...U)  
> i: item (i $$\in$$ 1,2,...I)  
> $$\theta_{u}$$: user u의 preference  
> $$\beta_{u}$$: item i의 attribute  
> $$a_{ui}$$: user u가 item i에 expose 되었는지 나타내는 값  ($$a_{ui} \in A$$)   
> $$y_{ui}$$: user u가 item i에 click 하였는지 나타내는 값  ($$y_{ui} \in Y$$)  
> $$\mu_{ui}$$: prior probability of exposure  
> $$\lambda$$: hyperparamter

- click expose값은 bernoulli distribution을 따른다.

# 작성중 
We observe the complete click matrix 부터..



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
