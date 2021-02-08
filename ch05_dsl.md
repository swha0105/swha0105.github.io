## Ch5: Anomaly Detection in Network Traffic with K-mean Clustering

### Spark Session


```python
import gc
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt

from pyspark.sql import SparkSession
from pyspark.sql.window import Window
import pyspark.sql.functions as F
import pyspark.sql.types as T
from pyspark.sql import Row

from pyspark.ml import Pipeline
from pyspark.ml.feature import VectorAssembler, StringIndexer, OneHotEncoder, StandardScaler, PCA
from pyspark.ml.clustering import KMeans, KMeansModel
from pyspark.ml.evaluation import ClusteringEvaluator

import matplotlib.pyplot as plt
import seaborn as sns

```


```python
EXECUTOR_MEMORY = "2g"
EXECUTOR_CORES = 2
EXECUTORE_INSTANCES = 3
DRIVER_MEMORY = "1g"
DRIVER_MAX_RESULT_SIZE = "1g"
```


```python
spark = (
    SparkSession.builder.appName(f"CH5")
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

spark.sparkContext.getConf().getAll()
```




    [('spark.executor.memory', '2g'),
     ('spark.executor.instances', '3'),
     ('spark.driver.extraJavaOptions',
      '"-Dio.netty.tryReflectionSetAccessible=true"'),
     ('spark.kryoserializer.buffer.max', '1024m'),
     ('spark.app.id', 'local-1612059470767'),
     ('spark.executor.id', 'driver'),
     ('spark.app.name', 'CH5'),
     ('spark.driver.maxResultSize', '1g'),
     ('spark.driver.memory', '1g'),
     ('spark.executor.cores', '2'),
     ('spark.driver.host', 'ce648118105d'),
     ('spark.executor.extraJavaOptions',
      '"-Dio.netty.tryReflectionSetAccessible=true"'),
     ('spark.sql.catalogImplementation', 'hive'),
     ('spark.rdd.compress', 'True'),
     ('spark.serializer.objectStreamReset', '100'),
     ('spark.master', 'local[*]'),
     ('spark.submit.pyFiles', ''),
     ('spark.submit.deployMode', 'client'),
     ('spark.driver.port', '42146'),
     ('spark.ui.showConsoleProgress', 'true')]



## Load Data


```python
schema = [
    T.StructField("duration", T.IntegerType(), True),
    T.StructField("protocol_type", T.StringType(), True),
    T.StructField("service", T.StringType(), True),
    T.StructField("flag", T.StringType(), True), 
    T.StructField("src_bytes", T.IntegerType(), True), 
    T.StructField("dst_bytes", T.IntegerType(), True), 
    T.StructField("land", T.IntegerType(), True), 
    T.StructField("wrong_fragment", T.IntegerType(), True),
    T.StructField("urgent", T.IntegerType(), True),
    T.StructField("hot", T.IntegerType(), True),
    T.StructField("num_failed_logins", T.IntegerType(), True),
    T.StructField("logged_in", T.IntegerType(), True),
    T.StructField("num_compromised", T.IntegerType(), True),
    T.StructField("root_shell", T.IntegerType(), True),
    T.StructField("su_attempted", T.IntegerType(), True),
    T.StructField("num_root", T.IntegerType(), True),
    T.StructField("num_file_creations", T.IntegerType(), True),
    T.StructField("num_shells", T.IntegerType(), True),
    T.StructField("num_access_files", T.IntegerType(), True),
    T.StructField("num_outbound_cmds", T.IntegerType(), True),
    T.StructField("is_host_login", T.DoubleType(), True),
    T.StructField("is_guest_login", T.DoubleType(), True),
    T.StructField("count", T.DoubleType(), True),
    T.StructField("srv_count", T.DoubleType(), True),
    T.StructField("serror_rate", T.DoubleType(), True),
    T.StructField("srv_serror_rate", T.DoubleType(), True),
    T.StructField("rerror_rate", T.DoubleType(), True),
    T.StructField("srv_rerror_rate", T.DoubleType(), True),
    T.StructField("same_srv_rate", T.DoubleType(), True),
    T.StructField("diff_srv_rate", T.DoubleType(), True),
    T.StructField("srv_diff_host_rate", T.DoubleType(), True),
    T.StructField("dst_host_count", T.DoubleType(), True),
    T.StructField("dst_host_srv_count", T.DoubleType(), True),
    T.StructField("dst_host_same_srv_rate", T.DoubleType(), True),
    T.StructField("dst_host_diff_srv_rate", T.DoubleType(), True),
    T.StructField("dst_host_same_src_port_rate", T.DoubleType(), True),
    T.StructField("dst_host_srv_diff_host_rate", T.DoubleType(), True),
    T.StructField("dst_host_serror_rate", T.DoubleType(), True),
    T.StructField("dst_host_srv_serror_rate", T.DoubleType(), True),
    T.StructField("dst_host_rerror_rate", T.DoubleType(), True),
    T.StructField("dst_host_srv_rerror_rate", T.DoubleType(), True),
    T.StructField("label", T.StringType(), True)
    ]

schema = T.StructType(schema)

df = (spark
      .read.format("csv")
      .option("header", False)
      .option("sep", ",")
      .schema(schema)
#       .option("inferSchema", True)
      .load("./data/kddcup.data.corrected"))

# df.printSchema()
df.show(2)
# df.select("label").show(5) # 모든 컬럼의 스키마가 반영되었는지 확인
```

    +--------+-------------+-------+----+---------+---------+----+--------------+------+---+-----------------+---------+---------------+----------+------------+--------+------------------+----------+----------------+-----------------+-------------+--------------+-----+---------+-----------+---------------+-----------+---------------+-------------+-------------+------------------+--------------+------------------+----------------------+----------------------+---------------------------+---------------------------+--------------------+------------------------+--------------------+------------------------+-------+
    |duration|protocol_type|service|flag|src_bytes|dst_bytes|land|wrong_fragment|urgent|hot|num_failed_logins|logged_in|num_compromised|root_shell|su_attempted|num_root|num_file_creations|num_shells|num_access_files|num_outbound_cmds|is_host_login|is_guest_login|count|srv_count|serror_rate|srv_serror_rate|rerror_rate|srv_rerror_rate|same_srv_rate|diff_srv_rate|srv_diff_host_rate|dst_host_count|dst_host_srv_count|dst_host_same_srv_rate|dst_host_diff_srv_rate|dst_host_same_src_port_rate|dst_host_srv_diff_host_rate|dst_host_serror_rate|dst_host_srv_serror_rate|dst_host_rerror_rate|dst_host_srv_rerror_rate|  label|
    +--------+-------------+-------+----+---------+---------+----+--------------+------+---+-----------------+---------+---------------+----------+------------+--------+------------------+----------+----------------+-----------------+-------------+--------------+-----+---------+-----------+---------------+-----------+---------------+-------------+-------------+------------------+--------------+------------------+----------------------+----------------------+---------------------------+---------------------------+--------------------+------------------------+--------------------+------------------------+-------+
    |       0|          tcp|   http|  SF|      215|    45076|   0|             0|     0|  0|                0|        1|              0|         0|           0|       0|                 0|         0|               0|                0|          0.0|           0.0|  1.0|      1.0|        0.0|            0.0|        0.0|            0.0|          1.0|          0.0|               0.0|           0.0|               0.0|                   0.0|                   0.0|                        0.0|                        0.0|                 0.0|                     0.0|                 0.0|                     0.0|normal.|
    |       0|          tcp|   http|  SF|      162|     4528|   0|             0|     0|  0|                0|        1|              0|         0|           0|       0|                 0|         0|               0|                0|          0.0|           0.0|  2.0|      2.0|        0.0|            0.0|        0.0|            0.0|          1.0|          0.0|               0.0|           1.0|               1.0|                   1.0|                   0.0|                        1.0|                        0.0|                 0.0|                     0.0|                 0.0|                     0.0|normal.|
    +--------+-------------+-------+----+---------+---------+----+--------------+------+---+-----------------+---------+---------------+----------+------------+--------+------------------+----------+----------------+-----------------+-------------+--------------+-----+---------+-----------+---------------+-----------+---------------+-------------+-------------+------------------+--------------+------------------+----------------------+----------------------+---------------------------+---------------------------+--------------------+------------------------+--------------------+------------------------+-------+
    only showing top 2 rows
    



```python
# ===== Check Label Distribution =====
label_df = df.groupBy('label').count().orderBy('count', ascending=False)
label_df = label_df.withColumn('percent', F.col('count')/F.sum('count').over(Window.partitionBy()))
label_df.show(30)   

# [NOTE]
#- 24 labels
#- most of them being smurf and neptune attacks

# [수정]
#- ratio to % and round
```

    +----------------+-------+--------------------+
    |           label|  count|             percent|
    +----------------+-------+--------------------+
    |          smurf.|2807886|  0.5732215070499105|
    |        neptune.|1072017|  0.2188490559528143|
    |         normal.| 972781| 0.19859032412623553|
    |          satan.|  15892|0.003244304145551...|
    |        ipsweep.|  12481| 0.00254795872392609|
    |      portsweep.|  10413|0.002125782725121...|
    |           nmap.|   2316|  4.7280445514084E-4|
    |           back.|   2203|4.497358439875952E-4|
    |    warezclient.|   1020|2.082299413832715E-4|
    |       teardrop.|    979|1.998599143276694...|
    |            pod.|    264|5.389480835802321...|
    |   guess_passwd.|     53|1.081979107187587...|
    |buffer_overflow.|     30|6.124410040684456E-6|
    |           land.|     21|4.287087028479119E-6|
    |    warezmaster.|     20|4.082940027122970...|
    |           imap.|     12|2.449764016273782...|
    |        rootkit.|     10|2.041470013561485...|
    |     loadmodule.|      9|1.837323012205336...|
    |      ftp_write.|      8|1.633176010849188...|
    |       multihop.|      7|1.429029009493039...|
    |            phf.|      4|8.165880054245942E-7|
    |           perl.|      3|6.124410040684456E-7|
    |            spy.|      2|4.082940027122971E-7|
    +----------------+-------+--------------------+
    



```python
def feature_pipeline(data, used_cols):
    
    assembler = VectorAssembler(inputCols = used_cols, outputCol='features')
    ppl = Pipeline(stages = [assembler])
                
    featureDf = ppl.fit(data).transform(data)
    return featureDf
```


```python
def encoding_pipeline(data, str_cols):

    stage_string = [StringIndexer(inputCol= c, outputCol= c+"_str_encoded") for c in str_cols]
    stage_onehot = [OneHotEncoder(inputCol= c+"_str_encoded", outputCol= c+ "_onehot_vec") for c in str_cols]

    ppl = Pipeline(stages = stage_string + stage_onehot)
    encodedDF = ppl.fit(data).transform(data)
    return encodedDF

# [NOTE]
#-  use a StringIndexer to turn the categorical values into category indices which are then converted into a column of binary vectors by the OneHotEncoder for each of the columns as
```


```python
def Kmeans_model(data, k):
    #-- Train k-mean model
    kmeans = KMeans(featuresCol='features', predictionCol='prediction', k=k).setSeed(1)
    model = kmeans.fit(data)

    #-- Make Predictions
    predDf = model.transform(data)

    #-- Evaluate clustering by computing Silhouette score
    evaluator = ClusteringEvaluator()
    silhouette = evaluator.evaluate(predDf)
    print(f"Silhouette with squared euclidean distance with k={k}: {str(silhouette)}")

    #-- Shows the result
    centers = model.clusterCenters()
#     print("\nCluster Centers: ")
#     for center in centers:
#         print(center)
    return predDf
```

## Basic K-means Clustering


```python
# ===== Data preprocessing =====
num_cols = df.drop('protocol_type', 'service', 'flag', 'label').columns
featureDf = feature_pipeline(df, num_cols)

print(f"Number of columns in featureDf: {len(df.columns)}")
featureDf.select("label", "features").show(5, False)
```

    Number of columns in featureDf: 42
    +-------+---------------------------------------------------------------------------------+
    |label  |features                                                                         |
    +-------+---------------------------------------------------------------------------------+
    |normal.|(38,[1,2,8,19,20,25],[215.0,45076.0,1.0,1.0,1.0,1.0])                            |
    |normal.|(38,[1,2,8,19,20,25,28,29,30,32],[162.0,4528.0,1.0,2.0,2.0,1.0,1.0,1.0,1.0,1.0]) |
    |normal.|(38,[1,2,8,19,20,25,28,29,30,32],[236.0,1228.0,1.0,1.0,1.0,1.0,2.0,2.0,1.0,0.5]) |
    |normal.|(38,[1,2,8,19,20,25,28,29,30,32],[233.0,2032.0,1.0,2.0,2.0,1.0,3.0,3.0,1.0,0.33])|
    |normal.|(38,[1,2,8,19,20,25,28,29,30,32],[239.0,486.0,1.0,3.0,3.0,1.0,4.0,4.0,1.0,0.25]) |
    +-------+---------------------------------------------------------------------------------+
    only showing top 5 rows
    



```python
# ===== KMeans Model =====
predDf = Kmeans_model(featureDf, k=2)

# -- Check Cluster Distribution
cluster_df = predDf.select("prediction", "label").groupBy('prediction', 'label').count().orderBy('count', ascending=False)
cluster_df.show(30)   

# [NOTE]
#- only one data point ended up in cluster 1
```

    Silhouette with squared euclidean distance with k=2: 0.999998858708827
    +----------+----------------+-------+
    |prediction|           label|  count|
    +----------+----------------+-------+
    |         0|          smurf.|2807886|
    |         0|        neptune.|1072017|
    |         0|         normal.| 972781|
    |         0|          satan.|  15892|
    |         0|        ipsweep.|  12481|
    |         0|      portsweep.|  10412|
    |         0|           nmap.|   2316|
    |         0|           back.|   2203|
    |         0|    warezclient.|   1020|
    |         0|       teardrop.|    979|
    |         0|            pod.|    264|
    |         0|   guess_passwd.|     53|
    |         0|buffer_overflow.|     30|
    |         0|           land.|     21|
    |         0|    warezmaster.|     20|
    |         0|           imap.|     12|
    |         0|        rootkit.|     10|
    |         0|     loadmodule.|      9|
    |         0|      ftp_write.|      8|
    |         0|       multihop.|      7|
    |         0|            phf.|      4|
    |         0|           perl.|      3|
    |         0|            spy.|      2|
    |         1|      portsweep.|      1|
    +----------+----------------+-------+
    



```python
# ===== PCA ====
pca = PCA(k=2, inputCol="features", outputCol="features_pca")
pca_model = pca.fit(predDf)

pcaDf = pca_model.transform(predDf).select("features_pca")
pcaDf.show(5, False)
```

    +----------------------------------------+
    |features_pca                            |
    +----------------------------------------+
    |[-228.931024196869,45075.93138650123]   |
    |[-163.39939875271935,4527.949711690792] |
    |[-236.3795087236227,1227.927001218682]  |
    |[-233.62798934133454,2031.9278878244147]|
    |[-239.15018651748508,485.9261057609585] |
    +----------------------------------------+
    only showing top 5 rows
    



```python
pca_df = pcaDf.toPandas()
dataX = []
dataY = []
for vec in pca_df.values:
    dataX.extend([vec[0][0]])
    dataY.extend([vec[0][1]])
sns.scatterplot(dataX, dataY, hue=dataY, )
plt.show()
```

    /opt/conda/lib/python3.8/site-packages/seaborn/_decorators.py:36: FutureWarning: Pass the following variables as keyword args: x, y. From version 0.12, the only valid positional argument will be `data`, and passing other arguments without an explicit keyword will result in an error or misinterpretation.
      warnings.warn(



```python
# ===== PCA ====
pca = PCA(k=3, inputCol="features", outputCol="features_pca")
pca_model = pca.fit(predDf)

pcaDf = pca_model.transform(predDf).select("features_pca")
pcaDf.show(5, False)
```


```python
pca_df = pcaDf.toPandas()
dataX = []
dataY = []
dataZ = []
for vec in pca_df.values:
    dataX.extend([vec[0][0]])
    dataY.extend([vec[0][1]])
    dataZ.extend([vec[0][2]])

fig = plt.figure(figsize=(10, 5))
ax = fig.add_subplot(111, projection='3d') # Axe3D object
ax.scatter(dataX, dataY, dataZ, c=dataZ, s=20, alpha=0.5)
```

## Pyspark K-means Clustering with appropriate K


```python
# =====  scores =====
cost = list()
for k in [20, 40, 60, 80, 100]:
    kmeans = KMeans(featuresCol='features', predictionCol='prediction', k=k).setSeed(1)
    model = kmeans.fit(featureDf)
    predDf = model.transform(featureDf)
    
    evaluator = ClusteringEvaluator()
    evaluator.setPredictionCol('prediction')
    cost.append(evaluator.evaluate(predDf))
    print(k, evaluator.evaluate(predDf))
```


```python
fig, ax = plt.subplots()
ax.plot(range(20,101, 20), cost, 'o-')
ax.set_xlabel('Number of clusters')
ax.set_ylabel('Score')
ax.set_title('Clustering Scores for k-means clustering')
```


```python
# ===== Silhouette scores =====
cost = list()
evaluator = ClusteringEvaluator()
for k in range(2,101):
    kmeans = KMeans().setK(k).setSeed(1).setFeaturesCol("features")
    model = kmeans.fit(featureDf)
    predictions = model.transform(featureDf)
    silhouette = evaluator.evaluate(predictions)
    cost.append(silhouette)
    
kIdx = np.argmax(cost)
```


```python
fig, ax = plt.subplots()
ax.plot(range(2,101), cost, 'o-')
ax.plot(range(2,101)[kIdx], cost[kIdx]
        , marker='o', markersize=14
        , markeredgewidth=2, markeredgecolor='r', markerfacecolor='None')
ax.set_xlim(0, 100)
ax.set_xlabel('Number of clusters')
ax.set_ylabel('Silhouette Coefficient')
ax.set_title('Silhouette Scores for k-means clustering')
```


```python
# ===== KMeans Model =====
predDf = Kmeans_model(featureDf, k=20)

# -- Check Cluster Distribution
cluster_df = predDf.select("prediction", "label").groupBy('prediction', 'label').count().orderBy('count', ascending=False)
cluster_df.show(30)   

# [NOTE]
#- only one data point ended up in cluster 1
```

### Scaling and Encoding
#- scalers are applied on Vector Data Types that is why we need to collect the features using a VectorAssembler first:


```python
# ===== Data preprocessing =====
str_cols = ["protocol_type", "service", "flag", "label"]
encodedDF = encoding_pipeline(df, str_cols)

used_cols = df.drop(*str_cols).columns + [c+ "_onehot_vec" for c in str_cols]
featureDf_enc = feature_pipeline(encodedDF, used_cols)

print(f"Number of columns used: {len(used_cols)}")
featureDf_enc.select("label", "features").show(5, False)

# [NOTE]
#-  use a StringIndexer to turn the categorical values into category indices which are then converted into a column of binary vectors by the OneHotEncoder for each of the columns as
```


```python
scaler = StandardScaler(inputCol="features", outputCol="scaled_features")
scaler_model = scaler.fit(featureDf_enc)
featureDf_scaled = scaler_model.transform(featureDf_enc)

# A StandardScaler standardizes features by removing the mean and scaling to unit standard deviation using column-summary-statistics.
# StandardScaler can take two additional parameters:
# withStd: True by default. Scales the data to unit standard deviation.
# withMean: False by default. Centers the data with mean before scaling.
```


```python
# ===== KMeans Model =====
predDf = Kmeans_model(featureDf_scaled, k=20)

# -- Check Cluster Distribution
cluster_df = predDf.select("prediction", "label").groupBy('prediction', 'label').count().orderBy('count', ascending=False)
cluster_df.show(30)   

# [NOTE]
#- only one data point ended up in cluster 1
```

## Dimension Reduction: PCA
- procedure that converts a set of observations from m to n dimensions (m > n), after analyzing the correlated features of the variables. It is used to move the data from high to a low dimension for visualization or dimensionality reduction purposes.
- our data has 42 features but we will reduce to 2 dimension


```python
# ===== PCA ====
pca = PCA(k=20, inputCol="features", outputCol="features_pca")
pca_model = pca.fit(predDf)

pcaDf = pca_model.transform(predDf).select("features_pca")
pcaDf.show(5, False)
```


```python
pca_df = pcaDf.toPandas()
dataX = []
dataY = []
for vec in pca_df.values:
    dataX.extend([vec[0][0]])
    dataY.extend([vec[0][1]])
sns.scatterplot(dataX, dataY, hue=dataY, )
plt.show()
```


```python
# ===== PCA ====
pca = PCA(k=3, inputCol="features", outputCol="features_pca")
pca_model = pca.fit(predDf)

pcaDf = pca_model.transform(predDf).select("features_pca")
pcaDf.show(5, False)
```


```python
pca_df = pcaDf.toPandas()
dataX = []
dataY = []
dataZ = []
for vec in pca_df.values:
    dataX.extend([vec[0][0]])
    dataY.extend([vec[0][1]])
    dataZ.extend([vec[0][2]])

fig = plt.figure(figsize=(10, 5))
ax = fig.add_subplot(111, projection='3d') # Axe3D object
ax.scatter(dataX, dataY, dataZ, c=dataZ, s=20, alpha=0.5)
```


```python

```
