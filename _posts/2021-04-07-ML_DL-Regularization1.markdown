---
layout: post
title:  "[ML Lecture] Regularization, Ridge"
subtitle:   "Machine learning"
categories: ml_dl
tags: ml_lecture
comments: true
---

**들어가기전..**  
이 강의에 있는 내용은 대충(?) 다 아는 내용이라 정리할까 말까 고민을 엄청 했다.  
그러다가 그냥 고민할 시간에 공부하자 라는 생각으로 복습 겸 정리를 했다.. ~~진작에 좀 하지~~   
강의 다 듣고보니 생각보다 모르는 내용이 조금 있었다. 역시 다 안다고 생각하지 말고 배움앞에서  겸손해야 된다..

<br/>

# What is a good model?
연속형 데이터를 인풋으로 받는 모델이라고 가정하자.    
- 현재데이터를 잘 설명하는 모델: MSE(traning) = $$(Y - \hat{Y})^2$$ 최소화
- 미래데이터를 잘 설명하는 모델: MSE(Expected) = $$E[(Y-\hat{Y})^2 \rvert X]$$ 최소화 

### Expected MSE 의 해석
$$E[(Y-\hat{Y})^2 \rvert X] = E(Y^2 - 2Y\hat{Y} + \hat{Y}^2)$$   
$$=E[Y^{2}] + E[\hat{Y}^2] - 2E[Y\hat{Y}]$$    
$$=Var[Y] + E[Y]^2 + Var[\hat{Y}]  + E[\hat{Y}]^2 - 2E[Y\hat{Y}]$$  $$ \quad  ( Var[Y] = E[Y^{2}] - E[Y]^2) $$  
$$=Var[Y] + Var[\hat{Y}] + (E[Y] - E[\hat{Y}])^2 $$  

> $$ Var[Y] $$ = Irreducible Error   
> $$ Var[\hat{Y}] $$ = Variance (예측된 값 끼리의 consistency의 정도)  
> $$(E[Y] - E[\hat{Y}])^2$$ = bias^2 (예측된 값이 정답(Label)과 얼마나 떨어져있는가)

따라서 좋은 모델을 위해서는 bias 와 variance 을 줄여야한다.   
이러한 수식 유도 과정을 `Bias-Variance Decomposition` 이라 한다.

<br/>

---

# Ordinary Linear Regression Model

### Least Squares Estimation method (최소 제곱법)
- 평균제곱오차 (MSE)를 최소화 하는 regression coefficient $$\beta$$ 를 계산.

$$ \hat{\beta}^{LS} = \underset{\beta}{argmin} \sum_{i}^{n} (y_{i} - x_{i}\beta)^2 = (X^T X)^{-1} X^{T}y$$

- $$\beta$$ 에 대한 unbiased estimator 중, 가장 분산이 작은 estimator

<br/>

---

# Regularization 

![정규화](https://swha0105.github.io/assets/ml/img/regularization.png)   

1. 정답과 거리가 건, Bias가 높은 모델
2. Good model
3. 정답과 일치하여 Bias가 낮지만, Variance가 높은 모델 (미래 데이터(test) 에 대한 예측 X)

3번 모델을 통해 LSE모델의 loss function을 구성할 때, regularization 개념은 다음과 같다.  

$$ L(\beta) = \underset{\beta}{min} ( \sum_{i} (y_{i} - \ \sum_{j=0}^{4} (\beta_{j}x_{j}) )^2 + \lambda \sum_{j=0}^{4}  \beta_{j}^2)$$  

> 첫번째 항: Traning accuracy를 의미, MSE와 일치  
> 두번째 항: Generalization accuracy를 의미.

위의 식 2번째 항 처럼 $$\beta$$ 값이 올라갈 때 패널티를 (큰 lambda coefficient) 줘서 모델이 학습 데이터에 overffiting 되는것을 방지. **즉 test데이터에 대한 variance 떨어뜨림.** 

**$$ \lambda $$ 값에 따라 traning & generalization accuracy가 trade off 됨**

<br/>

# Ridge regression

Ridge regression은 $$L_{2}$$-norm regularization 에 속한다.   

$$ \beta^{ridge} =  \underset{\beta}{argmin} \sum_{i}^{n} (y_{i} - x_{i}\beta)^2 \quad subject \; to \quad \sum_{j=1}^{p} \beta_{j}^2 \leq t $$ 

위의 수식에서 subject to는 제약을 둔다는 의미이다. 즉 $$\beta^2$$ 의 더한 값이 t를 넘지 않을 조건에서 위의 식을 풀어야한다.  이 문제는 내가 전공한 물리학, 그 중에서도 고전역학에 ~~지긋지긋하게~~ 정말 많이 나오는 문제이다.  
위와같이 제약이 있는 수식에서 해를 찾을때는 **`lagrangian multipluer`** 의 개념을 사용한다. 이 개념을 사용하여 위의 수식을 다시 풀어보자.

$$ \beta^{ridge} =  \underset{\beta}{argmin} ( \sum_{i} (y_{i} - \ \sum_{j=0}^{p} (\beta_{j}x_{j}) )^2 + \lambda \sum_{j=0}^{p}  \beta_{j}^2)$$  

이 식은 위에 나온 `Regularization`에서 최소의 loss function을 구할 때, $$\beta$$ 값의 수식과 일치 한다. 이 두개의 수식이 일치하는 이유는, $$\lambda$$ 값을 적절히 조절해  $$\sum_{j=1}^{p} \beta_{j}^2 \leq t$$ 을 만족하도록 만들 수 있기 때문이다.  

<br/>

### 1.  **`lagrangian multipluer`** 풀이법으로 **`Ridge regression`** 을 해석해보자

위의 수식에서 첫번째 항, traning data에 대한 것만 고려한 `MSE` 파트에서 $$\beta$$ 와 x의 feature가 2개만 있다고 가정하고 **수식을 전개해보자.**  


$$MSE(\beta_{1},\beta_{2}) = \sum_{i=1}^{n} (y_{i} - \beta_{1}x_{i1} - \beta_{2}x_{i2} )^2$$  
$$ = (\sum_{i=1}^{n} x_{i1}^2)\beta_{1}^{2} + (2 \sum_{i=1}^{n} x_{i1} x_{i2} )\beta_{1} \beta_{2} + (\sum_{i=1}^{n} x_{i2}^2)\beta_{2}^{2} $$   
$$\quad - 2 (\sum_{i=1}^{n} y_{i} x_{i1}) \beta_{1} - 2 (\sum_{i=1}^{n} y_{i} x_{i2}) \beta_{2} + (\sum_{i=1}^{n} y_{i}^2) $$


수식이 엄청 복잡해 보이지만 $$\beta$$ 에 대해서만 정리하고 나머지는 상수 취급을 하면 

$$ = A\beta_{1}^2 + B\beta_{1}\beta_{2} + C\beta_{2}^2 + D\beta_{1} + E\beta_{2} + F$$   
로 다시 쓸 수 있고 이 수식은 `Conic equation`의 form과 같다.  

여기서 판별식 B^2 - 4AC을 계산해보면

$$ B^2 - 4AC = 4( (\sum_{i}^{n} x_{i1} x_{i2}   )^2  - \sum_{i=1}^{n} x_{i1}^2 \sum_{i=1}^{n} x_{i2}^2)$$

위 수식은 고등학교때 배운 `Cauchy-Schwartz 부등식` 에 의해 무조건 음수가 된다.  
따라서 MSE는 $$\beta_{1} \beta_{2}$$에 대해 contour를 그렸을때 항상 **`타원`** 형태를 띄게 된다.   
  
이 사실을 염두해두고 밑에 그림을 보자.

![Ridge regression](https://swha0105.github.io/assets/ml/img/ridge_regression.png)   


위의 그림에서 $$\hat{\beta^{LS}}$$ 값은 제약조건이 없을때 Least squares estimator를 의미한다.  
초록색 타원형 등고선은 $$ \beta_{1} \beta_{2} $$의 공간에서 MSE 값을 나타낸 것이다.   
파란색 원은  $$ \beta_{1} \beta_{2} $$ 가 가잘수 있는 제약조건을 의미한다.  
  
**따라서, 파란색 원 (제약조건)과 만나는 최소의 초록색 타원 등고선 (MSE 값)에 $$\beta_{1} \beta_{2}$$ 가 $$\hat{\beta^{ridge}}$$ 가 된다.**

<br/>

### 2. **`Ridge regression`** 을 행렬 연산을 통해 해석해보자.

$$ MSE(\beta) = (y-X\beta)^{T} (y-X\beta) + \lambda \beta^{T} \beta $$    
$$ \frac{\sigma MSE(\beta)}{\sigma \beta}  = 2X^{T}y + 2(X^{T}X + \lambda I_{p}) \beta = 0 $$
을 만족하는 $$\beta$$ 값은 

$$ \hat{\beta^{ridge}} = (X^{T}X + \lambda I_{p})^{-1} X^{T}y $$ 
가 된다. 이때, 최소 제곱법의 $$\beta$$는 
$$ \hat{\beta_{LS}} = (X^{T}X )^{-1} X^{T}y$$ 이다.

이 수식의 의미하는 바로는 least squares esimator와 다르게 **X에 대한 데이터 matrix에다가 $$\lambda$$ 값을 더해주겠다** 가 된다.    
다른 말로는 unbias estimator가 아닌 bias estimator가 된다고 말할수도 있다.



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
