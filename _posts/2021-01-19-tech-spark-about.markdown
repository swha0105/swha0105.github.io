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


위의 정의에 따르면 데이터의 크기, 데이터 구조의 다양성, 데이터 인풋-아웃풋의 처리 속도 등 다양한 요소등을 고려하여 빅데이터를 정의한다. 

Variety 영역은 제처두고, 엄청난 Volume의 데이터를 Velocity를 만족시키면서 처리하려면 데이터에 대한 효율적인 연산이 필요하다. 
기술자들은 효율적인 연산을 위해 컴퓨터에서 일어나는 많은 작업들 중 가장 느린, 즉 병목 현상이 일어나는 I/O단계를 주목했다.

특히 슈퍼컴퓨터나 클러스터에서는 그 용량과 가격때문에 SSD를 사용하지 못하기에 여전히 HDD를 사용하는데 HDD는 물리적으로 자성을 이용해 데이터를 기록하기 때문에 속도에 한계가 있다.  또한 HDD에 대한 Access은 병렬적으로 접근할 수 없기에 크기가 **매우 큰 데이터에 대한 I/O는 굉장히 느릴 수 밖에 없다.**

이 한계를 타파하기 위해 `HDFS`의 개념이 나온다.

<br/>

---

# HDFS (Hadoop Distributed File System)

HDFS는 하나의 파일을 여러개의 컴퓨터에 분산해서 저장하는 시스템을 말한다.  
좀 더 자세히 설명하자면 `HDFS`는 `Primary-Secondary` 의 구조를 가지고 있다. (통상적으로 `Master-Slave`용어를 사용하지만 이 용어는 거부감이 들어 최근 들어 많이 사용되는 용어로 대체함) 

`Primary-Secondary` 구조는 보통 하나의 `Primary` 노드가 `Secondary` 노드들에게 연산, 데이터들을 뿌려주는 형태를 말한다.

**`HDFS`** 구조안에서 Primary와 Secondary는 다음의 역할을 한다.   
- `Primary`노드는 하나의 `Namenode` 이고 metadata들을 관리한다.  
- `Secondary`노드는 `Datanode`이고 데이터들을 저장하는 노드들이다. 

|![HDFS_architecture](https://swha0105.github.io/assets/tech/HDFS_architecture.JPG)  
|:--:| 
| 출처: Tacademy 교육 |

`Datanode`에게 이렇게 분산처리 

## HDFS Blocks
여기까지는 보통 
<!-- 내가 개인적으로 사용했던 슈퍼컴퓨터는 데이터 저장 및 관리는 `MasPrimaryter` 에서만, 연산은 `Secondary`에서 시행하였지만  -->


