---
layout: post
title:  "[ML/DL Lecture] Neural Network 1"
subtitle:   "Deep learning"
categories: ml_dl
tags: ml
comments: true
---


Back to Basic

기본부터..

### 선형회귀모델 

입력변수 X의 **선형결합**으로 출력변수 Y를 표현

$$ f(X} = w_{0} + w_{1}X_{1} + w_{2}X_{2} ... + w_{p}X_{p}$$

### 로지스틱 회귀모델

$$ f_{X} = \frac{1}{1+e^{-(w_(w_{0} + w_{1}X_{1} + w_{2}X_{2} ... + w_{p}X_{p})}}$$ 

입력변수 X의 **비선형결합**으로 출력변수 Y를 표현

1. 입력변수의 선형결합
2. 선형결합 값의 비선형 변환 (Nonlinear transformation)

$$a = \sum_{i=0}^{p} w_{i}x_{i}   \quad \quad  h = \frac{1}{1+e^{-a}}$$

### Perceptron

- 입력값의 선형결합후 그 값이 양수, 음수 인지 분류

단층 perceptron 
or , and 는 가능.
xor 는 불가능.

따라서 2-layer perceptron


Multilayer perceptron = ANN (Artificial Neural Networks)


입력층: 
- 입력변수의 값이 들어오는곳 
- 입력변수의 수 = 입력노드의 수

은닉층
- 은닉층에는 다수 노드 포함 가능
- 다수의 은닉층 형성가능.

출력층
- 출력노드의 수 = 출력변수의 범주 개수 (볌주형)
- 출력노드의 수 = 출력변수의 개수 (연솏형)



### ANN의 파라미터

파라미터: 가중치 -> 알고리즘으로 결정

하이퍼파라미터: 은닉층 개수, 은닉노드 개수, activation function

### cost function

Regression: MSE
Classification: Cross entropy 


### 학습방법

Gradient descent 

- First-order optimization algorithm
- Turning point의 개수는 함수의 차수에 의해 결정
- 모든 turning point가 최솟값 혹은 최댓값은 아님
-


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
