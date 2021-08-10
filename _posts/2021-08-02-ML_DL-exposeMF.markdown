---
layout: post
title:  "[ML Paper] Modeling User Exposure in Recommendation"
subtitle:   "Recommendation"
categories: ml_dl
tags: ml_paper
comments: False
---

|![Title](https://swha0105.github.io/assets/ml/img/expose_MF_title.png)  
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
| left: ExpoMF  Right: ExpoMF with exposure covariates  |  

- User-item prefence data가 있다고 가정한다. (Click matrix)
- `matrix factorization`을 통해 Click matrix를 K-dimesion을 갖는 user preference($$u_{i,1:K}$$)와 Item attribute($$\beta_{u,1:K}$$)으로 나눈다 (i,u 인덱스는 오타인거 같지만 그대로 작성한다.)

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
> $$x_{i}$$: exposure covariate  
> $$\phi_{u}$$: exposure model parameter

- click expose값은 bernoulli distribution을 따른다.
- $$y_{ui} \gt 0 $$ 이면 $$a_{ui} = 1$$ 이지만 $$y_{ui} = 0 $$ 이면 $$a_{ui}$$ 값은 latent이다.  
   - Y matrix는 보통 sparse하기에, 대부분의 a값은 latent하다.

$$ \log p(a_{ui},y_{ui} \lvert \mu_{ui},\theta_{u},\beta_{i},\lambda_{y}^{-1}) = $$  
$$ \log Bernolli(a_{ui} \lvert \mu_{ui}) + a_{ui} \log N(y_{ui} \lvert \theta_{u}^{T}\beta_{i},\lambda_{y}^{-1}) + (1-a_{ui}) \log I(y_{ui} = 0) $$

> I: indicator function ($$y_{ui} = 0$$ 이 참이면 1을, 아니면 0을 return)  
> 만약 $$y_{ui} \neq 0$$이면, $$\log I(y_{ui})$$ 값은 $$-\inf$$가 될텐데 어찌된 영문인지 논문엔 위와 같이 정의했다.


- 만약, 유저가 특정 item에 대해 클릭하지 않음이 관측 되었다면 ($$y_{ui}=0$$)
   - 특정 user에게 특정 item에 대한 선호도가 높게 예측이 될 때 ($$\theta_{u}^{T}\beta_{i}$$가 높을때) 클릭하지 않을 likelihood는 낮다. ($$ \log N(0 \lvert \theta_{u}^{T} \beta_{i}, \lambda_{y}^{-1})$$) 이를 통해 해당 item이 노출되었을 확률을 줄여준다 ($$a_{ui}$$ 텀에 대한 factor 낮춰줌)
      - **$$ \log N(0 \lvert \theta_{u}^{T} \beta_{i}, \lambda_{y}^{-1})$$ 은 전체 click matrix Y에 대해 $$y_{ui}$$ = 0 인 데이터들만 고려하였을 때, 나타나는 parameter($$\theta_{u}^{T} \beta_{i}, \lambda_{y}^{-1}$$)의 분포 값을 parameter로 받는 gaussian distribution을 의미한다**
   - 낮은 $$a_{ui}$$값 $$\theta_{u} \beta_{i}$$의 evidence 값을 낮춰준다. 
  

- Exposure Matrix가 single value로 고정된다면, ExpoMF는 [Gaussian Probabilistic matrix factorization](http://www.cs.utoronto.ca/~amnih/papers/pmf.pdf) 가 된다. 

- Exposure Matrix가 $$c_{0}, c_{1}$$(confidence) 을 통해만 얻어진다면 WMF와 같다.

- $$\mu_{ui}$$와 $$\theta_{u}, \beta_{i}$$의 **`conditional independence`** 관계는 inference procedure (EM, variational inference, gibbs sampling)할 때 중요한 조건이 된다.

## Hierarchical Modeling of Exposure

- 노출 (exposure)에 대한 사전 확률 (prior probability of exposure, $$\mu_{ui}$$)을 결정하는 방법은 2가지가 있다
   - user,item factor와 clicks에 관한 모든 변수를 global value로 잡는 방법 (figure 2, left)
   - 특정 u,i에 대해 **`exposure covariate` $$x_{i}$$** 를 통해 $$\mu_{ui}$$ 설정 (Hierarchical modeling figure 1, right)

- Figure 1의 오른쪽 그림에서 보이듯이, Prior probability of Exposure ($$\mu_{ui}$$)를 `Exposure covariate` $$x_{i}$$를 통해 생성한다. 이때, `Exposure covariate` $$x_{ui}$$는 external information을 의미한다. (location, text topic ...) 


- Exposure covariate를 모델에 넣는 방법은 두가지가 존재한다.
  1. Per-item $$\mu_{i}$$
    - item popularity로만 $$\mu_{i}$$를 업데이트 하고 싶다면 $$\mu_{i} \sim Betadistribution(\alpha_{1},\alpha_{2})$$를 따른다. 이 모델은 hyperparameter $$\alpha$$에만 의지하는 모델이기에 external information를 사용하지 않는 모델이다.
  2. External information as Exposure covariates  
    $$\mu_{ui} = sigmoid(\phi_{u}^{T} x_{i})$$  
    - 만약, text topic을 exposure covariate로 쓴다고 가정하자. 
      > Exposure covariate($$x_{i}$$)는 NLP (word embedding, LDA)에서 나온 i번째 document의 representation.  
      > Exposure covariate($$x_{i}$$)의 크기는 L, Matrix factorization의 dimension(K)와 같을 필요없음  
      > Exposure covariate($$x_{i}$$)의 값은 모두 positive이며 normalized 되어있다.  
      > Model paramter($$\phi_{u})$$는 Logisitic regression의 coefficient로 해석 된다.   
      > Model paramter($$\phi_{u})$$는 또한, user u가 관심있어 하는 topic을 표현하는 변수로 해석된다. (user의 관심분야를 제한)  
    
  - 예를 들어, l-th의 topic의 neural network 관련이고 $$x_{il}$$이 높다고 하자, 그렇다면 i번째 user는 l번째 topic에 대해 관심이 높을 것이다. $$\phi_{ul}$$이 높다. 


<br/>

---

# Inference 

- posterior의 parameter를 추정하는 방법으로 **`EM`** 알고리즘을 사용한다. 
  - **Input:** Click matrix Y, exposure covariates $$x_{1:I}$$
  - **Random intialize:** $$\theta_{1:U}$$ (user factors), $$\beta_{1:I}$$ (item factors), $$\mu_{1:I}$$ (exposure priors), $$\phi_{1:U}$$ (exposure parameter)

  1. click하지 않은 item $$y_{ui} \gt 0 $$에 대해 exposure 기댓값을 계산한다. (E-step)
    - $$E(a_{ui} \lvert \theta_{u},\beta_{i},\mu_{ui},(y_{ui} = 0)) = \frac{\mu_{ui} \cdot N(0 \lvert \theta_{u}^{T}\beta_{i},\lambda_{y}^{-1})}{\mu_{ui} \cdot N(0 \lvert \theta_{u}^{T} \beta_{i}, \lambda_{y}^{-1}) + (1 - \mu_{ui})}$$
    
      > click한 item($$y_{ui} = 0$$에 대해서는 exposure 가 결정된다.$$a_{ui}=1$$

  2. $$\theta_{1:U}, \beta_{1:I}$$를 업데이트 한다. (M-step) 
    - $$\theta_{u} \leftarrow (\lambda_{y} \sum_{i} p_{ui} \beta_{i} \beta_{i}^{T} + \lambda_{\theta}I_{K})^{-1} (\sum_{i} \lambda_{y} p_{ui} y_{ui} \beta_{i})$$
    - $$\beta_{i} \leftarrow (\lambda_{y} \sum_{i} p_{ui} \theta_{u} \theta_{u}^{T} + \lambda_{\beta}I_{K})^{-1} (\sum_{i} \lambda_{y} p_{ui} y_{ui} \theta_{u})$$

    > $$p_{ui} = E(a_{ui} \lvert \theta_{u}, \beta_{i}, \mu_{ui}, \y_{ui} = 0)

```



```





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
