---
layout: post
title:  "[ML Algorithms] Paper review - ALS"
subtitle:   "Alteranting Least Squares 추천 알고리즘."
categories: ml_dl
tags: ml
comments: true
---

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

# Collaborative Filtering for Implicit Feedback Datasets

이 논문은 추천시스템을 소개하는 논문이고 ALS(Alternating Least Square)에 대해 처음 소개한 논문으로 알고있다. 

<br/>

***

## Intro 

#### Recommand system은 크게 content based approach와 alternative strategy로 2가지로 나뉜다. 

1. content based approach 
    - explicit data를 이용하여 추천함
2. alternative strategy
    - implicit data를 이용하여 추천함
    - 데이터를 구하기 쉽지만 어려운 기술이 필요.
    - Collaborative Filtering (CF) 이라고도 불림.

<br/>

CF는 Implicit feedback에 적합하게 만들어 졌고  
**Implicit feedback에 대한 중요한 성질들은 다음과 같다.**

1. No negative feedback
    - Implicit feedback은은 negative한 반응에 대해 신뢰성이 부족하다. 
2. Implicit feedback은 노이즈가 내제되어있다. 
3. Implicit feedback의 numerical value는 confidence를 나타낸다 
    - explicit feedback의 numerical value는 preference를 나타낸다
4. 적절한 metric을 구해야 한다.
    - explicit feedback의 같은 경우 mse와 같은 직관적인 metric사용 가능

~~explicit과 비교해서 implicit이 얼마나 어려운지 논문에 하루종일 나온다..~~

<br/>

***

<details>
<summary> Implicit vs Explicit </summary>
<div markdown="1">

Explicit feedback은 데이터가 가지고 있는 값이 우리가 사용해야할 데이터와 일치할때 사용된다  
예를 들어, 이 예제와같이 음악추천 시스템을 만들때 유저가 듣는 음악에 대해 평점을 매긴다면 그 평점에 대한 데이터를 사용하면 Explicit feedback이 된다.  

예제에서 사용하고 있는 데이터는 단순히 조회수로만 카운팅하며 이럴 경우 많은 bias나 오차가 생길수 있다. (틀어놓고 딴짓한다던가.. 기타등등) 이런 데이터를 이용하는것을 Implicit feedback이라고 한다. Implicit은 Explicit에 비해 사용하기 힘들지만 큰 데이터를 모을수 있는 장점이 있다.

</div>
</details>

# Prvious work

**기존 연구들은 크게 Neighborhood models와  Latent factor models 2가지로 나뉜다.**

## Neighborhood models
**그 중 Neighborhood models같은 경우 user-oriented model과 item-oriented model로 나뉘는데, user-oriented model은 거의 사용하지 않았고 item-oriented같은 경우 연구가 꽤 진행되었다.**

1. user-oriented model
    - 유사한 사용자의 rating을 기반으로 알 수 없는 rating을 추정한다.  

2. ### item-oriented model
    - 동일한 사용자가 유사한 item에 대해 수행한 알려진 similarity을 이용해 추정
        - (수행이 implicit, expliclit에 따라 model이 implicit feedback에 사용될지, explicit feedback에 사용될지 결정 된다.)

    **Limitation** 
    - similarity의 계산이 직관적이지 않다.
    - outlier 유저들에 대한 예측이 어렵다.
    - numeric value가 implicit, explicit feedback에 대해 구분이 없다  
     (preference와 confidence의 개념이 혼용되어 사용)  
    
    <br/>

    예를 들어, User U와 Item i 에 대한 알려지지 않은 연관성을 예측하고 싶다. 
    이때, Item j들은 이미 알고 있다. ($$S^{k}$$ 집합에 속한다)  

    $$ 
    r_{ui} = \frac{ \Sigma_{j \subset S^{k} (i;u)} s_{ij}r_{uj}  }{\Sigma_{j \subset S^{k} (i;u)} s_{ij} } 
    $$    

    >$$r_{ui}$$ 는 User u와 Item i에 대한 연관성  
    $$r_{uj}$$ 는 User u와 Item j에 대한 연관성    
    $$s_{ij}$$ 는 Item i,j의 유사도 (similarity)  
    $$ \Sigma_{j \subset S^{k} (i;u)} s_{ij} $$ 는 User u에 대해 i와 비슷한 k개의 item에 대한 모든 similarity sum 한 값

<br/> 


## Latent factor models
- 관찰된 rating을 설명하는 Latent factor를 발견하기 위해 전반적인 접근을 하는 모델 
- user-item observation matrix를 SVD시킴.  
- 여기서 소개 하는 모델도 이 방법을 따름. (아래 설명..)

<br/> 


# Model

## 선행지식: Matrix Factorization

Matrix Factorization은 추천시스템에서 많이 쓰는 수학적 테크닉이다.  
유저와 Item으로 (비평가된 항목 포함) 구성된 Matrix R (N X M) 이 있을 때 초기 랜덤한값이 포함되어있는 User-latent factor(N x K)와 Item-latent factor(M X K) Matrix를 구성해 Matrix R를 재구성하여 R에 비평가된 항목도 채워주는 방식이다.
이때 K는 rank이고 feature 수를 나타낸다.

재구성된 Matrix R 을 $$R^{'}$$ 이라고 하자.  
이 방법의 **Cost function**은 다음과 같다.  

$$ \min_{q,p} \sum_{(u,i) \subset K} (r_{ui} - q_{i}^{T}p_{i})^2 + \lambda (|| {q_{i}}^2 || + || {p_{u}||^2 }  ) \quad \quad ||q_{i}^2|| = (\sum_{i}^n  |q_{i}| ^{p} )^{\frac{1}{p}}$$    
<br/>
이고 p = 2인 L2 Norm이다.

이 cost function을 최소화 하며 P 와 Q에 대한 값을 찾아 나간다. 이때 
>$$r_{ui}$$ 은 원래 구성되어있는 Matrix R에 대한 항목  
>$$ R^{'} = q_{i}^T p_{i} (\approx R) $$은 재구성하여 R과 비슷한 값을 가지지만 비평가된 항목이 업데이트 된 Matrix

수식의 의미를 뜯어보자면, 원래 구성되어있는 Matrix R에 재구성하여 비평가된 항목이 업데이트 된 Matrix R_prime을 빼주어서 그것의 값을 최소화 하는 수식이다.  
두번째 텀은 Regularization텀이고 overfitting을 방지하는 수식이다.  

Matrix Factorization은 이 cost function을 최소화 하는것이고 이 논문에서 소개하는 **ALS (Alternating Least Square)** 는 이 cost function을 optimazation하는 방법 중 하나이다.

<br/>

## Model 핵심:
이 모델의 핵심은 다음과 같다.  
1. confidence level 구성. 
2. x,y에 대한 optimazation을 따로 함. (ALS)


### 1. confidence level 구성. 
Implicit feedback모델의 같은 경우 Preference를 구성하기 힘든 단점이있다. 인트로에서 언급했다시피 Explicit feedback 모델들은 numeric value가 바로 prefence를 나타내지만 Implicit는 그러지 않아 유저가 얼만큼 선호하는지에 대해 수치화 하기 어려움이 있었다. 

이 논문에서는 Confidence level를 구성하여 Preference 역할을 대체하는 개념을 제시했다. 

$$ P_{ui} = \begin{cases} 1 \quad (r_{ui} > 0) \\ 2  \quad (r_{ui} = 0)\end{cases} \quad \quad c_{ui} = 1 + \alpha r_{ui}$$  

왼쪽 수식은 preference를 나타내는 binary 형식이고 오른쪽은 이 논문에서 제시한 **Confidence level**이다. 이 수식을 이용하여 **Cost function**을 다시 구성해보자.  

<br/>

$$ \min_{q,p} \sum_{(u,i) \subset K} c_{ui}(p_{ui} - q_{i}^{T}p_{i})^2 + \lambda (|| {q_{i}}^2 || + || {p_{u}||^2 }  ) $$    

이 cost function은 Matrix factorization에서 소개한 cost function과 거의 일치하지만 $$r_{ui}$$ 대신 $$c_{ui} = 1 + \alpha r_{ui}$$로 대체하였다.  

cost function에 대한 의미를 해석해보자면 Implicit dataset은 $$r_{ui}$$ 즉, rating에 대한 값이 부정확할 수 있기에 업데이트 되는 값들을 binary하게 받아들이지 않고 continous 하게 받아 들여 오차에 대한 flexibility를 가지게 만든것 같다.

### 2. x,y에 대한 optimazation을 따로 함. 
**이 개념이 ALS의 핵심개념이다.**   
Optimazation을 하기 위해 기존 explicit feedback에서 사용되는  SGD(stochastic gradient descent)은 쓰기에 implicit dataset은 대체로 너무 크다. 

이때 ALS 개념이 등장하는데 user-factor, item-factor 에 대해 iteration을 하며 값을 찾을 때, SGD 처럼 각 변수에 대해 한번씩 번갈아가며 미분을 하면 계산시간이 크니깐 **한 변수에 대해 몇번씩 미분을 한다음 업데이트를 하자**가 핵심이다. 이때 한 변수에대해 몇번씩 미분을 할때 계산 결과를 재활용(?) 할 수 있어 computation resource를 줄일수 있다고 한다.  

cost function을 최소화 하는 $$x_{u}$$을 찾으면 다음과 같다 (고 한다.. 논문에 나와있음..)  

$$ (x_{u} = Y^{T}C^{u}Y + \lambda I)^{-1} Y^{T}C^{u}p(u)$$

이 수식을 조금 손을 보게되면 $$Y^{T}C^{u}Y$$은 $$Y^{T}Y + Y^{T}(C^{u}-1)Y$$ 로 변형이 가능한데, 이때 $$Y^{T}Y$$은 user u 에 대한 Independent한 값이다. 따라서 이 값은 재활용이 가능하다.   
$$Y^{T}(C^{u}-1)Y$$ 값에 대해서는 $$C^{u}-1$$은 user가 선호하는 즉, $$c_{ui} > 1$$에 대해서만 계산하고 나머지는 0이 되기때문에 여기서도 계산량을 줄일 수 있다.

이와 같이 $$x_{u}$$에 대해 여러번 계산하며 재활용을 충분히 한 뒤 업데이트 하고, $$y_{u}$$에 대해서도 같은 일을 반복한다면 computations resource를 줄일 수 있을 것이다. (논문에서는 10번 정도가 적당하다고 언급함.)

<br/>

***

### Ref
1. [논문 링크](https://swha0105.github.io/assets/ml/paper/Collaborative_Filtering_for_Implicit_Feedback_Datasets.pdf)       
2. 논문만으로는 전부를 이해하지 못해 이 [블로그](https://yamalab.tistory.com/89)를 참조하였다.