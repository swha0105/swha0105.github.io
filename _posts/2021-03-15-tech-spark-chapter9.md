---
layout: post
title:  "[모두연 풀잎스쿨 14기] Chapter 9"
subtitle:   "Monte-Carlo Simulation"
categories: dev
tags: spark
comments: False
---

**본 포스팅은 모두의연구소(home.modulabs.co.kr) 풀잎스쿨에서 진행된 `Advanced Analytics with Spark` 과정 내용을 공유 및 정리한 자료입니다.**    

--- 


## Monte-Carlo Simulation


### 정의와 특징
몬테 카를로 시뮬레이션은 다양한 정의가 존재한다. 물리학과 ML/DL에서 말하는 몬테카를로 시뮬레이션은 정의부터 활용 용도까지 천차만별이다.  
많은 정의들 중 가장 일반적으로 잘 설명한다고 생각하는 정의를 가져왔다.

- 무작위수들(Random Numbers)을 이용한 반복적인 연산을 통해 특정 함수의 결과 값을 확률적(Probabilistic)으로 추론

몬테 카를로 시뮬레이션은 특정 함수의 결과 값을 확률적으로 추론하는 알고리즘이다.  
함수의 결과값을 해석학적으로 결정할수 없을때(Nondeterministic) 확률적으로라도 함수의 해를 추론하는 방법으로 수행횟수가 무한대로 발산 할수록 함수의 해는 정답에 수렴한다.

내가 전공했던 물리학에서는 플라즈마와 전자의 상호작용을 연구할때 사용하였다.   
특수한 상태에 있는 플라즈마는 특정한 에너지, 운동량를 가지는 전자들에만 상호작용을 하였다. 이때 전자가 가지는 물리량 (에너지, 운동량)의 추론이 현재 단계에서는 이론적으로 불가능하기 떄문에 전자의 가능한 거의 모든 물리량을 범위 준 다음 랜덤으로 샘플링해서 플라즈마와 작용하는지 지켜보았다.  
충분히 많은 수의 전자를 넣었을 때, 반응한 전자들만 모아서 물리량을 측정해보면 어떤 특수한 분포를 나타내었고 이 연구를 할때 몬테카를로 시뮬레이션를 경험해보았다.

<br/>

### 샘플링 
몬테카를로 시뮬레이션은 샘플링을 임의로, 그리고 독립적으로 시행한다. 이럴 경우 여러가지 단점이 존재하지만 샘플링을 굉장히 많이 해야 특정값에 수렴한다는 단점이 존재한다.   
이러한 단점을 보완하기 위해 여러가지 샘플링 기법이 존재한다.
- **MCMC(Markov Chain Monte Carlo)**
- Rejection Sampling
- Metropolice Hastings


### MCMC 
MCMC (Markov Chain Monte Carlo)은 Markov Chain에서 이용해 생성한 샘플링 데이터를 이용해 Monte-carlo 시뮬레이션 하는것이다. 

Markov Chain의 가장 큰 가정은 다음과 같다.  

> 현재 상태는 바로 직전의 상태에만 영향을 받는다  (Markov Property)

이러한 성질을 이용하여 가장 마지막에 뽑힌 샘플이 다음번 샘플에 영향을 미치고 이러한 성질은 고등학교 수학에서 배운 `조건부 확률` 과 같다. 이 `조건부 확률`을 이용하여 샘플링하여 Monte-carlo 시뮬레이션을 수행하는 것이 MCMC 이다. 









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