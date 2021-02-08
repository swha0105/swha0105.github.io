---
layout: post
title:  "[ML Algorithms] LSA, SVD"
subtitle:   "Text mining"
categories: ml_dl
tags: ml
comments: true
---

토픽 모델링.
- 문서 집합에 숨어 있는 주제를 찾아내는 텍스트 마이닝.

1. LSA
2. LDA

기존에는 문서 단어행렬 DTM (document term matrix)는 단어의 빈도수만 고려. 하지만 빈도 수 만으로는 주제를 찾기 부족. 따라서 의미를 정확히 고려하기 위해 DTM을 trancated SVD하는 위와같은 방법

빈도수 하면 문제점, 의미없는 단어들이 많음. 

n개의 문서에 단어의 총 개수 m


먼저 SVD에 대해.







<!-- EVD 


(EVD) eigendecomposition 과 달리 sqaure 행렬이 아닐때도 사용 가능.

수학적 정의 

EVD는 

$$ A v = v \lambda $$ 

> v: A에 대한 EigenVector
> $$lambda$$: v에 대한 EigenValue.

대각화.

$$ A = v \D v^{-1}$$ 

> D: \lambda 값을 성분으로 한 대각행렬.


물리적 의미

- 변하지 않는 본질. 
- 회전변환해도 값이 그대로.

예시 -->


**들어가기전..**  





<!-- 

https://wikidocs.net/24949
https://bkshin.tistory.com/entry/NLP-9-%EC%BD%94%EC%82%AC%EC%9D%B8-%EC%9C%A0%EC%82%AC%EB%8F%84%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%98%81%ED%99%94-%EC%B6%94%EC%B2%9C-%EC%8B%9C%EC%8A%A4%ED%85%9C -->





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
