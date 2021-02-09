---
layout: post
title:  "Prediction of baseball game result by Random forest"
subtitle:  "Legal TOTO"
categories: projects
tags: projects
comments: true
---

프로젝트 동기

전에 다니던 회사에서 야구 좋아하는 동료들끼리 이야기 하다가 `야구 데이터 가지고 random forest해서 토토 해보자` 라는 말이 나왔었다. 그 중 친한 동료 한명이 크롤링 코드를 만들어줬지만 이직 준비 때문에 더 진행하지 못햇다..  
요즘 반 백수(?) 처럼 지내는 시기에 다시 한번 해보자는 생각이 들어서 제대로 추친해본다. 

만들어준 크롤링 코드의 크레딧은 당연히 동료에게 있고 모델이 성공적으로 예측 할 경우 소스 코드를 크레딧이 있는 동료들에게 공유할 생각이다.


데이터 생성
1. 모든 선수에 대한 데이터 크롤링
2. Batter input data: 최근 5경기에 대해 평균, 그 선수 통산 평균 .
3. Picther input data.: 최근 3경기에 대한 평균 이닝, 평균 실점. 그 선수 통산 평균 

4. 날짜에 대해 라인업 오더와 선발 투수 pairing
- https://www.koreabaseball.com/Schedule/GameCenter/Main.aspx



