---
layout: post
title:  "[ML Lecture] Regularization 2, LASSO, Elastic Net"
subtitle:   "Machine learning"
categories: ml_dl
tags: ml_lecture
comments: true
---


# Least Absolute Shrinkage and Selection operator (LASSO)
- LASSO도 Ridge와 마찬가지로, MSE에 Regularization term 이 존재하는 형태이다.
- Shrinkage는 lambda 값이 커질수록 beta에 penalty가 붙어 줄어드는 현상을 의미한다. (Ridge와 동일)
- Selection은 LASSO의 특징 때문에 영향이 큰 데이터 feature들만 고려하는 현상을 의미한다. 
- **LASSO는 analytic form의 solution이 존재하지 않기 때문에 numeric 한 방법으로 계산한다.**

$$ \beta^{lasso} =  \underset{\beta}{argmin} \sum_{i}^{n} (y_{i} - x_{i}\beta)^2 \quad subject \; to \quad \sum_{j=1}^{p} \rvert \beta_{j} \rvert \leq t $$ 

$$ \beta^{lasso} =  \underset{\beta}{argmin} ( \sum_{i} (y_{i} - \ \sum_{j=0}^{p} (\beta_{j}x_{j}) )^2 + \lambda \sum_{j=0}^{p} \rvert \beta_{j} \rvert)$$  

위의 식을 보게된다면, Ridge regression과 매우 유사하다는걸 알 수 있다.  
다만, $$\beta$$에 패널티를 주는 부분이 **`L2-norm`이 아닌, `절대값 L1-norm`으로 바뀌었는데** 이 부분이 어떤 성질을 유도해내는지 알아보자. 


|![Lasso](https://swha0105.github.io/assets/ml/img/lasso.png)   
|:--:| 
|Lasso solution path|

<br/>

### Selection 원리

정규화 term의 $$\beta$$ 값을 그려보면 절대값의 형태로 되어있기 때문에 마름모 형태의 그림이 그려진다.   
이때, **최소 MSE coutour와 만나는 지점은 항상 `꼭짓점`이 되며** $$\beta$$ 값들 중 하나는 0이 된다.즉, 0이 된 $$\beta$$ 값에 해당하는 **데이터 feature는 완전 무시가** 된다.  

이러한 수학적 원리로 데이터 feature들 중 중요한 것들만 `selection` 하게 된다.


<br/>

### Ridge vs Lasso

|![Ridge_Lasso](https://swha0105.github.io/assets/ml/img/ridge_lasso.png)   
|:--:| 
|Solution path Ridge vs Lasso |

<br/>

|![Ridge_Lasso_graph](https://swha0105.github.io/assets/ml/img/ridge_lasso_graph.png)   
|:--:| 
|Beta vs tuning parameter (penalty)|

위의 그림은 **tuning parameter(penalty) 값이 작아질수록 (커질수록)**, 데이터 feature들의 coefficient **($$\beta$$)** 변화를 나타낸다.

당연하게도 `LASSO` 경우 penalty 값이 커질수록 중요하지 않은 데이터 feature들의 $$\beta$$ 값이 0에 빠르게 수렴하는것을 알 수 있다. 


|  | Ridge | Lasso |
|---|:---:|---:|
| 변수(feature)선택 유무 | X | O |
| Analytic solution | O | X |
| 좋은 예측성능 | 변수 간 상관관계 많을 때  | 변수 간 상관관계 적을 때 |


<br/>

---

# Elastic Net
- Ridge와 Lasso의 특성을 적절히 섞음 
- 변수 간 상관관계에 따라 penalty값이 조절


|![ElasticNet](https://swha0105.github.io/assets/ml/img/elasticnet.png)   
|:--:| 
|Beta vs tuning parameter (penalty)|


$$ \beta^{enet} =  \underset{\beta}{argmin} \sum_{i}^{n} (y_{i} - x_{i}\beta)^2 \quad subject \; to \quad s_{1} \sum_{j=1}^{p} \rvert \beta_{j} \rvert  + s_{2} \sum_{j=1}^{p} \beta_{j}^2  \leq t $$ 

$$ \beta^{lasso} =  \underset{\beta}{argmin} ( \sum_{i} (y_{i} - \ \sum_{j=0}^{p} (\beta_{j}x_{j}) )^2 + \lambda_{1} \sum_{j=0}^{p} \rvert \beta_{j} \rvert + \lambda_{2} \sum_{j=0}^{p} \beta_{j}^2)$$  

Elasticnet의 $$\beta$$ 추정 식은 Ridge의 L2, Lasso의 L1 term을 모두 넣은 형태이고 **이 식은 다음 수식을 내포한다.**  

$$\rvert \hat{\beta_{i}^{enet}} - \hat{\beta_{j}^{enet}}$$ $$\rvert \leq \frac{\sum_{i}^n \rvert y_{i} \rvert}{\lambda_{2}} \sqrt{(2(1-\rho_{ij}))}  $$ (zou and hastie, 2005)

>$$\rho_{ij}: X_{i}, X_{j}$$ 의 상관계수

따라서, $$\rho_{ij}$$ 크거나, $$\lambda_{2}$$가 크면, $$\rvert \hat{\beta_{i}^{enet}} - \hat{\beta_{j}^{enet}} \rvert$$ 가 작아든다, 즉, 두 변수의 상관관계가 강하거나, penalty를 강하게 주면 $$\beta_{i}, \beta{j}$$ 가 **서로 비슷한 값을 가져야** 하며 0으로 수렴할 확률이 줄어든다. 

이러한 현상을 **`grouping effect`** 라 한다.


<br/>

---

## 다양한 모델

- 상관관계를 높은 변수들 동시에 선택 `Elastic Net`
- 데이터상, 물리적으로 인접한 변수들 동시에 선택 `Fused Lasso`
- 사용자가 정의한 그룹 단위로 변수 선택 `Group Lasso`
- 사용자가 정의한 그래프의 연결 관계에 따라 변수 선택 `Grace`

의 다양한 모델이 존재한다. 그 중 수학적 직관으로 쉽게 이해 할 수 있는 `Fused Lasso`에 대해 설명해주었다. 

|![Fused Lasso](https://swha0105.github.io/assets/ml/img/fused.png)   
|:--:| 
|Fused Lasso 데이터 예제|

위 그림은 $$x_{i},x_{i+1} ... $$가 데이터 상 바로 인접해있는 spectrum 분석 그림이다.   
**peak**을 분석 하고 싶을 때, **peak**은 하나의 데이터 포인트로 정의 되지 않기 때문에 **주변의 데이터 포인트들과 함께 분석**을 수행해야 한다.

$$ \beta^{lasso} =  \underset{\beta}{argmin} ( \sum_{i} (y_{i} - \ \sum_{j=0}^{p} (\beta_{j}x_{j}) )^2 + \lambda_{1} \sum_{j=0}^{p} \rvert \beta_{j} \rvert + \lambda_{2} \sum_{j=0}^{p} (\beta_{j}-\beta_{j-1} ))$$  

위의 식에서 $$\lambda_{2} \sum_{j=0}^{p} (\beta_{j}-\beta_{j-1})$$은 **`smoothness`** 즉, 데이터가 인접한 변수로 이루어져 있을때 데이터에 해당되는 beta값들에 대해 차이가 클 수록 penalty를 준다.   
따라서 데이터가 물리적으로 인접할 때, **서로 크게 다른 값을 못가지도록 수식으로 `confinement`**를 준다


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
