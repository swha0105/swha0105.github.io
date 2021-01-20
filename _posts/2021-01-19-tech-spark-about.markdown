---
layout: post
title:  "[Spark] About Spark"
subtitle:   "분산 처리 클러스터와 Spark의 기본 개념 "
categories: tech
tags: spark
comments: true
---

# 빅데이터와 분산 처리 필요성 대두

<br/>

<!-- 

글로벌 컨설팅업체 IDC 분석에 따르면 2019년 전세계 데이터 총량은 40ZB (4*10^13 GB)으로 천문학적인 수치이다. ~~나의 석사학위 천체물리학..~~ -->
바야흐로 빅데이터 시대이다.  그런데 빅데이터가 뭘까??  
`빅데이터` 단어는 굉장히 많이 쓰이지만 정확한 정의는 딱히 내려져 있지 않다.  
찾아본 결과 그나마 가장 합당하다고 생각되는 정의를 하나 소개한다.

|![Time table](https://swha0105.github.io/assets/tech/big_data.JPG)  
|:--:| 
| 출처: Tacademy 교육 |


위의 정의에 따르면 **데이터의 크기**, **데이터 구조의 다양성**, **데이터 인풋-아웃풋의 처리 속도** 등 다양한 요소등을 고려하여 빅데이터를 정의한다. 

Variety 영역은 제처두고, 엄청난 Volume의 데이터를 Velocity를 만족시키면서 처리하려면 데이터에 대한 효율적인 연산이 필요하다. 
기술자들은 효율적인 연산을 위해 컴퓨터에서 일어나는 많은 작업들 중 가장 느린, 즉 병목 현상이 일어나는 I/O단계를 주목했다.

특히 슈퍼컴퓨터나 클러스터에서는 그 용량과 가격때문에 SSD를 사용하지 못하기에 여전히 HDD를 사용하는데 HDD는 물리적으로 자성을 이용해 데이터를 기록하기 때문에 속도에 한계가 있다.  또한 HDD에 대한 Access은 병렬적으로 접근할 수 없기에 크기가 **매우 큰 데이터에 대한 I/O는 굉장히 느릴 수 밖에 없다.**

이 한계를 타파하기 위해 `HDFS`의 개념이 나온다.

<br/>

---

# HDFS (Hadoop Distributed File System)

HDFS는 하나의 파일을 여러개의 컴퓨터에 분산해서 저장하는 시스템을 말한다.  
좀 더 자세히 설명하자면 `HDFS`는 `Primary-Secondary` 의 구조를 가지고 있다.  
(통상적으로 `Master-Slave`용어를 사용하지만 이 용어는 거부감이 들어 최근 들어 많이 사용되는 용어로 대체함) 

`Primary-Secondary` 구조는 보통 하나의 `Primary` 노드가 `Secondary` 노드들에게 연산, 데이터들을 뿌려주는 형태를 말한다.

**`HDFS`** 구조안에서 Primary와 Secondary는 다음의 역할을 한다.   
- `Primary`노드는 하나의 `Namenode` 이고 metadata들을 관리한다.  
- `Secondary`노드는 `Datanode`이고 데이터들을 저장하는 노드들이다. 

|![HDFS_architecture](https://swha0105.github.io/assets/tech/HDFS_architecture.JPG)  
|:--:| 
| 출처: Tacademy 교육 |




## HDFS Blocks
큰 데이터를 분산해서 `Datanode`에 저장할때 나누는 단위 블락을 **`HDFS Blocks`** 이라고 한다. 이 블락은 기본적으로 64MB,128MB의 단위를 가질수있고 가변 가능하다.

`HDFS Blocks`을 이용하여 큰 데이터를 분산 저장하고 여러 노드들에서 동시에 접근하는 건 좋다.  
하지만 만약 많은 노드들 중 하나가 어떤 이유로 결함이 생기면 어떻게 될까?   
노드가 하나만 고장나도 큰 데이터에 접근을 못하게 되는데 너무 리스크가 큰게 아닌가? 라고 생각 할 수 있다. 


이러한 상황을 방지하기 위해 Blocks들은 복제 (Replication)를 하여 다른 노드들에 저장을 한다. 보통 Blocks들은 3개의 Replication을 가지고 각기 다른 노드에 저장을 한다. Blocks의 복제버전이 있다면 하나의 머신에서 OS fault나 Hardware issue로 작동하지 않더라도 다른 노드들이 백업을 할 수 있다.  
따라서 HDFS는 Blocks의 복제를 통해 **내결함성(fault tolerance)**과 **가용성(availability)**을 높인다.


<br/>

***

# Spark
HDFS는 빅데이터를 저장하고 관리하는데 매우 큰 도움이 된다.  
큰 파일을 여러 노드에 나누어 저장하여 HDD에 동시에 접근하는 효과를 내어 접근속도의 한계를 많이 완화 시켰다.  

그런데 여전히 HDD는 느리다.. 아무리 데이터에 동시에 접근하는 효과를 내어도 HDD는 정말 ~~속터지게~~ 느리다. 써본사람은 공감할 것이다 ㅠ_ㅠ 

그래서 2009년에 UC berkeley에서 이런생각을 했나보다.  

    이거 HDD 너무 느려터졌는데 Memory에 올려놓고 처리하면 안되냐? 요즘 메모리 가격도 싸고 용량도 꽤 큰데?

**Memory** 에 올려놓고 데이터 처리를 하면 병목단계인 I/O단계를 거치지 않아도 되고 엄청 빨라 지겠네? 그전까지는 Memory의 가격도 비싸고 용량도 한계가 있어 이런 접근이 무리였지만 저 시기부터 메모리 가격이 엄청싸졌다(고 한다). 이러한 이유로 Spark가 등장하게 되었다. 

### Spark란 클러스터의 여러 노드로 프로그램을 분배하고, 그 위에서 동작하는 프로그램을 개발할 수 있도록 개발된 오픈 소스 프레임워크이다 
(ref.3)

<br/>

## Spark 구조

|![Spark_architecture](https://swha0105.github.io/assets/tech/spark_architecture.JPG)  
|:--:| 
| 출처: Tacademy 교육 |


Spark도 `Primary-Secondary` 의 구조를 가진다.   
`Primary`에 해당하는 노드에서 돌아가는 **`Driver Process`**    
`Secondary`에서 돌아가는 **`Excutors`** 가 있다.  

**Driver Process**: Spark session을 관리하는 JVM(Java Virutal Machine) 프로세스. DAG(Directed Acyclic Graph) 기반 테스크 스케쥴링을 수행하고 Excutors을 관리함.

**Excutors**: Driver Process에서 요청한 연산 수행.

<br/>

## RDD(Resilient Distributed Dataset)
 - **여러 노드에 저장되는 변경이 불가능한 데이터의 집합**

Spark는 데이터의 기본 단위인 **`RDD`** 을 생성하고 연산한다.   
또한 `RDD`는 데이터 처리 기본 단위인 여러개의 `파티션`으로 분리가 되고 파티션 끼리는 병렬진행, 파티션 안에서는 순차적 실행을 한다.

`RDD`는 두개의 Operation만을 지원한다.
- **Transformation**: RDD를 필터링하거나 변환하여 새로운 RDD 를 리턴함. 

- **Action**: RDD을 기반으로 계산을 하고 최종결과를 리턴하거나 데이터를 저장함.

Transformation은 Action이 나오기 전에 Spark에서 계산을 하지 않는다. 마치 설계도를 짜는것 처럼 어떻게 계산을 수행 해야 할것인지 설계만 하고 Action후에 모든 계산이 실행이 된다. 이와같은 연산을 `Lazy operation`이라 한다.


<br/>

***

## 마무리 하며..

이러한 분산시스템은 공부한지 얼마 안되어 이 글에 부족한점이 많을꺼같다.  
대학원생때 슈퍼컴퓨터를 ~~신물나게~~ 많이 사용했지만 Ganglia나 Sun grid engine을 사용했기 때문에 이러한 데이터를 메모리 단계에서 처리하는게 조금 어색하다. (이게 가능하다는게 아직 실감이 안난다..)

그래서 이 글은 배움이 늘어남에 따라 계속 수정할 예정이다..


*** 

## Ref   
1. Tacademy 교육  
2. 9가지 사례로 익히는 고급 스파크 분석 
3. https://gengmi.tistory.com/entry/%EC%8A%A4%ED%8C%8C%ED%81%ACSpark%EB%9E%80  
4. https://sshkim.tistory.com/
5. https://bcho.tistory.com/
