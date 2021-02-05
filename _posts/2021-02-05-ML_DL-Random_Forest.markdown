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

앙상블이란 여러 트리들을 한번에 묶는 개념이고 이때 트리들을 base model이라고 한다.   
Decision tree를 앙상블로 구성하면 랜덤포레스트가 된다.  
이때 랜덤포레스트가 단일 decision tree보다 더 좋은 성능을 보이러면 base model의 조건이 있다. 

- base 모델들이 서로 독립적
- base 모델이 최소한의 성능을 가져야함. (무작위 이상)

### 장점 

- 데이터에 비해 Computer resource를 덜 사용 한다.  
- Non parametric model, 즉 비모수적 모델이다. 사전에 데이터 분포에 대한 가정이 필요없다. (보통 많은 모델들은 gaussian distribution을 가정한다. )


### 성능 향상

**1. Diversity를 확보**  

   - **Bagging**: 여러개의 Traning data set를 생성하여 각 데이터셋 마다 개별 Decision tree를 생성한다.   

**2. Random의 성질을 확보**  

   - **Random subspace** : Decision tree 구축 시 모든 feature를 사용하지 않고 무작위로 몇개만 선택하여 모델 학습.


<br/>

--- 


## Bagging (Bootstrap + AGGregatING): 


### Bootstrapping (샘플링 기법)
- **복원 추출**, 이때 원래 데이터 수만큼 샘플링을 한다.
- 이를 통해 만든 데이터 셋을 **Bootstrapping set** 이라 한다
- 복원 추출을 원래 데이터 수만큼 샘플링 하였기 때문에 데이터가 중복 될 수 있음. 

### Aggregating  .

- Aggregating은 어떤 것을 합치고 모은다는 개념.
- Aggregating에는 여러 방법이 존재.

<br/>

# 아직 작성중

<br/>

#### Major voting 

$$ Ensemble(\hat{y}) =  \underset{i}{argmax} ( \sum_{j=1}{n} I (\hat{y_{j}} = i), i \in {0,1} )$$

=> 앙상블에 있는 트리들이 0,1 중 많은 쪽으로

이러면 트레이닝 accuracy 고려안함. ㅎ트리에 대한 확신이 없는 상태. 

#### Weighted voting 

$$ Ensemble(\hat{y}) =  \underset{i}{argmax} \frac{ \sum_{j=1}{n} (TrainAcc_{j}) I (\hat{y_{j}} = i)}{\sum_{j=1}{n} (TrainAcc_{j})}  , i \in {0,1} )$$

#### Pridicted probaility Weighted voting

$$ Ensemble(\hat{y}) =  \underset{i}{argmax} ( \frac{1}{n} \sum_{j=1}{n} P(y=i) , i \in {0,1} )$$

-> 확률은 노드에 대한 조건부 확률 인듯. 


<br/>

# 랜덤포레스트 평가

랜덤포레스트 특성. 

tree수가 충분히 많을때 

generalization error < \fracP\bar{\rho} ( 1 - s^2)}{s^2}

rho: decision tree 사이의 평균 상관관계  -> 독립
s: 올바로 예측한 tree와 잘못 예측한 tree수 차이의 평균 -> 개별 tree


변수의 중요성

out of bag (OOB): 붓스트렙셋에 포함되지 않은 데이터. 
이것을 일종의 validation ..

- 각 붓스트렙셋으로 부터 생성된 tree에서 OOB를 넣고 error 계산. 
- feature의 값을 바꿔가면서 OOB error 계산. 

di = abs(ei - ri)  (ri: 원래 OOB ei: feature값 바꿔가면서 계산한 OOB error )

d값에 대한 분산 

s_{d}^2 = \frac{1}{t-1} sum (d_{i} - mean(d)^2)

이 의미는 특정한 feature를 바꾸었을때 값이 많이 바뀌어 분산이 크다는건 그 변수가 중요한 역할을 한다는것.

중요도 v_{i} =  \frac{mean(d)}{s_{d}} ???


random forest hyperparameter

- 무작위 선택 변수의 수 ( classficiation: sqrt(변수의 수), regression: 변수의 수 /3)
- tree 갯수 ( 최소 2000개 이상.)



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
