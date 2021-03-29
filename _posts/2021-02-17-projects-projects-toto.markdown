---
layout: post
title:  "Prediction of baseball game result by Random forest"
subtitle:  "Legal TOTO"
categories: projects
tags: projects
comments: true
---


# ToTo Project

<br/>

## 개요
2019년 6~7월쯤 예전에 다니던 회사에서 야구좋아하는 친한동료들끼리 언제나처럼 수다떨다가 아이디어가 나왔다.  
야구는 스포츠중 데이터가 잘 정리되어있고 수치화하기 쉬워 이러한 데이터를 가지고 경기 결과를 예측하는 모델을 구축하면 잘 될꺼같다, 이걸로 합법토토인 배트맨토토를 해보자 라는 말이 오고갔었다. 하지만.. 9월에 퇴사하고 울산으로 귀양(?)가는 바람에 아이디어만 오고갔고 제대로 구체화 된건 없었다. 그렇게 잊고 지내다가 반백수처럼 지내는 요즘 이 아이디어를 구체화시킬 기회라 생각하여 작업중에 있다.  
  
2021년 3월 현재 크롤링 코드를 완성했고 데이터베이스 구축을 [QUOPA](https://github.com/QUOPA) 와 함께 작업중에있다.


<br/>


---



## 예상 Feature & Data
1. **각 팀의 최근 결과**
- 최근 N 경기 팀의 승, 패, 실점, 득점

2. **각 팀의 Batting Data**
- 배팅 오더에 맞게 각 선수마다 최근 N 경기 타율, OPS, .. 기타 등등

3. **각 팀의 Pitcher Data**
- 선발투수 최근 N경기 방어율, 볼넷, 실점

5. **Label & Metadata**
- 경기당 득/실점 (Label)
- 선발투수, 배팅오더, 최근 경기결과 등을 모아둔 Metadata

<br/>

- 위의 데이터를 모은 Database 구축 후, **Meta data를 참고하여 경기별 Data 생성** (학습단위)
- Meta data는 학습 시 필요, 실제 예측 단계에서는 파이프라인에서 나오는 라인업 사용.
- 불펜투수는 팀 Data에 속해있다고 가정. 
- 통산기록은 큰 의미없다고 가정. 최근 N 경기가 중요.

<br/>

---

## Sketch

### 데이터 베이스 구축 -> 전처리 -> 모델 구축 -> 파이프라인 구축

<br/>

**1. 데이터 베이스 구축**  
각 팀의 최근 결과, Batting, Picther Data들을 모아둔 데이터 베이스 구축 

- ~~크롤링 코드 작성~~ 
- ~~Html parsing & Data 추출~~
- AWS Database 구축 [QUOPA](https://github.com/QUOPA)

<br/>

**2. 전처리**  
데이터 베이스에서 학습에 필요한 데이터 추출 및 가공

- 필요한 feature 선택하여 경기별로 pairing. 데이터
- 손실 및 결손 데이터 처리 (1년차 신인, 외국인)

<br/>

**3. 모델 구축**  
AWS, 혹은 lightGBM 으로 Random Forest구축. (예상 Feature들이 수치화 잘되어있어 RF을 예상모델로 생각중.)

<br/>

**4. 파이프 라인 구축**  
잘 학습된 모델로, 경기 시작 2시간전 라인업을 크롤링하여 받은다음 그 경기에 대한 득,실점 예측.





