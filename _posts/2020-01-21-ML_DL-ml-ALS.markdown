---
layout: post
title:  "[ML Algorithms] ALS"
subtitle:   "Alteranting Least Squares 추천 알고리즘."
categories: ml_dl
tags: ml
comments: true
---

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

# Collaborative Filtering for Implicit Feedback Datasets


# Intro 

recommand system은 크게 2가지. 

1. content based approach 
    - explicit data를 이용하여 추천함
2. alternative strategy
    - implicit data를 이용하여 추천함
    - 데이터를 구하기 쉽지만 어려운 기술이 필요.
    - Collaborative Filtering (CF) 이라고도 불림.


CF는 Implicit feedback에 적합하게 만들어 졌고 Implicit feedback에 대한 중요한 성질들은 다음과 같다.
1. No negative feedback
    - Implicit feedback은은 negative한 반응에 대해 신뢰성이 부족하다. 
2. Implicit feedback은 노이즈가 내제되어있다. 
3. Implicit feedback의 numerical value는 confidence를 나타낸다 
    - explicit feedback의 numerical value는 preference를 나타낸다
4. 적절한 metric을 구해야 한다.
    - explicit feedback의 같은 경우 mse와 같은 직관적인 metric사용 가능

explicit과 비교해서 implicit이 얼마나 어려운지 논문에 하루종일 나온다..

# Prvious work

우린 지금 추천 시스템을 보고 있다.
## Neighborhood models
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


## Latent factor models
- 관찰된 rating을 설명하는 Latent factor를 발견하기 위해 전반적인 접근을 하는 모델 ~~먼말인지 모르겠다~~    
- user-item observation matrix를 SVD시킴.  
~~이 논문에선 너무 짧게 설명 되어있다. 시간나면 여기 부분 다시 공부~~

<br/> 

# 현재 작성중