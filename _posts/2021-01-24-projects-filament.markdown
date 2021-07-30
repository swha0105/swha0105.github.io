---
layout: post
title:  "[고에너지 천체물리 연구] 머신러닝/딥러닝을 통한 3차원 은하 필라멘트 검출"
subtitle:   "Statistical study on morphologies of Leo filaments via Label spreading"
categories: projects
tags: projects
comments: False
---

<details>    
<summary> Project Info  </summary>
<div markdown="1"> 

<br/>

<!-- ### 머신러닝/딥러닝을 통한 3차원 거대우주구조 데이터상의 은하 필라멘트 검출  -->
- **기간:** 19.09 ~ 21.02 (1년 6개월)
- **주관 기관:** 고에너지 천체물리 연구센터 (CHEA, Center for High Energy Astrophysics)
- **주관 부처:** 한국연구재단 선도연구센터
- **사용 언어** : Python, MATLAB, Scikit-learn, Tensorflow-Keras
- **기여 및 결과**:
    1. 머신러닝 / 딥러닝을 통한 3차원 거대우주구조에서 은하 필라멘트 검출
        - 해외 초청발표: Joint Australian and Korean Contributions to the SKA meeting (2020)
        - 국내 발표: 4th CHEA workshop (2020)
    2. 상대론적 유체역학 코드 병렬화
        - SCI논문 제출 준비: The Astrophysics Journal(ApJ) (~2021.05)

![filament project](https://swha0105.github.io/assets/projects/filament/filament_project.png)

<br/>

</div>
</details> 


<details>    
<summary> Motivation & Problem  </summary>
<div markdown="1"> 

## Motivation

|![LSS example](https://swha0105.github.io/assets/projects/filament/LSS_example.JPG)
|:--:| 
| LSS example,  1. Sloan sky survey(left),  2. [그림 출처](https://arxiv.org/pdf/1803.11156.pdf)(right) |

현재 인류가 관측 할 수 있는 가장 큰 우주를 보았을 때, 우주에 존재하는 물질 들은 균등하게 분포해있지 않고 특정한 구조를 가지며 분포 해있다.  
전체 우주의 물질분포와 그 구조를 **거대우주구조 (LSS, Large Scale Structure of the Universe)**라고 하며 이 거대우주구조에는 `Galaxy of Clusters`, `Galaxy of Filaments`, `Wall`, `Void` 의 구조가 존재한다.

- Galaxy of Clusters: 구형으로, 다른 구조들에 비해 온도와 밀도가 월등히 높다.

- **`Galaxy of Filaments`**: 3차원상에서 **원통형**이며 온도와 밀도가 상대적으로 Wall 보다 높다

- **`Wall`** :3차원상에서 **평면구조**이며 온도와 밀도가 상대적으로 Filament 보다 낮다

- Void: 매우 낮은 밀도와 온도를 가지고 있다.

이 중 **`Galaxy of Filament`** (이하 `Filament`)는 우주 전체 물질 질량의 50%를 차지하지만 부피는 6%를 차지하는 구조로서 우주의 물질들이 굉장히 밀집되어 있어 흥미로운 물리현상들이 많이 일어나는 곳이다. [참조 논문](https://academic.oup.com/mnras/article/441/4/2923/1213214)   

따라서, `Filament`를 Identification 하는 문제는 천체물리학계에서 많은 시도가 있고 난제로 속한다. 

![Filament in LSS](https://swha0105.github.io/assets/projects/filament/filament_mass.png)

<br/>

## Problem:

하지만 이 **Filament**는 **Wall**과 구분하기 힘든 성질이 있는데 이유는 다음과 같다.

1. 밀도, 온도와 같은 Filament와 Wall를 나누는 **물리적 기준의 부재**
2. 기하학적 정보 수치화의 어려움
3. 주변 배경(Cluster) 마다 기준이 다름.  

Filament와 Wall은 Cluster 구조 주변에 존재하며, 기하학적으로 **Filament는 3차원 공간상 원기둥, Wall은 평면 구조**를 띈다.   
하지만 Filament와 Wall은 구분할 수 있는 물리적인 기준이 없고, 따라서 이를 구분하기 위해 3차원 상 기하학적 패턴, 물리량, 시뮬레이션 환경 등 **모든 것을 고려해 종합적인 판단을 내린다.**  
    
여기서 가장 문제가 되는 점은 **Filament와 Wall이 존재하는 배경에 따라 물리적 기준과 기하학적 기준이 변한다는 점이다.**

<br/>  

이 문제를 풀기 위해 우리가 힌트를 얻은 지점은 다음과 같다.

1. 절대적인 기준을 찾는 방향 보다는 주변 배경(Cluster) 맞게 기준을 각각 생성한다
2. Cluster와 Void는 물리적으로 구분이 가능하다.
3. Filament는 Cluster와, Wall은 Void에 상대적으로 조금 더 연관성이 있다. 
4. 기하학적 정보를 수치화 하여 물리적 정보와 함께 쓴다.

이러한 접근을 위해 **Machine Learning** 의  **Label spreading** 을 사용하였다.  
특히, Cluster와 Void는 물리적으로 구분이 가능하기에 이를 `Label`로 사용할 수 있음을 알고 Semi-supervised learning 접근을 하였다.  

이를 통해 우리는 특히 `Filament`의 **`Curvature`** 를 계산을 하였다. 실제 관측데이터에서는 Curvature가 큰 Highly curved Filament가 많이 나오지만 시뮬레이션 데이터 상 아직까지 보고 된 바가 없음으로 ML을 통한 Filament Identification이 모든 데이터에 성공적으로 적용할 수 있다면 무수히 많은 Filament들의 통계값을 통해 Highly curved Filament의 존재 유무와 존재 확률을 통계적으로 접근 할 수 있기 때문에 Identification 후 Curvature 계산에 집중하였다.   

데이터는 본 [논문](https://ui.adsabs.harvard.edu/abs/1993ApJ...414....1R/abstract) 에 언급된 코드를 (Fortran) 이용해 계산된 데이터를 이용하였다.    
Size: 32GB x 8    
Format: Binary (3차원)


<br/>

</div>
</details> 



<details>    
<summary> Previous Research & Limitation  </summary>
<div markdown="1"> 

## Previous Research

기존의 연구들은,  물리학적 정보를 사용하지 않고 기하학적 정보들만으로 `Filament` 와 `Wall` 을 구분하는 시도를 하였다. 

1.  3차원 밀도공간에서 [Hessian Matrix](https://en.wikipedia.org/wiki/Hessian_matrix) 의 Eigenvalue를 계산해 **Shape strength** 를 구성하는 방법이 있다. 각각의 **Shape Stength** 는 특정 포인트에서의 주변 밀도의 구조를 고려해 **구, 원기둥, 평면** 정도 를 나타낸다. **Shape Strength**를 이용해 `Filament` 와 `Wall`의 후보군을 찾아낸 뒤, 알려진 물리량과 함께 `Filament`를 판단한다 ([논문 1](https://arxiv.org/abs/1401.7866), [논문 2](https://arxiv.org/abs/1209.2043))\

    <p float="center">
        <img src="https://swha0105.github.io/assets/projects/filament/signature_equation.png" width="400"/> 
    </p>
    Eq 1. Shape Strength. for each lambda means Eigenvalue of Hessian Matrix 
<br/>

2. 위상수학에서 사용되는 [Morse Theory](https://en.wikipedia.org/wiki/Morse_theory)를 이용하여 3차원 공간의 밀도분포에서 가장 안정화된 Saddle point들을 찾아 이를 잇는 선을 찾아내여 `Filament` 를 정의하는 것이다. 오픈소스 코드로 배포가 되어있으나 현재 우리가 사용하는 데이터 형태와 호환이 불가능하다.
[논문 1](https://academic.oup.com/mnras/article/414/1/350/1090746?searchresult=1)
<br/>
<br/>


## Limitation

기존 연구에서 기하학적 정보를 수치화를 하여 Filament를 식별을 하였다.  
이 방법의 문제점은 크게 2가지가 있는데.

1. 물리량과 기하학적 정보의 단위가 달라 각각의 정보만 사용하고 유기적으로 분석하지 못하였다.
2. 주변 배경(Cluster)에 따라 변하는 Filament와 Wall의 정보를 전혀 담지 못하였다.

물리량과 기하학적 정보를 동시에 사용하지 못하여, 신호가 약하거나 애매한 Filament인 경우 식별을 하지 못하였다.  
또한 주변 배경과 관계없이 절대적인 기준을 찾는 시도는 다른 시뮬레이션 데이터 셋으로 확장 불가능하였다.  


따라서, 우리의 목표는 **전문가의 개입 없이, 데이터에 의존성없이, 주변 배경과 상대적인 Criteria 각각 생성하여 데이터에 대한 Bias와 상관없이 Filament를 식별하는 것이다.**

<br/>

</div>
</details> 


<details>    
<summary> Approach  </summary>
<div markdown="1"> 
 

 
### 1. **Gaussian Pyramid를 통한 데이터 압축**  -> [code](https://github.com/swha0105/Filament_Project/blob/main/pyramid.py) 
> `Filament` 와 `Wall`은 `Cluster`주변에 존재하기 때문에 데이터를 `Cluster` 주변 40~45Mpc/h 의 크기로 Crop하였다.  Crop을 한 데이터들도 크기가 250~300Mb 하기 때문에 ML/DL에 사용하기 적합하지 않았다. 따라서 Gaussian Pyramid 알고리즘을 2번 적용해 데이터를 압축하였다. 이 과정에서 기하학적 정보가 손실 되지 않았는지 체크하였고 아래 그림과 같이 확인 하였다
<br/><br/>
![Gaussian example](https://swha0105.github.io/assets/projects/filament/Cluster_dens.png)
<br/>
<br/>


### 2. **GPU 병렬화를 통한 Signature 및 Shape strength계산** -> [code](https://github.com/swha0105/Filament_Project/blob/main/gpu_signature.py) 
> 기존 연구방법을 참고하여 기하학적 정보를 **Shape Signature**를 계산하여 사용하기로 하였다. 
**Shape Signature**는 밀도 데이터의 **Hessian Matrix**를 구성하여 **Eigenvalue**를 계산하고 [논문](https://arxiv.org/abs/1401.7866)에 나온대로 적절히 조합하여 **Shape Strength**을 계산한다.  
이는 주변의 밀도분포를 고려하여 특정 포인트가 어떤 형태를 띄고 있는지 숫자로 나타내준다. 이 과정은 매우 큰 계산자원을 필요로 하기에 **GPU를 이용한 가속화**를 하여 CPU로 처리했을때에 비해 약 10~100배 정도 빠른속도를 구현하였다. 
<br/><br/>
![Signature visualization](https://swha0105.github.io/assets/projects/filament//signature_visual.JPG)  
왼쪽 위부터 시계방향으로 `Density`, `Filament signature`, `Wall signature`, `Cluster signature`를 나타낸다. 
<br/>
<br/>

 
### 3. **Label Spreading을 통한 Filament 식별** -> [code](https://github.com/swha0105/Filament_Project/blob/main/label_spreading_v2.py) 
### [Label spreading에 대한 포스팅](https://swha0105.github.io/_posts/2021-01-21-ML_DL-Label_Spreading.markdown)  
>**Label spreading**은 개수가 적지만 확실한 Label를 통해 Unlabeled 데이터에 Label을 지정 알고리즘으로 **Semi-supervised Learning** 중 하나 이다.    
거대우주구조 4가지 구조중 `Cluster`와 `Void`는 물리량으로 정확히 정의가 된다.   
또한, `Filament`는 `Cluster`와, `Wall`은 `Void`와 물리량 및 기하학적으로 비슷한 특성을 가진다.  이를 이용해 `Cluster`와 `Void`를 X-ray 및 온도를 이용하여 정확히 정의 한 뒤 이를 **Label**로 가정한다. Label의 **기하학적 정보와 물리량들**을 이용해 Unlabeled 데이터 안에 존재하는`Filament`와 `Wall`에 대한 **Classification**을 시도 한다.<br/><br/>
![Gaussian example](https://swha0105.github.io/assets/projects/filament/label_spreading_example.JPG)
<br/>
왼쪽 그림은 밀도, 오른쪽 그림은 Label spreading이후 `Filament` Classification 된 부분을 의미한다.   
<br/>
<br/>



### 4. **MATLAB을 통한 후처리 및 DFS를 통한 Filament 개별화** -> [Skeleton code](https://github.com/swha0105/Filament_Project/blob/main/ref/matlab/skeleton.m), [DFS code](https://github.com/swha0105/Filament_Project/blob/main/find_filament_v2.py)
> 식별된 Filament를 개별화 위해 MATLAB의 `3D Volumetric Image Processing`을 이용해 **Skeletonized**를 하였다.
<br/><br/>
![Skeleton example](https://swha0105.github.io/assets/projects/filament//Skeletonization.png)
<br/>
Skeletonized된 데이터를 이용하여 Python의 모듈 `Networkx`을 이용해 **Graph** 화 하여 **DFS** 개념을 활용해 Cluster 중심에서 부터 Filament가 끝나는 지점까지 길찾기를 하여 필라멘트 개별화를 하였다.

</div>
</details> 


<details>    
<summary> Result & Summary </summary>
<div markdown="1"> 


### 총 40개정도의 cluster에서 segmentation된 105개의 Filament에 대한 물리량들을 구해보았다.  

### **1. Linear density**
먼저, Filament가 잘 segmentation 되었는지 확인하기 위해 구해 볼수 있는 **Linear density**를 구해보았다. 기존 논문들과 10배 가량 차이를 보았지만, 이는 시뮬레이션 환경에서 `암흑물질`을 고려 여부의 차이로 나타나는 것으로 이를 보정하면 충분히 **reliable한 결과**라고 판단 하였다.
<br/>
![Linear Density](https://swha0105.github.io/assets/projects/filament//linear_density.png)
왼쪽은 [기존 논문](https://arxiv.org/abs/1401.7866)의 Linear density이고 오른쪽은 우리가 계산한 Filamnet Linear density의 평균을 나타낸다.  

### **2. Curvature**
Filament가 제대로 식별이 됐다고 판단이 되었으므로 우리의 목적인 **Curvature**를 계산해보기로 하였다. 

3차원상에서 Curvature를 계산하는 수식은 아래와 같다.
![Curvature](https://swha0105.github.io/assets/projects/filament//curvature_equation.png)

총 105개의 Filament의 Mean Curvature, Max Curvature를 구하였고 히스토그램으로 나타내었다.

![Curvature_statistics](https://swha0105.github.io/assets/projects/filament//curvature.png)

위의 데이터를 분석하면 우리가 찾던 Highly curved filament (Curvature > 0.4) 가 존재한다. 하지만 105개중 2개 존재함으로 통계적으로 유효하기 위해 좀 더 많은 데이터가 필요할것으로 보인다.


어떤 필라멘트들이 Highly curvature를 가지는지 알아보기 위해 Length와 함께 데이터를 구성해보았다.

![Curvature_statistics](https://swha0105.github.io/assets/projects/filament//curvature_length.png)

15~20 Mpc/h 의 길이를 가지는 Filament들이 Highly curvature를 가지는 것으로 보였다. 하지만 앞서 언급 했듯이 통계적으로 분석하기 위해 좀 더 많은 데이터가 필요할것으로 보인다.

<br/>

위와 같은 일련의 과정으로 Machine Learning을 도입하여 Large Scale Structure of the universe에서의 Galaxy of Filament를 segmentation 및 Identification하는 작업을 하였다.

결론적으로, Filament를 segmentation 하는 작업은 성공하였으나, 이를 과학적인 결과와 연관시키고 결과가 논문화가 되러면 좀 더 많은 데이터가 필요로 할 것으로 판단이 된다 (Highly curved filament problem). 이 데이터는 굉장히 cost가 비싼 데이터로 하나를 생성하는데 있어 3~4개월이 걸릴 예정이다.

Code works은 끝났으니, 데이터가 생성되는대로 좀 더 테스트를 할 예정이다.




### reference
[1] [https://aip.scitation.org/doi/pdf/10.1063/1.3382336](https://aip.scitation.org/doi/pdf/10.1063/1.3382336)

[2] [https://arxiv.org/abs/1209.2043](https://arxiv.org/abs/1209.2043)

[3] [https://www.semanticscholar.org/paper/A-machine-learning-approach-to-galaxy-LSS-I.-on-Hui-Aragon-Calvo/3376717081ed443ca09c689a261717a3a3675511](https://www.semanticscholar.org/paper/A-machine-learning-approach-to-galaxy-LSS-I.-on-Hui-Aragon-Calvo/3376717081ed443ca09c689a261717a3a3675511)

[4] [https://academic.oup.com/mnras/article/414/1/350/1090746?searchresult=1](https://academic.oup.com/mnras/article/414/1/350/1090746?searchresult=1)

[5] [https://arxiv.org/abs/1611.00437](https://arxiv.org/abs/1611.00437)

[6] [https://arxiv.org/abs/1401.7866](https://arxiv.org/abs/1401.7866)

</div>
</details> 

