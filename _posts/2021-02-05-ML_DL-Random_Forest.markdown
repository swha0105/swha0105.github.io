---
layout: post
title:  "[ML Algorithms] Random Froest"
subtitle:   "supervised Learning"
categories: ml_dl
tags: ml
comments: true
---

**들어가기전..**  
Random forest는 예전에 하드코딩도 해보았고 사용하면서 알고리즘 복잡도에 비해 굉장히 강력하다는 느낌을 받았다.  
알고있는 내용을 체계적으로 정리하기 위해 포스팅하고 주로 [김성범 교수님 강의를](https://www.youtube.com/watch?v=lIT5-piVtRw) 참고하였다.

<br/>


## Random Forest의 필요성:

앞선 [Decision Tree 정리한 포스트]((https://swha0105.github.io/_posts/2021-02-04-ML_DL-Decision_Tree.markdown))  에서 마지막에 언급 했듰이 local optimal에 빠질수 밖에 없다. 솔직히 이 느낌적인 느낌을 말로 설명할 자신은 없고 다들 흔하게 하는말로는 overfitting 즉, 데이터에 과적합 될 확률이 높다고 한다.

그리고 Tree에 오류가 생길 시 그 오류가 계속 전파가 되고 커질 수 있기에 Decision Tree는 실전에서 사용하기 한계가 있다고 한다.

따라서 랜덤포레스트, 즉 무작위하게 나무를 심고 숲처럼 구성해서 모델을 구축하자는 아이디어가 나왔다.

<br/>

---

# Random Forest 개요

### 앙상블

- 앙상블이란 여러 모델들을 사용하여 결과를 도출하는 개념이다.  
- Random Forest는 Decision tree를 base model로 앙상블을 구성한다.
- 앙상블의 결과가 단일 Base model의 결과보다 좋기 위해 base model이 가져야할 조건은 다음과 같다. 

   1. base 모델들이 서로 독립적
   2. base 모델이 최소한의 성능을 가져야함. (무작위 이상)

    <p float="center">
        <img src="https://swha0105.github.io/assets/ml/img/RF_error_rate.JPG" width="300"/> 
    </p>


### 장점 

- 데이터에 비해 Computer resource를 덜 사용 한다.  
- Non parametric model, 즉 비모수적 모델이다. 사전에 데이터 분포에 대한 가정이 필요없다. (보통 많은 모델들은 gaussian distribution을 가정한다. )


## 성능 향상

**<U> 1. Diversity를 확보 </U>**  

   - **Bagging**: 여러개의 Traning data set를 생성하여 각 데이터셋 마다 개별 Decision tree를 생성한다.   

**<U> 2. Random의 성질을 확보 </U>**  

   - **Random subspace** : Decision tree 구축 시 모든 feature를 사용하지 않고 무작위로 몇개만 선택하여 모델 학습.

<br/>

## Bagging (Bootstrap + AGGregatING): 

### Bootstrapping (샘플링 기법)
- **복원 추출**, 이때 원래 데이터 수만큼 샘플링을 한다.
- 이를 통해 만든 데이터 셋을 **Bootstrapping set** 이라 한다
- 복원 추출을 원래 데이터 수만큼 샘플링 하였기 때문에 데이터가 중복 될 수 있음. 

### Aggregating.
- Aggregating은 어떤 것을 합치고 모은다는 개념.
- 각 트리가 예측한 결과를 취합하여 다수결 혹은 평균값으로 Random forest값 결정. **(Voting)**

**<U>1. Major voting</U>**

$$ Ensemble(\hat{y}) =  \underset{i}{argmax} ( \sum_{j=1}{n} I (\hat{y_{j}} = i), i \in {0,1} )$$

   - 각 트리 예측한 Label들중 많은 쪽으로 Voting

**<U>2. Weighted voting</U>**

$$ Ensemble(\hat{y}) =  \underset{i}{argmax} \frac{ \sum_{j=1}{n} (TrainAcc_{j}) I (\hat{y_{j}} = i)}{\sum_{j=1}{n} (TrainAcc_{j})}  , i \in {0,1} )$$

- 각 트리의 Training Accuracy를 고려하여 Voting

**<U>3. Pridicted probaility Weighted voting</U>**

$$ Ensemble(\hat{y}) =  \underset{i}{argmax} ( \frac{1}{n} \sum_{j=1}{n} P(y=i) , i \in {0,1} )$$

- 각 트리에 대해서 모든 Label에 대해 정확히 분류한 비율을 확률로 계산하여 높은 쪽으로 트리가 Voting.  

<br/>

---

# 랜덤포레스트 평가


## Error 계산

- 각각의 개별 tree는 overfitting 될 수 있음.
- Tree 수가 충분히 많을 때, error 의 upper bound인 **generalization error**를 계산 할 수 있음.  
  > generalization error < $$\frac{\bar{\rho} ( 1 - s^2)}{s^2}$$  
  > $$\bar{\rho}$$: Decision Tree 사이의 평균 상관관계  
  > s: 올바로 예측한 tree와 잘못 예측한 tree수 차이의 평균.

$$\bar{\rho}$$: base model인 decision tree끼리는 서로 독립적이야 한다는 전제조건.  
s: base model인 decision tree에 대해 최소한의 성능은 나와야 된다는 전제조건


## 중요 Feature 선택

Bootstrapping 기법을 사용하기 때문에 복원추출에서 선택되지 않는 데이터들이 존재한다. 이와 같은 데이터는 out of bag (OOB) 라고 하며 이 데이터를 이용해 Random forest의 중요 Feature를 다음과 같이 계산한다.

1. 각 붓스트렙셋으로 부터 생성된 tree에서 OOB를 넣고 error 계산. ($$r_{i}$$: OOB error)
2. feature의 값을 바꿔가면서 OOB error 계산. ($$e_{i}$$) 
3. 개별 feature의 중요도는 ($$d_{i} = e_{i} - r_{i} $$)의 평균과 분산으로 결정.

feature i에 대한 중요도는 다음과 같다.

$$v_{i} = \frac{\overset{-}{d}}{s_{d}}$$  

> $$\overset{-}{d}= \frac{1}{t} \sum_{i=1}^{t} d_{i}$$ : 모든 트리의 d 평균 값.   
$$s_{d}^{2} = \frac{1}{t-1} \sum_{i=1}^{t} (d_{i} - \overset{-}{d})^2: d에 대한 분산$$

위 수식의 의미는 **특정한 feature의 값을 변경했을때 만약, 예측 결과가 많이 바뀌어 $$\overset{-}{d}$$이 크다는건 그 변수가 중요한 역할을 한다는것**.  
$$s_{d}^2$$는 어느 정도의 보정치 라고 한다.

## hyperparameter

- 무작위 선택 변수의 수   
  (classficiation: sqrt(변수의 수), regression: 변수의 수/3)
- tree 갯수 
  (보통 2000개 이상.)



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
