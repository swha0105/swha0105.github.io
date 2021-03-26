---
layout: post
title:  "[모두연 풀잎스쿨 14기] Chapter 6"
subtitle:   "자연어처리 with LSA (Latent Semantic Analysis)"
categories: tech
tags: spark
comments: true
---

**본 포스팅은 모두의연구소(home.modulabs.co.kr) 풀잎스쿨에서 진행된 `Advanced Analytics with Spark` 과정 내용을 공유 및 정리한 자료입니다.**    

--- 

### Chapter 6 제목은 `숨은 의미 분석으로 위키백과 이해하기`이다.   

위키백과 페이지들을 크롤링 하여 특정 페이지에서 어떤 단어들이 중요한지 알아내는 기법이다.   
책에서는 모든 위키백과 페이지를 크롤링하는거 같지만 굳이 그럴필요 없을꺼같아서 https://en.wikipedia.org/wiki/Special:Export 에서 `Machine learning`만 검색하여 나온 페이지들만 추출하였다.  

자연어 처리는 아직 미숙하여 같이 스터디하는 분의 [코드](https://github.com/pko89403/Spark-Test/blob/master/AdvancedAnalyticswithSpark/ch6/LSA.ipynb) 를 많이 참조했다 

---

# Spark 시작

## data load 및 dataframe 생성

```python
from pyspark.sql import SparkSession
import pyspark.sql.functions as F
import os

jar_path = '/usr/spark/jars/spark-xml_2.12-0.11.0.jar'
spark = SparkSession.builder.appName('Chapter06')\
    .master('local[4]')\
    .config("spark.executor.memory", "2g")\
    .config("spark.jars", jar_path)\
    .getOrCreate()

df = spark.read.format("xml") \
                .option("rowTag", "page") \
                .load("wiki_ml.xml")

df.printSchema()
```
<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    root
     |-- id: long (nullable = true)
     |-- ns: long (nullable = true)
     |-- redirect: struct (nullable = true)
     |    |-- _VALUE: string (nullable = true)
     |    |-- _title: string (nullable = true)
     |-- revision: struct (nullable = true)
     |    |-- comment: string (nullable = true)
     |    |-- contributor: struct (nullable = true)
     |    |    |-- id: long (nullable = true)
     |    |    |-- ip: string (nullable = true)
     |    |    |-- username: string (nullable = true)
     |    |-- format: string (nullable = true)
     |    |-- id: long (nullable = true)
     |    |-- minor: string (nullable = true)
     |    |-- model: string (nullable = true)
     |    |-- parentid: long (nullable = true)
     |    |-- sha1: string (nullable = true)
     |    |-- text: struct (nullable = true)
     |    |    |-- _VALUE: string (nullable = true)
     |    |    |-- _bytes: long (nullable = true)
     |    |    |-- _space: string (nullable = true)
     |    |-- timestamp: timestamp (nullable = true)
     |-- title: string (nullable = true)

</div>
</details> 

## data에서 필요한 title, text만 추출 

```python
# html data 형태: title, revision의 text안의 _value가 본문, _space, _byte는 metadata
parsedDF = df.select("title", "revision.text._VALUE")
parsedDF = parsedDF.withColumn("raw_text",F.col("_VALUE")).drop("_VALUE")
parsedDF.show(5)
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    +----------------+--------------------+
    |           title|                text|
    +----------------+--------------------+
    | Bongard problem|  file bongard pr...|
    |Generative model|  about generativ...|
    +----------------+--------------------+
    only showing top 2 rows
</div>
</details> 

- F.regexp_replace을 이용, 'raw_text' 컬럼의 값들에 대해 정규식을 적용하여 특수 문자들을 제외한다

<br/>
<br/>

---

# 전처리 

## Tokenizer & Stopwords remove
```python
import pyspark.ml.feature as ml

# Tokenizer는 python의 string split 역할.

tokenizer = ml.Tokenizer(inputCol='text', outputCol='words_token')
df_words_token = tokenizer.transform(df_clean).select("title","words_token")

# stop word는 자연어 처리에서 나온 개념. 의미없는 단어 삭제. (I, my.. )
remover = ml.StopWordsRemover(inputCol='words_token', outputCol='words_clean')
df_wo_stopWords = remover.transform(df_words_token).select('title','words_clean')
df_wo_stopWords.show(2)
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    +----------------+--------------------+
    |           title|         words_clean|
    +----------------+--------------------+
    | Bongard problem|[, , file, bongar...|
    |Generative model|[, , generative, ...|
    +----------------+--------------------+
    only showing top 2 rows
    
</div>
</details> 

- Tokenizer는 Python의 string split의 역할. 띄어쓰기 되어있는 단어들을 분리한다.
- stopword는 자연어 처리에서 나오는 개념. 의미없는 단어 (I, My, This) 같은거 삭제.

<br/>

## Empty space removed
```python
# python udf은 row-a-time으로 느리다. 
# pandas udf: The input and output series must have the same size.
from pyspark.sql.functions import pandas_udf,PandasUDFType
from pyspark.sql.types import ArrayType,StringType
import pyspark.sql.column as c
import pandas as pd


#pandas_udf 안에는 return data type, https://spark.apache.org/docs/latest/sql-ref-datatypes.html


@pandas_udf(ArrayType(StringType()))
def remove_empty_word(input_array: pd.Series) -> pd.Series:
    import os
    os.environ['ARROW_PRE_0_15_IPC_FORMAT']='1'
    return_list = []
    
    for word_list in input_array:
        tmp_list = []
        for word in word_list:
            if word != '':
                tmp_list.append(word)
        return_list.append(tmp_list)
    return pd.Series(return_list)


removed = remove_empty_word(df_wo_stopWords.words_clean)
df_words_wo_empty = df_wo_stopWords.withColumn('refined_text',removed).select('title','refined_text')  
df_words_wo_empty.show(2)
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    +----------------+--------------------+
    |           title|        refined_text|
    +----------------+--------------------+
    | Bongard problem|[file, bongard, p...|
    |Generative model|[generative, mode...|
    +----------------+--------------------+
    only showing top 2 rows

</div>
</details> 


- pyarrow의 버전문제(?)로 한 3시간을 고생했다. 
- 이유는 모르겠으나  os.environ['ARROW_PRE_0_15_IPC_FORMAT']='1' 설정해주니 돌아간다. 대충 찾아보니 pyarrow와 spark의 버전 호환이 안맞다고 하는데.. 나중에 자세히 찾아봐야겟다 [링크1](https://stackoverflow.com/questions/58458415/pandas-scalar-udf-failing-illegalargumentexception), [링크2](https://issues.apache.org/jira/browse/SPARK-29367)

- Pandas_udf에 함수 decorate에 들어가는 데이터 타입은 output의 형태.
- Pandas_udf에 argument type hint 명시적 작성 필요.


<br/>


## Word Stemmed 

```python
from nltk.stem.snowball import SnowballStemmer


@pandas_udf(ArrayType(StringType()))
def word_stem(input_array: pd.Series) -> pd.Series:
    import os

    stemmer = SnowballStemmer(language='english')
    os.environ['ARROW_PRE_0_15_IPC_FORMAT']='1'
    return_list = []
    for array in input_array:
        tmp_list = []
        for word in array:
            tmp_list.append(stemmer.stem(word))

        return_list.append(tmp_list)
    
    return pd.Series(return_list)

stemmed = word_stem(df_words_wo_empty.refined_text)

# stemmer_udf = udf(lambda tokens: [stemmer.stem(token) for token in tokens], ArrayType(StringType()))
# stemmed_words = words_clean_remove_empty.withColumn("SnowballStemmed", stemmer_udf("text"))

stemmed_words = df_words_wo_empty.withColumn("SnowballStemmed", stemmed).drop('refined_text')
stemmed_words.show(2)

```
<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    +----------------+--------------------+
    |           title|     SnowballStemmed|
    +----------------+--------------------+
    | Bongard problem|[file, bongard, p...|
    |Generative model|[generat, model, ...|
    +----------------+--------------------+
    only showing top 2 rows

</div>
</details> 

- stemmed, 즉 여러 단어들의 파생형태를 어근으로 모은다. 
- nltk의 stem 모듈을 사용하여 영어 단어 처리

<br/>
<br/>

---

# 모델구성 

## TF-IDF 구성
```python
from pyspark.ml.feature import CountVectorizer
from pyspark.ml.feature import HashingTF, IDF, Tokenizer
from pyspark.sql.functions import monotonically_increasing_id

# hashingTF = HashingTF(inputCol="SnowballStemmed", outputCol="TF", numFeatures=20)
# featurizedData = hashingTF.transform(stemmed_words)

countVectorizer = CountVectorizer(inputCol="SnowballStemmed",
                                      outputCol="termFreqs",
                                      vocabSize=2000)

vocabModel = countVectorizer.fit(stemmed_words)
docTermFreqs = vocabModel.transform(stemmed_words)
docTermFreqs.cache()
docTermFreqswithID = docTermFreqs.withColumn('id', monotonically_increasing_id()).cache()

idf = IDF(inputCol="termFreqs", 
          outputCol="tfidfVec")
idfModel = idf.fit(docTermFreqs)
docTermMatrix = idfModel.transform(docTermFreqs).select("title", "tfidfVec")
docTermMatrix.show()
```
<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    +--------------------+--------------------+
    |               title|            tfidfVec|
    +--------------------+--------------------+
    |     Bongard problem|(2000,[1,2,3,4,5,...|
    |    Generative model|(2000,[0,1,2,3,4,...|
    |      Inductive bias|(2000,[1,2,3,4,5,...|
    |Category:Bayesian...|(2000,[2,10,14,27...|
    |Category:Classifi...|(2000,[2,8,10,13,...|
    |Category:Evolutio...|(2000,[2,10,12,13...|
    |Semi-supervised l...|(2000,[0,1,2,3,4,...|
    |  Learning automaton|(2000,[1,2,3,4,5,...|
    |Category:Machine ...|(2000,[2,8,10,45,...|
    |Conditional rando...|(2000,[0,1,2,3,4,...|
    |Cross-entropy method|(2000,[0,1,2,3,4,...|
    |       Concept drift|(2000,[2,3,7,8,10...|
    |    Concept learning|(2000,[0,1,2,3,4,...|
    |      Robot learning|(2000,[1,2,5,6,7,...|
    |Version space lea...|(2000,[0,1,2,3,4,...|
    |Evolvability (com...|(2000,[0,2,3,5,7,...|
    |Prior knowledge f...|(2000,[0,1,2,7,8,...|
    |  Granular computing|(2000,[0,2,3,4,5,...|
    |Probability matching|(2000,[0,2,3,4,5,...|
    |Structural risk m...|(2000,[0,1,2,3,5,...|
    +--------------------+--------------------+
    only showing top 20 rows
</div>
</details> 

- CountVectorizer대신 HashingTF를 사용했을때 나중 데이터 처리할때 계산에서 오류가났었다.
- CountVectorizer와 HashingTF는 개념적으로 

<br/>

### TF-IDF값들에 대해 SVD 
```python
from pyspark.mllib.linalg.distributed import RowMatrix

from pyspark.mllib.util import MLUtils
vecDF = MLUtils.convertVectorColumnsFromML(docTermMatrix, "tfidfVec")
vecRDD = vecDF.select("tfidfVec").rdd.flatMap(lambda x:x)
mat = RowMatrix(vecRDD)
svd = mat.computeSVD(50, computeU=True)
```

- tfidfVec column의 값들을 SVD를 하기위해, convertVectorColumnsFromML, rdd.flatMap으로 데이터 포맷 맞춰줌

<br/>


```python
from pyspark.sql.functions import create_map
docIds = docTermFreqswithID.select(create_map('id', 'title').alias('map'))
docIds.show(5)

```
<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    +--------------------+
    |                 map|
    +--------------------+
    |[0 -> Bongard pro...|
    |[1 -> Generative ...|
    |[2 -> Inductive b...|
    |[3 -> Category:Ba...|
    |[4 -> Category:Cl...|
    +--------------------+
    only showing top 5 rows
    
</div>
</details> 


```python
v = svd.V
arr = v.toArray()
print(arr)
```
<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    [[-5.00858180e-02 -4.35775792e-02 -7.34478182e-01 ...  3.56759872e-02
       2.37933367e-03 -4.18013106e-02]
     [-1.20153559e-01 -3.41520254e-02 -3.08447681e-02 ... -1.37015740e-02
      -7.39789114e-03  5.90371606e-04]
     [ 1.85971700e-18  7.50783037e-18 -1.79453700e-19 ...  6.50199192e-18
       3.71956703e-19 -3.98281096e-18]
     ...
     [-8.78457403e-04 -7.14098445e-04 -3.51210501e-03 ...  1.00068501e-02
       7.24102615e-03  1.51146114e-03]
     [-4.25546919e-03 -1.31132128e-02  2.64652064e-03 ... -1.68947999e-02
      -1.44310037e-02 -3.66791509e-02]
     [-1.78161013e-03 -9.71648805e-05 -1.71628121e-03 ...  5.86748774e-03
       1.67048447e-03  4.57107444e-04]]

</div>
</details> 




```python
def topTermsInTopConcepts(svd, numConcepts, numTerms, termIds):
    arr = svd.V.toArray().transpose()
    res = []
    for i,v  in enumerate(arr):
        if( i > numConcepts ): break

        v = list(enumerate(v))
        v.sort(key=lambda x : x[1], reverse=True)
        v = v[0:numTerms]
        v = list((termIds[termId], score) for termId, score in v)
        res.append(v)
    return res
```


```python
topTermsInTopConcepts(svd, 4, 10, termIds)

```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 


    [[('learn', 1.8597169995295885e-18),
      ('machin', -1.6035925690269777e-18),
      ('categori', -2.151518488287089e-18),
      ('stub', -7.000211758918947e-05),
      ('leakag', -0.00018164111877602349),
      ('syntaxhighlight', -0.00026571460434930694),
      ('automl', -0.00028923376034158035),
      ('defaultsort', -0.00029098684759717563),
      ('grammat', -0.0003018289912002426),
      ('cohen', -0.0003102629786163099)],
     [('scope', 0.20737308029646853),
      ('col', 0.17076117435749236),
      ('width', 0.16774931818494626),
      ('style', 0.13944109951896713),
      ('dataset', 0.08764291274177796),
      ('et', 0.07907108650953734),
      ('al', 0.07708054580845672),
      ('text', 0.045301999677119204),
      ('imag', 0.04000949418161115),
      ('none', 0.03226686485644428)],
     [('defn', 0.1512680558263076),
      ('gli', 0.0945995509340557),
      ('scope', 0.08570611183432925),
      ('col', 0.06006244542456554),
      ('width', 0.054011334730391075),
      ('style', 0.04656215515172694),
      ('ghat', 0.02959268003578154),
      ('dataset', 0.024496733670962202),
      ('term', 0.020517386156565735),
      ('yes', 0.01979637788678219)],
     [('math', 0.12035446676646126),
      ('defn', 0.08440393286376088),
      ('scope', 0.05551200968539763),
      ('gli', 0.052785512582327394),
      ('col', 0.039741935288122546),
      ('mathbf', 0.03972109659217821),
      ('width', 0.038732257712885326),
      ('mathcal', 0.036889835499956),
      ('style', 0.03418748743436764),
      ('x', 0.028324704245546502)],
     [('quantum', 0.2315360596044999),
      ('defn', 0.22515417354363873),
      ('math', 0.20228975725189738),
      ('scope', 0.15304733820740882),
      ('gli', 0.14084361851145225),
      ('mathbf', 0.11779818858054367),
      ('col', 0.1043623802493719),
      ('mathcal', 0.09779095110059333),
      ('width', 0.09342344553382659),
      ('style', 0.07644497349047717)]]

</div>
</details> 



```python
def topDocsInTopConcept(svd, numConcepts, numDocs, docIds):
    u = svd.U
    res = []

    for i, u in enumerate(u.rows.map(lambda i : i.toArray()).collect()):
        if( i > numConcepts ): break
        u = list(enumerate(u))
        u.sort(key=lambda x: x[1], reverse=True)
        u = u[0:numDocs]
        u = list((docIds.collect()[docId][0][docId], score) for docId, score in u)
        res.append(u)
    return res
```


```python
topDocsInTopConcept(svd, 4, 10, docIds)

```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 


    [[('Uncertain data', 0.012153971612533395),
      ('Evolvability (computer science)', 0.011101410429071921),
      ('CIML community portal', 0.00883950570724018),
      ('Curse of dimensionality', 0.008634068295918167),
      ('Concept learning', 0.006993182301313434),
      ('Learning with errors', 0.006506466855122496),
      ('Probability matching', 0.006330621971353632),
      ('Ugly duckling theorem', 0.00459418746538158),
      ('Matthews correlation coefficient', 0.004528106337838542),
      ('Conditional random field', 0.0043785429594990375)],
     [('Data pre-processing', 0.05897285911052117),
      ('Matthews correlation coefficient', 0.04545270688773978),
      ('Learning to rank', 0.0431350496466185),
      ('Category:Machine learning researchers', 0.04070487278632756),
      ('Prior knowledge for pattern recognition', 0.040071601265590465),
      ('Granular computing', 0.035673146974228284),
      ('CIML community portal', 0.030504282474660145),
      ('Predictive state representation', 0.028612932855883837),
      ('Category:Learning in computer vision', 0.02721265731133607),
      ('Evolvability (computer science)', 0.02581816576294145)],
     [('Uniform convergence in probability', 0.018356433280116918),
      ('Ugly duckling theorem', 0.017158346365931196),
      ('Learning with errors', 0.01422857726997861),
      ('Category:Learning in computer vision', 0.013522047090777483),
      ('Predictive state representation', 0.010884224561886334),
      ('Center for Biological and Computational Learning', 0.0099396233259063),
      ('CIML community portal', 0.009856527832767602),
      ('Category:Ensemble learning', 0.009762324278326084),
      ('Eager learning', 0.009285613084181611),
      ('Probability matching', 0.008679169306842639)],
     [('Category:Ensemble learning', 0.0011468238013988086),
      ('CIML community portal', 0.0004482124621339058),
      ('Overfitting', 0.00035325684809392726),
      ('Neural modeling fields', 0.00031652554171964094),
      ('Knowledge integration', 0.0003111156445903527),
      ('Learning to rank', 0.0002654950538033083),
      ('Prior knowledge for pattern recognition', 0.00025042571026802596),
      ('Evolvability (computer science)', 0.0002481833462528634),
      ('Transduction (machine learning)', 0.00022341436144686583),
      ('Learning automaton', 0.00018789097710742732)],
     [('Neural modeling fields', 0.0007513902988702331),
      ('Matthews correlation coefficient', 0.0006970466348994987),
      ('Learning to rank', 0.0005868103050316529),
      ('Knowledge integration', 0.0005238659760471336),
      ('Expectation propagation', 0.00048574683907414985),
      ('Multiple-instance learning', 0.00038986405424075346),
      ('Transduction (machine learning)', 0.00038617563827325504),
      ('Evolvability (computer science)', 0.00037930596646474236),
      ('Semantic analysis (machine learning)', 0.00032205348767300423),
      ('Concept drift', 0.0003065307621572611)]]

</div>
</details> 


### 마무리
- NLP 전처리는 Tokenizer & Stopwords remove -> Empty space remove ->  Word Stemmed 순서로 이루어진다.
- pyspark.mllib에 클래스에 있는 SVD를 하기 위해 rdd.flat등과 같은 데이터 변형을 많이했다.
- Pandas_udf는 정말 강력하다
- 모델구성하는 코드는 아직 완벽히 이해 못한듯하다. LSA와 SVD에 대한 수학적인 이해는 하지만 스파크에서 처리하는 코드가 조금 복잡하다. 참고할 수 있는 코드가 없었으면 시간이 많이 걸렸을듯하다..

