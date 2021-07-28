---
layout: post
title:  "[모두연 풀잎스쿨 14기] Chapter 3"
subtitle:   "추천 알고리즘"
categories: dev
tags: spark
comments: true
---

**본 포스팅은 모두의연구소(home.modulabs.co.kr) 풀잎스쿨에서 진행된 `Advanced Analytics with Spark` 과정 내용을 공유 및 정리한 자료입니다.**  

---

Chapter 3 제목은 `음악 추천과 Audioscrobbler 데이터셋`이다.  
음악 감상 데이터셋을 받아 추천 알고리즘을 사용하여 특정 유저에 대한 추천 아티스트들을 보여주는 실습이다. ~~그냥 유튜브 생각하면 쉽다~~

여기서 사용된 추천 알고리즘은 ALS(Alternating Least Square)이고 관련된 자세한 내용은 [여기](https://swha0105.github.io/_posts/2021-01-27-ML_DL-ml-ALS.markdown) 정리 해놓았다.

<br/>

***

## Data set
Auhdioscrobbler에서 만든 데이터 셋이고 https://www.last.fm/ 에 추천 시스템을 구축하는데 사용 되었다. 총 3개의 데이터로 구성되어있고 모두 txt 파일이다.    

1. **user_artist_data**: userid  artistid  playcount 
    - user의 artist에 대한 재생 정보를 나타냄.  
    - empty space로 칼럼 구분
    - integer type으로 구성되어 있음.

2. **artist_data.txt**: artistid  artist_name

3. **artist_alias.txt**: badid  goodid
   - Artist들의 이름이 통일 되어있지 않음으로 여러 이름들의 형태를 저장

<br/>

***

# Code


**Referrence**
- ALS 알고리즘 설명 : https://ggoals.github.io/Spark_ALS_Algorithm_tuning/
- Towards data science : https://towardsdatascience.com/build-recommendation-system-with-pyspark-using-alternating-least-squares-als-matrix-factorisation-ebe1ad2e7679
- Spark tutorial : https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
- 코드 원본 : https://github.com/dream2globe/advanced-spark



```python
!wget https://storage.googleapis.com/aas-data-sets/profiledata_06-May-2005.tar.gz
!tar xvf ./data/profiledata_06-May-2005.tar.gz

```
       
<br/>

## Spark session


```python
import gc
import logging
import subprocess
from datetime import datetime
from pathlib import Path

import pyspark.sql.functions as F
import pyspark.sql.types as T
from pyspark.sql import Row
from pyspark.sql import SparkSession
from pyspark.sql.window import Window
# from pytz import timezone
# from pytz import utc

EXECUTOR_MEMORY = "2g"
EXECUTOR_CORES = 2
EXECUTORE_INSTANCES = 3
DRIVER_MEMORY = "1g"
DRIVER_MAX_RESULT_SIZE = "1g"

spark = (
    SparkSession.builder.appName(f"Advanced analytics with SPARK - Chapter 3")
    .config("spark.executor.memory", EXECUTOR_MEMORY)
    .config("spark.executor.cores", EXECUTOR_CORES)
    .config("spark.executor.instances", EXECUTORE_INSTANCES)
    .config("spark.driver.memory", DRIVER_MEMORY)
    .config("spark.driver.maxResultSize", DRIVER_MAX_RESULT_SIZE)
    .config("spark.kryoserializer.buffer.max", "1024m")
#     .config("spark.sql.warehouse.dir", "/user/bigdata/members/shyeon/advanced-spark/data")
    .enableHiveSupport()
    .getOrCreate()
)

```

---

<details>
<summary> 실행결과 </summary>
<div markdown="1">   
[('spark.executor.memory', '2g'),  
('spark.executor.instances', '3'),  
('spark.driver.host', '0b63c6cfbaf6'),  
('spark.driver.extraJavaOptions',  
'"-Dio.netty.tryReflectionSetAccessible=true"'),  
('spark.app.id', 'local-1611128012143'),  
('spark.driver.port', '33195'),  
('spark.kryoserializer.buffer.max', '1024m'),  
('spark.executor.id', 'driver'),  
('spark.driver.maxResultSize', '1g'),  
('spark.driver.memory', '1g'),  
('spark.executor.cores', '2'),  
('spark.executor.extraJavaOptions',  
'"-Dio.netty.tryReflectionSetAccessible=true"'),  
('spark.sql.catalogImplementation', 'hive'),  
('spark.rdd.compress', 'True'),  
('spark.app.name', 'Advanced analytics with SPARK - Chapter 3'),  
('spark.serializer.objectStreamReset', '100'),  
('spark.master', 'local[*]'),  
('spark.submit.pyFiles', ''),  
('spark.submit.deployMode', 'client'),  
('spark.ui.showConsoleProgress', 'true')]  

</div>
</details> 

<br/>


Spark 기본 설정을 해주는 코드.  

Executor_instance: = --num-executor와 같음  executor의 갯수 설정  
Executor_cores: 각 executor가 사용하는 thread의 갯수

---

<br/>


## Load dataset and Preprocessing

---

```python
!head /home/jovyan/work/ch03/profiledata_06-May-2005/user_artist_data.txt
```
<details>
<summary> 데이터 구조 </summary>
<div markdown="1">   
    1000002 1 55
    1000002 1000006 33
    1000002 1000007 8
    1000002 1000009 144
    1000002 1000010 314
    1000002 1000013 8
    1000002 1000014 42
    1000002 1000017 69
    1000002 1000024 329
    1000002 1000025 1
    
</div>
</details>    

```python
!head /home/jovyan/work/ch03/profiledata_06-May-2005/artist_data.txt
```
<details>
<summary> 데이터 구조 </summary>
<div markdown="1">   
    1134999	06Crazy Life
    6821360	Pang Nakarin
    10113088	Terfel, Bartoli- Mozart: Don
    10151459	The Flaming Sidebur
    6826647	Bodenstandig 3000
    10186265	Jota Quest e Ivete Sangalo
    6828986	Toto_XX (1977
    10236364	U.S Bombs -
    1135000	artist formaly know as Mat
    10299728	Kassierer - Musik für beide Ohren

</div>
</details>    


```python
!head /home/jovyan/work/ch03/profiledata_06-May-2005/artist_alias.txt
```
<details>
<summary> 데이터 구조 </summary>
<div markdown="1">   
    1092764	1000311
    1095122	1000557
    6708070	1007267
    10088054	1042317
    1195917	1042317
    1112006	1000557
    1187350	1294511
    1116694	1327092
    6793225	1042317
    1079959	1000557

</div>
</details>    

<br/>

---
**data load 및 schema생성**

```python
user_artist_schema = T.StructType([
    T.StructField("userid", T.IntegerType(), True),
    T.StructField("artistid", T.IntegerType(), True),
    T.StructField("playcount", T.IntegerType(), True),
])

user_artist_df = (
    spark
    .read.format("csv")
    .option("header", False)
    .option("sep", " ")
    .schema(user_artist_schema)
    .load(data_path + "user_artist_data.txt")
)

user_artist_df.show(5)

artist_schema = T.StructType([
    T.StructField("artistid", T.IntegerType(), True),
    T.StructField("artistname", T.StringType(), True),
])

artist_df = (
    spark
    .read.format("csv")
    .option("header", False)
    .option("sep", "\t")
    .schema(artist_schema)
    .load(data_path + "artist_data.txt")
)

artist_df.show(5, False)

artist_alias_schema = T.StructType([
    T.StructField("badid", T.IntegerType(), True),
    T.StructField("goodid", T.IntegerType(), True),
])

artist_alias_df = (
    spark
    .read.format("csv")
    .option("header", False)
    .option("sep", "\t")
    .schema(artist_alias_schema)
     .load(data_path + "artist_alias.txt")
)

artist_alias_df.show(5, False)
```
---

- Schema를 만든 다음에 data를 load하는 습관.
- option("header"): data의 첫 row가 header경우 True로 반환한다.  
- option("sep"): data끼리의 간격을 지정한다. 이 경우 Space  

---

**읽은 데이터들을 병합하는 부분**

```python
new_user_artist_df = (
    user_artist_df
    .join(F.broadcast(artist_alias_df), user_artist_df.artistid == artist_alias_df.badid, "left")
    .withColumn("artistid", F.when(F.col("badid").isNull(),  F.col("artistid")).otherwise(F.col("goodid")))
    .where(F.col("badid").isNotNull())
    .cache()
)

new_user_artist_df.show()

```

<details>
<summary> 데이터 구조 </summary>
<div markdown="1">   

    +-------+--------+---------+-------+-------+
    | userid|artistid|playcount|  badid| goodid|
    +-------+--------+---------+-------+-------+
    |1000002| 1000518|       89|1000434|1000518|
    |1000002| 1001514|        1|1000762|1001514|
    |1000002|     721|        1|1001220|    721|
    |1000002| 1034635|        5|1001410|1034635|
    |1000002|    3066|        1|1002498|   3066|
    |1000002| 6691692|        1|1003377|6691692|
    |1000002| 1237611|        1|1003633|1237611|
    |1000002| 1034635|        4|1006102|1034635|
    |1000002| 1001172|        1|1007652|1001172|
    |1000002| 1008391|        2|1010219|1008391|
    |1000002| 2006683|        1|1017405|2006683|
    |1000002| 1000840|        2|1059598|1000840|
    |1000002| 2058809|        2|   3197|2058809|
    |1000002| 1066440|       76|   5702|1066440|
    |1000002| 2003588|        2|    709|2003588|
    |1000019| 1239413|        1|1000287|1239413|
    |1000019| 2001739|        1|1000586|2001739|
    |1000019| 1247540|        6|1000943|1247540|
    |1000019| 1049809|        4|1001379|1049809|
    |1000019|    4377|        1|1002143|   4377|
    +-------+--------+---------+-------+-------+
    only showing top 20 rows
    
 </div>
</details>    


---


- withColumn("artist")(F.when..)
    > "artist" column을 대체 한다.   
    F.col("badid").isNull() 만족하면,  F.col("artistid")로 대체, F.col("badid").isNull() 이 아니면 F.col("goodid")
- cache 함수를 적용하면 Storage 탭에서 메모리 사용량을 알 수 있음

<br/>

### Join의 종류

Spark에는 여러 join의 형태가 있다. 그 중 일부만 정리했다.
1. Inner join: 두 데이터에 공통적으로 존재하지 않으면 지우고 condition을 만족하는것만 return (default)
2. Left join: left에 있는 모든 데이터를 기준으로, right 데이터들이 condition을 만족하면 값을 이어 붙이고 만족하지 못하면 null 을 붙임.  
3. right join: right에 있는 모든 데이터를 기준으로, left 데이터들이 condition을 만족하면 값을 이어 붙이고 만족하지 못하면 null 을 붙임. 
4. outer join: 양쪽 데이터를 condition에 관계없이 다 붙인다.

|![join 종류](https://upload.wikimedia.org/wikipedia/commons/9/9d/SQL_Joins.svg)   
|:--:| 
| [출처](https://upload.wikimedia.org/wikipedia/commons/9/9d/SQL_Joins.svg) |




<br>

---

## Build Model (Spark Tutorial)


```python
from pyspark.ml.evaluation import RegressionEvaluator
from pyspark.ml.recommendation import ALS 

als = ALS(seed=42,
          implicitPrefs=False, # Explicit vs Implicit
          rank=10,
          regParam=0.01,
          alpha=1.0,
          maxIter=5,
          userCol="userid", itemCol="artistid", ratingCol="playcount",
          coldStartStrategy="drop")

(train, test) = new_user_artist_df.randomSplit([0.8, 0.2])
als_model = als.fit(new_user_artist_df)

predictions = als_model.transform(test)
evaluator = RegressionEvaluator(metricName="rmse", labelCol="playcount", predictionCol="prediction")
rmse = evaluator.evaluate(predictions)

# Generate top 10 movie recommendations for each user
userRecs = als_model.recommendForAllUsers(10)
# Generate top 10 user recommendations for each movie
movieRecs = als_model.recommendForAllItems(10)

# Show recomeadations
# userRecs.show(n=10, truncate=False)
# movieRecs.show(n=10, truncate=False)

# Generate top 10 movie recommendations for a specified set of users
users = new_user_artist_df.select(als.getUserCol()).distinct().limit(3)
userSubsetRecs = als_model.recommendForUserSubset(users, 10)

print("Root-mean-square error = " + str(rmse))
users.show(n=3, truncate=False)
userSubsetRecs.show(n=3, truncate=False)
```

<details>
<summary> 실행결과 </summary>
<div markdown="1">   

    Root-mean-square error = 11.475695888786678


    +-------+
    |userid |
    +-------+
    |2130958|
    |2131738|
    |2132906|
    +-------+
    
    +-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |userid |recommendations                                                                                                                                                                                                            |
    +-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |1000484|[[1279475, 509.51498], [4423, 285.59818], [1254487, 198.77986], [1002128, 174.14009], [2134114, 156.99306], [1308254, 120.996124], [1061239, 114.24648], [1086065, 110.887115], [1240919, 110.20026], [1082068, 110.14881]]|
    |1000584|[[6899306, 180.5835], [1279475, 120.61687], [1123047, 113.81703], [1280614, 112.68537], [1313603, 97.26634], [1055411, 94.64791], [1101322, 92.81742], [6846841, 79.280426], [10073912, 78.5072], [4371, 77.58006]]        |
    |1000509|[[1017916, 633.2707], [2043183, 617.6902], [6799188, 440.63922], [1101322, 329.008], [1001017, 271.35806], [2036225, 232.64966], [1248506, 231.83801], [7034181, 212.84283], [7018959, 210.13376], [1237708, 204.71098]]   |
    +-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    
</div>    
</details>


- pyspark ml library에 있는 ALS모델 구축.
- 위 데이터는 User가 특정 artist의 음악을 play한 횟수를 카운트 하는 측면에서는 explicit 하지만, artist의 음악을 play 한다고 해서 그 artist를 선호한다고 단정지을 수 없고 간접적으로 추론만 가능하니 implicit 데이터 라고 볼 수 있다.
- 그런데 왜 implicitPrefs 해서 성능이 잘나오는지는 잘모르겠다..

<br/>


```python
# Generate top 10 user recommendations for a specified set of movies
movies = new_user_artist_df.select(als.getItemCol()).distinct().limit(3)
movieSubSetRecs = als_model.recommendForItemSubset(movies, 10)

movies.show(n=3, truncate=False)
movieSubSetRecs.show(n=3, truncate=False)
```
<details>
<summary> 실행결과 </summary>
<div markdown="1">   
    +--------+
    |artistid|
    +--------+
    |1239413 |
    |1001129 |
    |1239554 |
    +--------+
    
    +--------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |artistid|recommendations                                                                                                                                                                                                           |
    +--------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |1239413 |[[2088690, 452.4639], [2027150, 441.07208], [1036787, 136.33311], [1060553, 127.61982], [1031451, 123.03507], [2288164, 117.47447], [1077099, 117.15578], [1047508, 110.3871], [2188843, 106.95569], [1070204, 106.48883]]|
    |1239554 |[[1077099, 957.29675], [2213427, 686.50806], [1031723, 643.5707], [2019846, 614.0945], [1058542, 552.22235], [1073616, 512.73346], [1064240, 511.27615], [1010177, 495.19293], [2042801, 492.02536], [2248675, 471.95676]]|
    |1001129 |[[1041675, 6512.148], [2027150, 6130.94], [1036787, 4110.981], [2009645, 2701.3748], [1041919, 2336.6821], [2045193, 2287.1392], [1001440, 1945.2133], [2017087, 1920.5879], [1058542, 1863.7944], [1047972, 1838.2716]]  |
    +--------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
</div> 
</details>
  

<br/>

## Build Model (Text Book)                                                                                 


```python
als_model.userFactors.show(1, truncate=False) # Rank 10
```
<details>
<summary> 실행결과 </summary>
<div markdown="1">  
    +---+-----------------------------------------------------------------------------------------------------------------------------+
    |id |features                                                                                                                     |
    +---+-----------------------------------------------------------------------------------------------------------------------------+
    |90 |[-0.028082313, 7.6311367E-4, 0.2491423, -0.5567669, -0.13148631, -0.3197596, 4.7639868E-4, -0.64081705, 0.6159623, -0.577028]|
    +---+-----------------------------------------------------------------------------------------------------------------------------+
    only showing top 1 row
</div>    
</details>
    


```python
userID = 1004666

existing_artist_ids = (
    new_user_artist_df.where(F.col("userid") == userID)
    .select(F.col("artistid").cast(T.IntegerType()))
    .collect()
)
existing_artist_ids = [row.artistid for row in existing_artist_ids]

artist_df.where(F.col("artistid").isin(existing_artist_ids)).show(n=30, truncate=False)
```
<details>
<summary> 실행결과 </summary>
<div markdown="1">   

    +--------+----------------------------------+
    |artistid|artistname                        |
    +--------+----------------------------------+
    |1302232 |郭富城                            |
    |1020    |The Dave Brubeck Quartet          |
    |1328360 |鄧麗君                            |
    |1034635 |[unknown]                         |
    |1338195 |山崎まさよし                      |
    |6796568 |Les Petits Chanteurs de Saint-Marc|
    |9988765 |伊藤多喜雄                        |
    |1300816 |相川七瀬                          |
    |1003579 |LeAnn Rimes                       |
    |1280437 |倉木麻衣                          |
    |1345189 |米米CLUB                          |
    |1349540 |渡辺美里                          |
    |3066    |Nat King Cole                     |
    |1029324 |TM Network                        |
    |1020059 |Young M.C.                        |
    |1230410 |Billy Paul Williams               |
    |1300525 |氣志團                            |
    |2061677 |渡辺香津美                        |
    |1266817 |Stan Getz & João Gilberto         |
    |1028104 |Intenso Project                   |
    |2003588 |KoЯn                              |
    |1261516 |The Bobby Fuller Four             |
    |1271892 |Zigo                              |
    |2007793 |Luiz Bonfá                        |
    |1330911 |大黒摩季                          |
    |1328587 |木村弓                            |
    |1307741 |少年隊                            |
    |2065806 |柏原芳恵                          |
    |6834961 |大森俊之                          |
    |1235439 |The Murmurs                       |
    +--------+----------------------------------+
    only showing top 30 rows
    
</div>    
</details>


```python
def makeRecomendations(model, userid, howmany):
    to_recommend = (
        model.itemFactors.select(F.col("id").alias("artistid"))
        .withColumn("userid", F.lit(userid))
    )

    return model.transform(to_recommend).select("artistid", "prediction").orderBy(F.col("prediction").desc()).limit(howmany)

recomendation = makeRecomendations(als_model, userID, 3)
recomendation.show()

```

<details>
<summary> 실행결과 </summary>
<div markdown="1">   


    +--------+----------+
    |artistid|prediction|
    +--------+----------+
    |    4423| 488.04785|
    | 1279475| 458.93817|
    | 2134114|  413.4322|
    +--------+----------+
    
    

</div>    
</details>


- ALS 모델의 가중치 확인
- 한 유저에 대한 모든 artist의 음악 play한 횟수 조회.
- 한 유저에 대해 artist prediction 높은 순서대로 orderby. 

<br/>

## 마무리 & 배운점
- 이 단원 정리하며 ALS에 대해 논문찾아보고 관련내용을 [정리하였다](https://swha0105.github.io/ml_dl/2021/01/27/ML_DL-ml-ALS/). 수학적으로 굉장히 간단한 내용이라 비교적 이해하기 편했다.
- column의 join 형태에 대해 찾아보고 정리했다. 아직 이해가 부족한듯 하지만 대충 흐름은 잡히는듯.
- ALS 논문을 찾다 발견했는데 요즘은 이 ALS을 바탕으로 좀 더 advanced된 알고리즘을 사용한다고 한다. 시간될때 찾아보도록 하자.