---
layout: post
title:  "[ML Algorithms] Label Spreading"
subtitle:   "Semi-supervised Learning"
categories: ml_dl
tags: ml
comments: true
---

# Label spreading 개요 및 경험
Label spreading 알고리즘은 **Semi-Supervised Learning 알고리즘** 중 하나로써 [Filament Project](https://swha0105.github.io/projects/2021/01/24/projects-projects-filament/) 를 할때 사용하였다.
이 프로젝트는  총 4개의 구조를 가지는 데이터에서 정량적으로 구분이 되는 2개의 구조와 (`Cluster`, `Void`) 그렇지 않은 구조 특정 구조를 2개를 (`Filament`, `Wall`)구분하여 분류하는 프로젝트였다. 

이 두개의 구조는 (`Filament`,`Wall`) 3차원 공간 데이터상에 존재하며 기하학적 정보와 물리학적 정보를 함께 보며 판단해야하고 또한 주변의 환경들도 문맥적으로 고려해야한다.

우리의 가설은 `Filament`는 `Cluster`와, `Wall`은  `Void`와 4차원 공간상에서 좀 더 가까이 있을꺼라 판단하였다. 4차원 공간은 물리 데이터 3개 (온도, 밀도, X-ray)와 기하학적 정도를 나타내는 데이터 (Shape Strength)를 계산하여 4차원 데이터 공간을 구성하였다.

따라서, 정량적인 지표로 구분이 가능한 `Cluster`와 `Void`를 **Label**로 가정하고 **Unlabel**데이터 인 `Filament`와 `Wall`이 4차원 공간상에서 어디와 더 가까운지 판단하려고 했다.

<br/>

# Mathematical Principle
Let data pointsX={x1,x2,...xl,xl+1,...xn}, labeled pointsxLb{−1,1}(1≤L≤l) and unlabeled pointsxU= 0(l+ 1≤U≤n)  
Since To ignore outlier effect (cluster center), we choose K-nearest neighborhood for affinity matrix (strictly Adja-cency matrix) typeWij= 1if xibneighborsk(xj)Wij= 0otherwiseCompute  Probability  transition  matrixPij=WijΣkWkjand  Normalized  graph  laplacianL=D−12WD12whereDis a Degree matrixLet define set of Probability transition Matrix at some pointYi={Pi,1(label= 1),Pi,2(label=−1)}ΣkPi,k= 1iteration below equation untilYt+1−Yt≤toleranceYt+1=αLYt+ (1−α)Y0


원리 
- 수식.


예시 결과 





## Reference

논문, Scikit learn