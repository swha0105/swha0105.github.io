---
layout: post
title:  "[모두연 스터디 Spark] Chapter 3"
subtitle:   "추천 알고리즘"
categories: tech
tags: spark
comments: true
---
# 모두연 스터디
1년 전 쯤에 딥러닝 최신 논문을 읽고 스터디 하는 `슬로우 페이퍼` 코스로 모두의 연구소를 처음 경험했는데 그때 당시 괜찮았다고 생각했었다. 다만 그때 너무 힘든 개인적인 일들과 코스 자체의 난이도가 ~~많이~~ 높아서 다 따라가지는 못했지만 다음에 좋은기회가 있으면 모두의 연구소에서 다시한번 공부해보고 싶다는 생각을 하고있었다.  

그러던 도중 모두연 Slack에 알람이 떠서 봤더니 
`advanced-analytics-with-spark` 의 코스를 개설한다는 것이다.  

쿠팡에 다니는 지인이 spark 정도는 배우놓으라는 언질도 있었고, 현업에서 많이들 요구하는 툴이지만 커리큘럼처럼 체계적으로 배울기회가 부족했다.  

이 스터디를 기회 삼아 배우고자 하였고 스터디 내용은 Spark의 중~고급 내용을 다루는거 같아 경험이 없던 나는 미리 공부를 조금 해갔다. 공부를 좀 해보니 다행히 내가 대학원생때 ~~죽도록~~ 하던 병렬처리의 개념과 매우 흡사한거 같다. 꼭 필요한 내용이고 관련된 개념 이해도 빠른거 같아서 즐겁게 스터디 할 수 있을꺼같다.

관련된 교과서(?)는 `Advanced Analytics with Spark 2판`이고, Spark을 데스크탑에 돌리기 위해 셋팅이 많이 필요한데 다행히 퍼실님께서 Docker로 셋팅 해주셨다. 
그리고 여기 나오는 모든 코드는 퍼실님 [깃허브](https://github.com/dream2globe/advanced-spark) 에서 긁어와 수정한것을 밝힙니다.

열심히 공부해보자고.

<br/>
<br/>

***

# Chapter 3
Chapter 3 제목은 `음악 추천과 Audioscrobbler 데이터셋`이다.  
음악 감상 데이터셋을 받아 ALS 알고리즘을 사용하여 특정 유저에 대한 추천 아티스트들을 보여주는 실습이다. ~~그냥 유튜브 생각하면 쉽다~~

## Data set
Auhdioscrobbler에서 만든 데이터 셋이고 https://www.last.fm/ 에 추천 시스템을 구축하는데 사용 되었다. 총 3개의 데이터로 구성되어있고 모두 txt 파일이다.    

1. **user_artist_data**: userid  artistid  playcount 
    - user의 artist에 대한 재생 정보를 나타냄.  
    - empty space로 칼럼 구분
    - integer type으로 구성되어 있음.

2. **artist_data.txt**: artistid  artist_name

3. **artist_alias.txt**: badid  goodid
   - Artist들의 이름이 통일 되어있지 않음으로 여러 이름들의 형태를 저장

## ALS (Alternating Least Squares)
자세한 내용은 여기로..




<!-- ALS 에 대한 자세한 설명은 여기 따로 정리를 해 -->

# Chapter 3 Code


## Referrence
- ALS 알고리즘 설명 : https://ggoals.github.io/Spark_ALS_Algorithm_tuning/
- Towards data science : https://towardsdatascience.com/build-recommendation-system-with-pyspark-using-alternating-least-squares-als-matrix-factorisation-ebe1ad2e7679
- Spark tutorial : https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
- 코드 원본 : https://github.com/dream2globe/advanced-spark

## Download data


```python
!wget https://storage.googleapis.com/aas-data-sets/profiledata_06-May-2005.tar.gz
!tar xvf ./data/profiledata_06-May-2005.tar.gz


# 실행결과
profiledata_06-May-2005/
profiledata_06-May-2005/artist_data.txt
profiledata_06-May-2005/README.txt
profiledata_06-May-2005/user_artist_data.txt
profiledata_06-May-2005/artist_alias.txt

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
```


```python
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

Spark 기본 설정을 해주는 코드.  

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


Executor_instance: = --num-executor와 같음  executor의 갯수 설정  
Executor_cores: 각 executor가 사용하는 thread의 갯수

---

<br/>


## Load dataset and Preprocessing

---

### Data summary   

#### user_artist_data.txt   
- User가 어떤 artist들을 몇번 play했는지 기록한 data
- 3 columns: userid artistid playcount


#### artist_data.txt
- Artist와 번호를 대응시켜 저장

#### artist_alias.txt
- 2 columns: badid, goodid
- 한 artist가 여러이름으로 저장되어있음, 이름에 대한 테크

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


### Data를 읽고 Schema에 load하는 부분

#### user_artist_data.txt   
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
```
#### artist_data

```python
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
```

### artist_alias

```python
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

- option("header"): data의 첫 row가 header경우 True로 반환한다.  
- option("sep"): data끼리의 간격을 지정한다. 이 경우 Space  

---

### 읽은 데이터들을 병합하는 부분

```python
new_user_artist_df = (
    user_artist_df
    .join(F.broadcast(artist_alias_df), user_artist_df.artistid == artist_alias_df.badid, "left")
    .withColumn("artistid", F.when(F.col("badid").isNull(),  F.col("artistid")).otherwise(F.col("goodid")))
    .where(F.col("badid").isNotNull())
    .cache()
)
```

---

- user_artist_df 데이터에 artist_alias_df를 붙이는데, user_artist_df의 artistid가 alias에 등록되어있는 badid와 같을때, left join을 한다.
- user_artist_df의 artistid 필드를 artist_df를 참조하여 badid를 goodid로 교체
- broadcase 함수 적용
- cache 함수를 적용하면 Storage 탭에서 메모리 사용량을 알 수 있음

---

```python
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
```

--- 
ALS 모델 (링크)  


### Implicit vs Explicit 

Explicit feedback은 데이터가 가지고 있는 값이 우리가 사용해야할 데이터와 일치할때 사용된다  
예를 들어, 이 예제와같이 음악추천 시스템을 만들때 유저가 듣는 음악에 대해 평점을 매긴다면 그 평점에 대한 데이터를 사용하면 Explicit feedback이 된다.  

예제에서 사용하고 있는 데이터는 단순히 조회수로만 카운팅하며 이럴 경우 많은 bias나 오차가 생길수 있다. (틀어놓고 딴짓한다던가.. 기타등등) 이런 데이터를 이용하는것을 Implicit feedback이라고 한다. Implicit은 Explicit에 비해 사용하기 힘들지만 큰 데이터를 모을수 있는 장점이 있다.


---

```python
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



### Build Model (towards data science)


```python
# Import the required functions
from pyspark.ml.evaluation import RegressionEvaluator
from pyspark.ml.recommendation import ALS
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from pyspark.ml.evaluation import RegressionEvaluator

# Create ALS model
als = ALS(
         userCol="userid", 
         itemCol="artistid",
         ratingCol="playcount", 
         nonnegative = True, 
         implicitPrefs = False,
         coldStartStrategy="drop"
)

# Add hyperparameters and their respective values to param_grid
param_grid = (
    ParamGridBuilder()
    .addGrid(als.rank, [5, 30])
    .addGrid(als.regParam, [4.0, 0.0001])
    .addGrid(als.alpha, [1.0, 40.0])    
    .build()
)

# Define evaluator as RMSE and print length of evaluator
evaluator = RegressionEvaluator(
    metricName="rmse", 
    labelCol="playcount", 
    predictionCol="prediction") 

# Build cross validation using CrossValidator
cv = CrossValidator(estimator=als, estimatorParamMaps=param_grid, evaluator=evaluator, numFolds=5)




# Fit cross validator to the 'train' dataset
model = cv.fit(train)

# Extract best model from the cv model above
best_model = model.bestModel

# View the predictions
test_predictions = best_model.transform(test)
RMSE = evaluator.evaluate(test_predictions)
print(RMSE)

print("**Best Model**")
print("  Rank:", best_model._java_obj.parent().getRank())
print("  MaxIter:", best_model._java_obj.parent().getMaxIter())
print("  RegParam:", best_model._java_obj.parent().getRegParam())




# Generate n Recommendations for all users
recommendations = best_model.recommendForAllUsers(5)
recommendations.show(10, False)




nrecommendations = recommendations\
    .withColumn("rec_exp", F.explode("recommendations"))\
    .select("userid", "rec_exp.artistid", "rec_exp.rating")
    
nrecommendations.limit(10).show()
```
<details>
<summary> 실행결과 </summary>
<div markdown="1">  

    +-------+--------+---------+
    | userid|artistid|   rating|
    +-------+--------+---------+
    |1000144| 2043183|61.646343|
    |1000144| 1273059|59.896374|
    |1000144| 1027760| 38.36873|
    |1000144| 9985060| 37.61595|
    |1000144| 2091861| 36.61011|
    |1000465| 6672069|21.925694|
    |1000465| 1032434|21.773518|
    |1000465| 6812406|21.544062|
    |1000465| 2091861|  21.4468|
    |1000465| 1337692|20.663816|
    +-------+--------+---------+
    
 </div>    
</details>

   

## Prediction vs Real Data 


```python
nrecommendations.join(artist_df, on="artistid").filter('userid = 1000190').sort(F.col("rating").desc()).show()
```
<details>
<summary> 실행결과 </summary>
<div markdown="1">  
    +--------+-------+---------+-----------------+
    |artistid| userid|   rating|       artistname|
    +--------+-------+---------+-----------------+
    | 6672069|1000190|32.698563|           hiro:n|
    |    2513|1000190|30.011713|          Merzbow|
    | 2091861|1000190|  29.3715|Purified in Blood|
    | 2043183|1000190|  29.2197|       中川幸太郎|
    | 1167516|1000190|27.088203|       Putsch '79|
    +--------+-------+---------+-----------------+
    
    
</div>    
</details>



```python
(
    new_user_artist_df.select("userid", "artistid", "playcount")
    .join(artist_df, on="artistid")
    .filter('userid = 1000190')
    .sort(F.col("playcount").desc())
).show()
```
<details>
<summary> 실행결과 </summary>
<div markdown="1">  
    +--------+-------+---------+--------------------+
    |artistid| userid|playcount|          artistname|
    +--------+-------+---------+--------------------+
    |     754|1000190|       20|           Sigur Rós|
    | 6715171|1000190|        9|        The '89 Cubs|
    | 1283231|1000190|        6|The Les Claypool ...|
    | 1290488|1000190|        4|The Nation of Uly...|
    | 1146220|1000190|        1|   Animal Collective|
    | 1013111|1000190|        1|  Murder City Devils|
    | 1004758|1000190|        1|         Silver Jews|
    +--------+-------+---------+--------------------+
    
    
</div>    
</details>

