---
layout: post
title:  "[모두연 풀잎스쿨 14기] Chapter 5"
subtitle:   "K means clustering"
categories: dev
tags: spark
comments: true
---

**본 포스팅은 모두의연구소(home.modulabs.co.kr) 풀잎스쿨에서 진행된 `Advanced Analytics with Spark` 과정 내용을 공유 및 정리한 자료입니다.**    

--- 

Chapter 5 제목은 `K-평균 군집화로 네트워크 이상 탐지하기`이다.  

해킹 데이터셋을 받아 공격패턴을 알아내는 챕터이다.  
데이터에 label이 포함되어 있지만 학습할때는 Label을 제외한 데이터를 넣어 Unsupervised learning중 하나인 `Kmeans clustering`을 한다. 

이 챕터는 내가 발표 준비를 했기에 여기 나오는 코드는 본인이 작성했다

<br/>

---

## dataset download & spark setting

```python
from pyspark.sql import SparkSession
import pyspark.sql.functions as F

import os

ref_path = os.getcwd()
data_path = ref_path + '/data/'
# if 'kdd.ics.uci.edu' not in os.listdir(ref_path):
#     !wget -A.gz https://kdd.ics.uci.edu/databases/kddcup99/
#     !wget https://kdd.ics.uci.edu/databases/kddcup99/kddcup.names
#     !gunzip $ref_path'kdd.ics.uci.edu/databases/kddcup99/*'
    
    
spark = SparkSession.builder.appName('Chapter05')\
    .master('local[4]')\
    .config("spark.executor.memory", "2g")\
    .getOrCreate()

!head $data_path'kddcup.data'
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    0,tcp,http,SF,215,45076,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,1,1,0.00,0.00,0.00,0.00,1.00,0.00,0.00,0,0,0.00,0.00,0.00,0.00,0.00,0.00,0.00,0.00,normal.
    0,tcp,http,SF,162,4528,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,2,2,0.00,0.00,0.00,0.00,1.00,0.00,0.00,1,1,1.00,0.00,1.00,0.00,0.00,0.00,0.00,0.00,normal.
    0,tcp,http,SF,236,1228,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,1,1,0.00,0.00,0.00,0.00,1.00,0.00,0.00,2,2,1.00,0.00,0.50,0.00,0.00,0.00,0.00,0.00,normal.
    0,tcp,http,SF,233,2032,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,2,2,0.00,0.00,0.00,0.00,1.00,0.00,0.00,3,3,1.00,0.00,0.33,0.00,0.00,0.00,0.00,0.00,normal.
    0,tcp,http,SF,239,486,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,3,3,0.00,0.00,0.00,0.00,1.00,0.00,0.00,4,4,1.00,0.00,0.25,0.00,0.00,0.00,0.00,0.00,normal.
    0,tcp,http,SF,238,1282,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,4,4,0.00,0.00,0.00,0.00,1.00,0.00,0.00,5,5,1.00,0.00,0.20,0.00,0.00,0.00,0.00,0.00,normal.
    0,tcp,http,SF,235,1337,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,5,5,0.00,0.00,0.00,0.00,1.00,0.00,0.00,6,6,1.00,0.00,0.17,0.00,0.00,0.00,0.00,0.00,normal.
    0,tcp,http,SF,234,1364,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,6,6,0.00,0.00,0.00,0.00,1.00,0.00,0.00,7,7,1.00,0.00,0.14,0.00,0.00,0.00,0.00,0.00,normal.
    0,tcp,http,SF,239,1295,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,7,7,0.00,0.00,0.00,0.00,1.00,0.00,0.00,8,8,1.00,0.00,0.12,0.00,0.00,0.00,0.00,0.00,normal.
    0,tcp,http,SF,181,5450,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,8,8,0.00,0.00,0.00,0.00,1.00,0.00,0.00,9,9,1.00,0.00,0.11,0.00,0.00,0.00,0.00,0.00,normal.

</div>
</details> 

<br/>

```python
dataWithoutHeader = spark\
    .read.format("csv")\
    .option('inferSchema',True)\
    .option("header", False)\
    .option("sep", ",")\
    .load(data_path + "kddcup.data")

cols = []

name = open(data_path + 'kddcup.names.txt','r')
lines = name.readlines()
print(lines)
for line in lines[1:]:
    cols.append(line.split(':')[0])

cols.append('label')
print(cols)
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    ['back,buffer_overflow,ftp_write,guess_passwd,imap,ipsweep,land,loadmodule,multihop,neptune,nmap,normal,perl,phf,pod,portsweep,rootkit,satan,smurf,spy,teardrop,warezclient,warezmaster.\n', 'duration: continuous.\n', 'protocol_type: symbolic.\n', 'service: symbolic.\n', 'flag: symbolic.\n', 'src_bytes: continuous.\n', 'dst_bytes: continuous.\n', 'land: symbolic.\n', 'wrong_fragment: continuous.\n', 'urgent: continuous.\n', 'hot: continuous.\n', 'num_failed_logins: continuous.\n', 'logged_in: symbolic.\n', 'num_compromised: continuous.\n', 'root_shell: continuous.\n', 'su_attempted: continuous.\n', 'num_root: continuous.\n', 'num_file_creations: continuous.\n', 'num_shells: continuous.\n', 'num_access_files: continuous.\n', 'num_outbound_cmds: continuous.\n', 'is_host_login: symbolic.\n', 'is_guest_login: symbolic.\n', 'count: continuous.\n', 'srv_count: continuous.\n', 'serror_rate: continuous.\n', 'srv_serror_rate: continuous.\n', 'rerror_rate: continuous.\n', 'srv_rerror_rate: continuous.\n', 'same_srv_rate: continuous.\n', 'diff_srv_rate: continuous.\n', 'srv_diff_host_rate: continuous.\n', 'dst_host_count: continuous.\n', 'dst_host_srv_count: continuous.\n', 'dst_host_same_srv_rate: continuous.\n', 'dst_host_diff_srv_rate: continuous.\n', 'dst_host_same_src_port_rate: continuous.\n', 'dst_host_srv_diff_host_rate: continuous.\n', 'dst_host_serror_rate: continuous.\n', 'dst_host_srv_serror_rate: continuous.\n', 'dst_host_rerror_rate: continuous.\n', 'dst_host_srv_rerror_rate: continuous.\n']


</div>
</details> 

- 첫번째 줄에는 label이 명시되어있다. 따라서 header를 만들기 위해 첫번째 줄은 제외한다.
- 2번째 줄부터는 [feature: data type]으로 구성 되어있다.

<br/>

```python
data = dataWithoutHeader.toDF(*cols)
data.cache()
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 


    DataFrame[duration: int, protocol_type: string, service: string, flag: string, src_bytes: int, dst_bytes: int, land: int, wrong_fragment: int, urgent: int, hot: int, num_failed_logins: int, logged_in: int, num_compromised: int, root_shell: int, su_attempted: int, num_root: int, num_file_creations: int, num_shells: int, num_access_files: int, num_outbound_cmds: int, is_host_login: int, is_guest_login: int, count: int, srv_count: int, serror_rate: double, srv_serror_rate: double, rerror_rate: double, srv_rerror_rate: double, same_srv_rate: double, diff_srv_rate: double, srv_diff_host_rate: double, dst_host_count: int, dst_host_srv_count: int, dst_host_same_srv_rate: double, dst_host_diff_srv_rate: double, dst_host_same_src_port_rate: double, dst_host_srv_diff_host_rate: double, dst_host_serror_rate: double, dst_host_srv_serror_rate: double, dst_host_rerror_rate: double, dst_host_srv_rerror_rate: double, label: string]


</div>
</details> 

<br/>

- 데이터에 header를 붙이는 작업. 
- Inferscheme 했기에 Scheme의 타입이 제대로 설정되어있는지 확인


```python
data.groupBy("label").count().orderBy('count',ascending=False).show()
```

    +----------------+-------+
    |           label|  count|
    +----------------+-------+
    |          smurf.|2807886|
    |        neptune.|1072017|
    |         normal.| 972781|
    |          satan.|  15892|
    |        ipsweep.|  12481|
    |      portsweep.|  10413|
    |           nmap.|   2316|
    |           back.|   2203|
    |    warezclient.|   1020|
    |       teardrop.|    979|
    |            pod.|    264|
    |   guess_passwd.|     53|
    |buffer_overflow.|     30|
    |           land.|     21|
    |    warezmaster.|     20|
    |           imap.|     12|
    |        rootkit.|     10|
    |     loadmodule.|      9|
    |      ftp_write.|      8|
    |       multihop.|      7|
    +----------------+-------+
    only showing top 20 rows
    

<br/>

---

## Model Pipeline 


```python
import numpy as np
from pyspark.ml.feature import VectorAssembler
from pyspark.ml.clustering import KMeans,BisectingKMeans
from pyspark.ml import Pipeline
import pyspark.sql.functions as f
from pyspark.ml.evaluation import ClusteringEvaluator

numericOnly = data.drop("protocol_type","service","flag").cache()

InputCols =[columns for columns in numericOnly.columns if columns != 'label']


assembler = VectorAssembler()\
    .setInputCols(InputCols)\
    .setOutputCol("featureVector")

kmeans = BisectingKMeans()\
    .setK(2)\
    .setPredictionCol("cluster")\
    .setFeaturesCol("featureVector")

pipeline = Pipeline(stages=[assembler,kmeans])
piplelinemodel = pipeline.fit(numericOnly)
result_data = piplelinemodel.transform(numericOnly)
```

---

### pipeline 
- Pipeline은 stage의 연속, stage는 Transformer와 Estimator로 구성. 
- pipeline은 estimator로 fit함수 호출을 통해 학습을 진행할 수 있고 결과로 pipelinemodel을 반환한다. 

### transformer 
- feature를 변형, 추출하기 위한 알고리즘과 학습된 모델을 위한 추상 클래스 또는 구현한 클래스 집합이다.
- transformer클래스들은 내부적으로 transform함수가 구현되어 있다.
- DataFrame을 입력받아 하나 이상의 칼럼이 추가된 새로운 DataFrame을 만들어낸다

### Estimators
- 데이터로 알고리즘을 학습하는 과정을 추상화한 클래스 또는 구현한 클래스 집합이다.
- estimator클래스들은 내부적으로 fit함수가 구현되어있다
- DataFrame을 입력받아 Transformer인 모델을 반환한다.
> ex: estimator class인 LogisticRegression은 fit함수 호출을 통해 학습된 LogisticRegressionModel을 반환하며 반환되는 LogisticRegressionModel은 Transformer이다. 

<br/>

### Silhouette score (clustering metric)
$$ -1 \leq \frac{ (b_{i}-a_{i})}{max(a_{i},b_{i})} \leq 1 $$
 
> Intra-cluster distance: $$a(i) = \frac{1}{C_{i}-1} \sum_{j \subset C_{i}, i \neq j} d(i,j)$$   
> mean nearest-cluster distance: $$b(i) = \min_{k \neq i} \frac{1}{C_{k}} \sum_{j \subset C_{k}} d(i,j)$$  
> distance: $$d(i,j)$$  
> number of object in cluster C: $$ C_{i}:  i$$

<br/>

### SC (Silhouette score) 해석 
Range of SC Interpretation  
0.71-1.0 A strong structure has been found  
0.51-0.70 A reasonable structure has been found  
0.26-0.50 The structure is weak and could be artificial  
< 0.25 No substantial structure has been found  

[출처 1](https://spark.apache.org/docs/latest/ml-pipeline.html#properties-of-pipeline-components) [출처 2](https://www.stat.berkeley.edu/~spector/s133/Clus.html)

<br/>

<!-- Kmeans는 모든 iteration step마다 distance를 계산하여 clustering,  
Bisecting clustering은 가장 Intra-cluster distance가 높은 cluster를 픽 하여 나눈다.  -->


---


```python
result_data.select("cluster","label")\
    .groupBy("Cluster","label").count()\
    .orderBy('count',ascending=False)\
    .show(25)
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 


    +-------+----------------+-------+
    |Cluster|           label|  count|
    +-------+----------------+-------+
    |      0|          smurf.|2807886|
    |      0|        neptune.|1072017|
    |      0|         normal.| 972781|
    |      0|          satan.|  15892|
    |      0|        ipsweep.|  12481|
    |      0|      portsweep.|  10408|
    |      0|           nmap.|   2316|
    |      0|           back.|   2203|
    |      0|    warezclient.|   1020|
    |      0|       teardrop.|    979|
    |      0|            pod.|    264|
    |      0|   guess_passwd.|     53|
    |      0|buffer_overflow.|     30|
    |      0|           land.|     21|
    |      0|    warezmaster.|     20|
    |      0|           imap.|     12|
    |      0|        rootkit.|     10|
    |      0|     loadmodule.|      9|
    |      0|      ftp_write.|      8|
    |      0|       multihop.|      7|
    |      1|      portsweep.|      5|
    |      0|            phf.|      4|
    |      0|           perl.|      3|
    |      0|            spy.|      2|
    +-------+----------------+-------+
    
</div>
</details>  

- 책의 예제대로 cluster의 군집의 갯수를 k=2로 했을때는 결과가 좋지 않다.

<br/>

## 최적의 K 찾기

```python
import numpy as np
from pyspark.ml.feature import VectorAssembler
from pyspark.ml.clustering import KMeans,BisectingKMeans
from pyspark.ml import Pipeline
import pyspark.sql.functions as f
from pyspark.ml.evaluation import ClusteringEvaluator
numericOnly = data.drop("protocol_type","service","flag").cache()

InputCols =[columns for columns in numericOnly.columns if columns != 'label']

cost = []
for k in [5,10,15,20]:

#for k in [20]:

    assembler = VectorAssembler()\
        .setInputCols(InputCols)\
        .setOutputCol("features")

    kmeans = KMeans()\
        .setSeed(np.random.randint(42))\
        .setK(k)\
        .setPredictionCol("cluster")\
        .setFeaturesCol("features")

 #init='k-means++'

    pipeline = Pipeline().setStages([assembler,kmeans])
    pipelinemodel = pipeline.fit(numericOnly)
    result_data = pipelinemodel.transform(numericOnly)

    evaluator = ClusteringEvaluator()
    evaluator.setPredictionCol("cluster")
    cost.append(evaluator.evaluate(result_data))
    print(k,(evaluator.evaluate(result_data)))
    #print(k, evaluator.evaluate(result_data)) #자동으로 "features" 만 찾아줌 
```

    5 0.9999994341550521
    10 0.99996839424901
    15 0.9999231883576829
    20 0.999933043370616

<br/>

---

## 마무리
- 왜 최적의 K 가 안찾아지는지 아무도 알아내지 못했다.. 아마 데이터가 책에 나온 형태와 다른것으로 추론됨
- pipeline의 구성, Transformer, Estimator개념에 대해 알아보았다.
- K-clustering 평가 방법 silhouette score에 대해 알아보았다.

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