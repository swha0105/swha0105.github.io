---
layout: post
title:  "[ML/DL Posts] Varience Inference"
subtitle:   "Post"
categories: ml_dl
tags: ml
comments: true
---

[참조](https://swha0105.github.io/assets/ml/posts/VariationalInference.pdf)  

### 들어가기전
Variational Inference에 대해 깊게 공부할일이 있어 구글링하다가 좋은 자료를 발견하여 정리한다.   
수식 중간중간에 크리티컬한 오타가 있기에 보는데 주의해야한다.


# Variational inference

## Intro

- Posterior distribution이 Intractable이여서 parameter estimation이 불가능할때 사용하는 방법이다.
- Latent variable에 대한 point estimate하는 Expectation Maximize(EM)과는 다르게, Variantional Inference estimate는 Closed form denst function을 추론한다
- Sampling을 통한 추론을 하는 Monte-Carlo Markov Chain  (MCMC)와는 다르게, analytical approximiation을 통해 posterior를 추론한다

## Goal

- Bayes thm에서 나온 **Z에 대한 posterior distribution**을 이용해 hidden data를 inference 하는것 이다.  

$$ p(z \lvert x) = \frac{p ( x \lvert z) p(z)}{p(x)} $$ 

>  Z: hidden random variable  
>  X: observed data  
>  p(z,x): joint model over z,x  
 
- $$p(z \lvert x)$$을 추론하기 어렵기에 (Intractable) 그와 비슷한 `q(z)`를 만들어 그 두개의 차이를 줄이는 방향으로 접근한다.

- `q(z)`는 z에 대한 family of surrogate distribution으로 도메인을 제한하는데 이 제한된 함수의 족들을 $$q(z \lvert \phi)$$ (variational distribution family)라고 한다  

- **Variational Inference의 `posterior distribution` $$P(z \lvert x)$$에 최대한 가까운 `variational distribution family` $$q(z \lvert \phi)$$를 구하는게 목적이다.** 이는 적절한 $$\phi$$를 찾는것과 동일하다
 >$$\phi$$: variational parameters. variational family의 member를 characterize하는데 사용한다.)


## Method

- 두개의 probability distribution의 `closeness` 측정하기 위해 **`KL-divergence`**을 사용한다.

$$KL(q(z \lvert \phi) \lvert\lvert p(z \lvert x)) = E_{z \sim q}(log \frac{q(z \lvert \phi)}{p(z \lvert x)})$$

- variational inference는 위의식을 이용해 **최적의 $$\phi$$**를 찾는다. 

$$\hat{\phi} = argmin_{\phi} KL(q(z \lvert \phi) \lvert\lvert p(z \lvert \phi))$$

- 이를 이용해 우리가 최종적으로 구하는 variational distribution은 $$q(z \lvert \hat{\phi})$$ 이다.  

- 위의 식을 풀기 위해 먼저  **`Evidence Lower BOund` (ELBO)**의 정의를 보면 다음과 같다.

$$ \log(p(x)) = \log \int p(x,z) dz $$ (marginal probability)
$$ = \log \int p(x,z) \frac{q(z \lvert \phi)}{q(z \lvert \phi)} dz = \log \int q(z \lvert \phi) \frac{p(x,z)}{q(z \lvert \phi)} dz \quad ( E(f(x))_{x \sim  p} = \int p(x)f(x) dx )$$  
$$ = \log( E_{Z \sim  q}(\frac{p(x,Z)}{q(Z \lvert \phi)})) \; \geq E_{Z \sim  q}(\log \frac{p(x,Z)}{q(Z \lvert \phi)})$$  (by Jensen's Inequality)  
$$ = E_{Z \sim q}(\log p(x,Z)) - E_{Z \sim  q}(\log q(Z \lvert \phi))$$  **(Defintion of ELBO)**  
  
<!-- $$ \log(p(x)) \geq  E_{Z \sim q}(\log p(x,Z)) - E_{Z \sim  q}(\log q(Z,\phi))$$ (ELBO) -->

- **`Evidence Lower BOund` (ELBO)**의 정의를 활용해 KL divergence를 다시 쓰게 되면 다음과 같다.

$$KL(q(z \lvert \phi ) \lvert \lvert p(z \lvert x)) = E_{Z \sim q}(\log\frac{q(Z \lvert \phi)}{p(Z \lvert x})$$  
$$ = E_{Z \sim q} ( \log q(Z \lvert \phi)) - E_{Z \sim q} ( \log q(Z \lvert x))$$  
$$ = E_{Z \sim q} ( \log q(Z \lvert \phi)) - E_{Z \sim q} ( \log p(Z,x) - \log p(x))$$  
$$ = E_{Z \sim q} (\log p(x)) - (E_{Z \sim q} ( \log p(Z,x) - E_{Z \sim q} ( \log q(Z \lvert \phi)))$$  
$$ = \log p(x) - ELBO$$

- 따라서, $$p(z \lvert x)$$ 를 구하기 위해 비슷한 $$q(z \lvert \phi)$$ 를 추론하는것은 KL을 최소화 하는것이고 이는 ELBO를 최대화 하는 $$\phi$$를 찾는것과 동일하다.

- **Posterior probability**를 모르고 **observed data** (Joint model, $$ \log p(x,Z)$$) 을 알고 있다고 가정할 때 구할 수 있는 방법.




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
