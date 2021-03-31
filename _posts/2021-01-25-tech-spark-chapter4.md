---
layout: post
title:  "[모두연 풀잎스쿨 14기] Chapter 4"
subtitle:   "Random Forest"
categories: tech
tags: spark
comments: true
---

**본 포스팅은 모두의연구소(home.modulabs.co.kr) 풀잎스쿨에서 진행된 `Advanced Analytics with Spark` 과정 내용을 공유 및 정리한 자료입니다.**  
  
---

Chapter 4 제목은 `의사 결정 나무로 산림 식생 분포 예측하기`이다.  
산림 데이터셋을 받아 여러가지 특성 (토양, 기후 등등..)을 통해 산림 식생을 예측하는 챕터이다.  
사용된 알고리즘은 [의사 결정나무](https://swha0105.github.io/_posts/2021-02-04-ML_DL-Decision_Tree.markdown) 와 [랜덤 포레스트](https://swha0105.github.io/_posts/2021-02-05-ML_DL-Random_Forest.markdown) 이고 각 링크에 정리되어 있다.

이 코드는 모두연 스터디 퍼실님 [깃허브](https://github.com/dream2globe/advanced-spark)에서 긁어와 수정했습니다.

<br/>

---

## Spark Session setting 

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
    SparkSession.builder.appName(f"Advanced analytics with SPARK - Chapter 4")
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


## Load dataset and Preprocessing


```python
!cat /home/jovyan/work/ch04/data/covtype.info
```
<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    The Forest CoverType dataset
    
    
    1.	Title of Database:
    
    	Forest Covertype data
    
    
    2.	Sources:
    
    	(a) Original owners of database:
    		Remote Sensing and GIS Program
    		Department of Forest Sciences
    		College of Natural Resources
    		Colorado State University
    		Fort Collins, CO  80523
    		(contact Jock A. Blackard, jblackard 'at' fs.fed.us
    		      or Dr. Denis J. Dean, denis.dean 'at' utdallas.edu)
    
    	NOTE:	Reuse of this database is unlimited with retention of 
    		copyright notice for Jock A. Blackard and Colorado 
    		State University.
    
    	(b) Donors of database:
    		Jock A. Blackard (jblackard 'at' fs.fed.us)
    		GIS Coordinator
    		USFS - Forest Inventory & Analysis
    		Rocky Mountain Research Station
    		507 25th Street
    		Ogden, UT 84401
    
    		Dr. Denis J. Dean (denis.dean 'at' utdallas.edu)
    		Professor
    		Program in Geography and Geospatial Sciences
    		School of Economic, Political and Policy Sciences
    		800 West Campbell Rd
    		Richardson, TX  75080-3021 
    		
    		Dr. Charles W. Anderson (anderson 'at' cs.colostate.edu)
    		Associate Professor
    		Department of Computer Science
    		Colorado State University
    		Fort Collins, CO  80523  USA
    
    	(c) Date donated:  August 1998
    
    
    3.	Past Usage:
    
    	Blackard, Jock A. and Denis J. Dean.  2000.  "Comparative
    		Accuracies of Artificial Neural Networks and Discriminant
    		Analysis in Predicting Forest Cover Types from Cartographic
    		Variables."  Computers and Electronics in Agriculture 
    		24(3):131-151.
    
    	Blackard, Jock A. and Denis J. Dean.  1998.  "Comparative
    		Accuracies of Neural Networks and Discriminant Analysis
    		in Predicting Forest Cover Types from Cartographic 
    		Variables."  Second Southern Forestry GIS Conference.
    		University of Georgia.  Athens, GA.  Pages 189-199.
    
    	Blackard, Jock A.  1998.  "Comparison of Neural Networks and
    		Discriminant Analysis in Predicting Forest Cover Types."
    		Ph.D. dissertation.  Department of Forest Sciences.  
    		Colorado State University.  Fort Collins, Colorado.  
    		165 pages.
    
    	Abstract of dissertation:
    		Natural resource managers responsible for developing 
    	ecosystem management strategies require basic descriptive 
    	information including inventory data for forested lands to 
    	support their decision-making processes.  However, managers 
    	generally do not have this type of data for inholdings or 
    	neighboring lands that are outside their immediate 
    	jurisdiction.  One method of obtaining this information is 
    	through the use of predictive models.  
    		Two predictive models were examined in this study, a 
    	feedforward neural network model and a more traditional 
    	statistical model based on discriminant analysis.  The overall 
    	objectives of this research were to first construct these two 
    	predictive models, and second to compare and evaluate their 
    	respective classification accuracies when predicting forest 
    	cover types in undisturbed forests.  
    		The study area included four wilderness areas found in 
    	the Roosevelt National Forest of northern Colorado.  A total 
    	of twelve cartographic measures were utilized as independent 
    	variables in the predictive models, while seven major forest 
    	cover types were used as dependent variables.  Several subsets 
    	of these variables were examined to determine the best overall 
    	predictive model.  
    		For each subset of cartographic variables examined in 
    	this study, relative classification accuracies indicate the 
    	neural network approach outperformed the traditional 
    	discriminant analysis method in predicting forest cover types.  
    	The final neural network model had a higher absolute 
    	classification accuracy (70.58%) than the final corresponding 
    	linear discriminant analysis model(58.38%).  In support of these 
    	classification results, thirty additional networks with randomly 
    	selected initial weights were derived.  From these networks, the 
    	overall mean absolute classification accuracy for the neural 
    	network method was 70.52%, with a 95% confidence interval of 
    	70.26% to 70.80%.  Consequently, natural resource managers may 
    	utilize an alternative method of predicting forest cover types 
    	that is both superior to the traditional statistical methods and 
    	adequate to support their decision-making processes for 
    	developing ecosystem management strategies.
    
    
    	-- Classification performance
    		-- first 11,340 records used for training data subset
    		-- next 3,780 records used for validation data subset
    		-- last 565,892 records used for testing data subset
    		-- 70% Neural Network (backpropagation)
    		-- 58% Linear Discriminant Analysis
    
    
    4.	Relevant Information Paragraph:
    
    	Predicting forest cover type from cartographic variables only
    	(no remotely sensed data).  The actual forest cover type for
    	a given observation (30 x 30 meter cell) was determined from
    	US Forest Service (USFS) Region 2 Resource Information System 
    	(RIS) data.  Independent variables were derived from data
    	originally obtained from US Geological Survey (USGS) and
    	USFS data.  Data is in raw form (not scaled) and contains
    	binary (0 or 1) columns of data for qualitative independent
    	variables (wilderness areas and soil types).
    
    	This study area includes four wilderness areas located in the
    	Roosevelt National Forest of northern Colorado.  These areas
    	represent forests with minimal human-caused disturbances,
    	so that existing forest cover types are more a result of 
    	ecological processes rather than forest management practices.
    
    	Some background information for these four wilderness areas:  
    	Neota (area 2) probably has the highest mean elevational value of 
    	the 4 wilderness areas. Rawah (area 1) and Comanche Peak (area 3) 
    	would have a lower mean elevational value, while Cache la Poudre 
    	(area 4) would have the lowest mean elevational value. 
    
    	As for primary major tree species in these areas, Neota would have 
    	spruce/fir (type 1), while Rawah and Comanche Peak would probably
    	have lodgepole pine (type 2) as their primary species, followed by 
    	spruce/fir and aspen (type 5). Cache la Poudre would tend to have 
    	Ponderosa pine (type 3), Douglas-fir (type 6), and 
    	cottonwood/willow (type 4).  
    
    	The Rawah and Comanche Peak areas would tend to be more typical of 
    	the overall dataset than either the Neota or Cache la Poudre, due 
    	to their assortment of tree species and range of predictive 
    	variable values (elevation, etc.)  Cache la Poudre would probably 
    	be more unique than the others, due to its relatively low 
    	elevation range and species composition. 
    
    
    5.	Number of instances (observations):  581,012
    
    
    6.	Number of Attributes:	12 measures, but 54 columns of data
    				(10 quantitative variables, 4 binary
    				wilderness areas and 40 binary
    				soil type variables)
    
    
    7.	Attribute information:
    
    Given is the attribute name, attribute type, the measurement unit and
    a brief description.  The forest cover type is the classification 
    problem.  The order of this listing corresponds to the order of 
    numerals along the rows of the database.
    
    Name                                     Data Type    Measurement                       Description
    
    Elevation                               quantitative    meters                       Elevation in meters
    Aspect                                  quantitative    azimuth                      Aspect in degrees azimuth
    Slope                                   quantitative    degrees                      Slope in degrees
    Horizontal_Distance_To_Hydrology        quantitative    meters                       Horz Dist to nearest surface water features
    Vertical_Distance_To_Hydrology          quantitative    meters                       Vert Dist to nearest surface water features
    Horizontal_Distance_To_Roadways         quantitative    meters                       Horz Dist to nearest roadway
    Hillshade_9am                           quantitative    0 to 255 index               Hillshade index at 9am, summer solstice
    Hillshade_Noon                          quantitative    0 to 255 index               Hillshade index at noon, summer soltice
    Hillshade_3pm                           quantitative    0 to 255 index               Hillshade index at 3pm, summer solstice
    Horizontal_Distance_To_Fire_Points      quantitative    meters                       Horz Dist to nearest wildfire ignition points
    Wilderness_Area (4 binary columns)      qualitative     0 (absence) or 1 (presence)  Wilderness area designation
    Soil_Type (40 binary columns)           qualitative     0 (absence) or 1 (presence)  Soil Type designation
    Cover_Type (7 types)                    integer         1 to 7                       Forest Cover Type designation
    
    
    Code Designations:
    
    Wilderness Areas:  	1 -- Rawah Wilderness Area
                            2 -- Neota Wilderness Area
                            3 -- Comanche Peak Wilderness Area
                            4 -- Cache la Poudre Wilderness Area
    
    Soil Types:             1 to 40 : based on the USFS Ecological
                            Landtype Units (ELUs) for this study area:
    
      Study Code USFS ELU Code			Description
    	 1	   2702		Cathedral family - Rock outcrop complex, extremely stony.
    	 2	   2703		Vanet - Ratake families complex, very stony.
    	 3	   2704		Haploborolis - Rock outcrop complex, rubbly.
    	 4	   2705		Ratake family - Rock outcrop complex, rubbly.
    	 5	   2706		Vanet family - Rock outcrop complex complex, rubbly.
    	 6	   2717		Vanet - Wetmore families - Rock outcrop complex, stony.
    	 7	   3501		Gothic family.
    	 8	   3502		Supervisor - Limber families complex.
    	 9	   4201		Troutville family, very stony.
    	10	   4703		Bullwark - Catamount families - Rock outcrop complex, rubbly.
    	11	   4704		Bullwark - Catamount families - Rock land complex, rubbly.
    	12	   4744		Legault family - Rock land complex, stony.
    	13	   4758		Catamount family - Rock land - Bullwark family complex, rubbly.
    	14	   5101		Pachic Argiborolis - Aquolis complex.
    	15	   5151		unspecified in the USFS Soil and ELU Survey.
    	16	   6101		Cryaquolis - Cryoborolis complex.
    	17	   6102		Gateview family - Cryaquolis complex.
    	18	   6731		Rogert family, very stony.
    	19	   7101		Typic Cryaquolis - Borohemists complex.
    	20	   7102		Typic Cryaquepts - Typic Cryaquolls complex.
    	21	   7103		Typic Cryaquolls - Leighcan family, till substratum complex.
    	22	   7201		Leighcan family, till substratum, extremely bouldery.
    	23	   7202		Leighcan family, till substratum - Typic Cryaquolls complex.
    	24	   7700		Leighcan family, extremely stony.
    	25	   7701		Leighcan family, warm, extremely stony.
    	26	   7702		Granile - Catamount families complex, very stony.
    	27	   7709		Leighcan family, warm - Rock outcrop complex, extremely stony.
    	28	   7710		Leighcan family - Rock outcrop complex, extremely stony.
    	29	   7745		Como - Legault families complex, extremely stony.
    	30	   7746		Como family - Rock land - Legault family complex, extremely stony.
    	31	   7755		Leighcan - Catamount families complex, extremely stony.
    	32	   7756		Catamount family - Rock outcrop - Leighcan family complex, extremely stony.
    	33	   7757		Leighcan - Catamount families - Rock outcrop complex, extremely stony.
    	34	   7790		Cryorthents - Rock land complex, extremely stony.
    	35	   8703		Cryumbrepts - Rock outcrop - Cryaquepts complex.
    	36	   8707		Bross family - Rock land - Cryumbrepts complex, extremely stony.
    	37	   8708		Rock outcrop - Cryumbrepts - Cryorthents complex, extremely stony.
    	38	   8771		Leighcan - Moran families - Cryaquolls complex, extremely stony.
    	39	   8772		Moran family - Cryorthents - Leighcan family complex, extremely stony.
    	40	   8776		Moran family - Cryorthents - Rock land complex, extremely stony.
    
            Note:   First digit:  climatic zone             Second digit:  geologic zones
                    1.  lower montane dry                   1.  alluvium
                    2.  lower montane                       2.  glacial
                    3.  montane dry                         3.  shale
                    4.  montane                             4.  sandstone
                    5.  montane dry and montane             5.  mixed sedimentary
                    6.  montane and subalpine               6.  unspecified in the USFS ELU Survey
                    7.  subalpine                           7.  igneous and metamorphic
                    8.  alpine                              8.  volcanic
    
            The third and fourth ELU digits are unique to the mapping unit 
            and have no special meaning to the climatic or geologic zones.
    
    Forest Cover Type Classes:	1 -- Spruce/Fir
                                    2 -- Lodgepole Pine
                                    3 -- Ponderosa Pine
                                    4 -- Cottonwood/Willow
                                    5 -- Aspen
                                    6 -- Douglas-fir
                                    7 -- Krummholz
    
    
    8.  Basic Summary Statistics for quantitative variables only
    	(whole dataset -- thanks to Phil Rennert for the summary values):
    
    Name                                    Units             Mean   Std Dev
    Elevation                               meters          2959.36  279.98
    Aspect                                  azimuth          155.65  111.91
    Slope                                   degrees           14.10    7.49
    Horizontal_Distance_To_Hydrology        meters           269.43  212.55
    Vertical_Distance_To_Hydrology          meters            46.42   58.30
    Horizontal_Distance_To_Roadways         meters          2350.15 1559.25
    Hillshade_9am                           0 to 255 index   212.15   26.77
    Hillshade_Noon                          0 to 255 index   223.32   19.77
    Hillshade_3pm                           0 to 255 index   142.53   38.27
    Horizontal_Distance_To_Fire_Points      meters          1980.29 1324.19
    
    
    9.	Missing Attribute Values:  None.
    
    
    10.	Class distribution:
    
               Number of records of Spruce-Fir:                211840 
               Number of records of Lodgepole Pine:            283301 
               Number of records of Ponderosa Pine:             35754 
               Number of records of Cottonwood/Willow:           2747 
               Number of records of Aspen:                       9493 
               Number of records of Douglas-fir:                17367 
               Number of records of Krummholz:                  20510  
               Number of records of other:                          0  
    		
               Total records:                                  581012
    
    =====================================================================
    Jock A. Blackard
    08/28/1998 -- original text
    12/07/1999 -- updated mailing address, citations, background info 
    		  for study area, added summary statistics.
    =====================================================================
  
</div>
</details>    

<br/>


```python
!head /home/jovyan/work/ch04/data/covtype.data
```
<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    2596,51,3,258,0,510,221,232,148,6279,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,5
    2590,56,2,212,-6,390,220,235,151,6225,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,5
    2804,139,9,268,65,3180,234,238,135,6121,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2
    2785,155,18,242,118,3090,238,238,122,6211,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,2
    2595,45,2,153,-1,391,220,234,150,6172,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,5
    2579,132,6,300,-15,67,230,237,140,6031,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,2
    2606,45,7,270,5,633,222,225,138,6256,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,5
    2605,49,4,234,7,573,222,230,144,6228,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,5
    2617,45,9,240,56,666,223,221,133,6244,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,5
    2612,59,10,247,11,636,228,219,124,6230,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,5

</div>
</details> 


<br/>

```python
# Elevation                               quantitative    meters                       Elevation in meters
# Aspect                                  quantitative    azimuth                      Aspect in degrees azimuth
# Slope                                   quantitative    degrees                      Slope in degrees
# Horizontal_Distance_To_Hydrology        quantitative    meters                       Horz Dist to nearest surface water features
# Vertical_Distance_To_Hydrology          quantitative    meters                       Vert Dist to nearest surface water features
# Horizontal_Distance_To_Roadways         quantitative    meters                       Horz Dist to nearest roadway
# Hillshade_9am                           quantitative    0 to 255 index               Hillshade index at 9am, summer solstice
# Hillshade_Noon                          quantitative    0 to 255 index               Hillshade index at noon, summer soltice
# Hillshade_3pm                           quantitative    0 to 255 index               Hillshade index at 3pm, summer solstice
# Horizontal_Distance_To_Fire_Points      quantitative    meters                       Horz Dist to nearest wildfire ignition points
# Wilderness_Area (4 binary columns)      qualitative     0 (absence) or 1 (presence)  Wilderness area designation
# Soil_Type (40 binary columns)           qualitative     0 (absence) or 1 (presence)  Soil Type designation
# Cover_Type (7 types)                    integer         1 to 7                       Forest Cover Type designation

wilderness_area_cols = [f"wilderness_area_{i}" for i in range(4)] # 황야 지역 (4 dummy variables)
soil_type_cols = [f"soil_type_{i}" for i in range(40)] # 토양 유형 (40 dummy variables)
# 이 문법 기억하기!

schema = [
    T.StructField("elevation", T.DoubleType(), True),
    T.StructField("aspect", T.DoubleType(), True),
    T.StructField("slope", T.DoubleType(), True),
    T.StructField("horz_dist_to_hydro", T.DoubleType(), True), # 가장 가까운 지표수까지 거리
    T.StructField("vert_dist_to_hydro", T.DoubleType(), True), # 가장 가까운 지표수까지 거리
    T.StructField("horz_dist_to_road", T.DoubleType(), True), # 가장 가까운 도로까지 거리
    T.StructField("hillshade_9am", T.IntegerType(), True), # 언덕 그늘
    T.StructField("hillshade_noon", T.IntegerType(), True),
    T.StructField("hillshade_3pm", T.IntegerType(), True),
    T.StructField("horz_dist_to_fire", T.DoubleType(), True),
]

wilderness_area_schema = [T.StructField(col, T.IntegerType(), True) for col in wilderness_area_cols] 
soil_type_schema = [T.StructField(col, T.IntegerType(), True) for col in soil_type_cols] 
cover_type_schema = [T.StructField("cover_type", T.IntegerType(), True)]

#wilderness_area_schema 너무길어서 따로 만들어 합침. 

schema.extend(wilderness_area_schema)
schema.extend(soil_type_schema)
schema.extend(cover_type_schema)
schema = T.StructType(schema)
```


```python
df = (spark
      .read.format("csv")
      .option("header", False)
      .option("sep", ",")
      .schema(schema)
      .load(ref_path + "data/covtype.data"))
      

wilderness_area_cols = [f"wilderness_area_{i}" for i in range(4)]
soil_type_cols = [f"soil_type_{i}" for i in range(40)]

df.select("cover_type").show(5) # 모든 컬럼의 스키마가 반영되었는지 확인
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    +----------+
    |cover_type|
    +----------+
    |         5|
    |         5|
    |         2|
    |         2|
    |         5|
    +----------+
    only showing top 5 rows

</div>
</details>   

<br/>

- schema를 정할때 `inferschema`를 사용할 수 있지만 데이터 구조를 알수있는 경우 직접 schema를 구성하는걸 추천.
- 데이터를 보게되면 `wildress_type` 과 `soil_type`은 줄줄이 나열되어있는 **One-hot encoding**으로 저장되어있다. 이 같은 경우 굉장히 비효율적으로 데이터를 처리할 수 밖에없기 때문에 챕터 중간 쯤 다른 방법으로 처리를 하게 된다.
- **One-hot encoding**은 columns수가 많기 때문에 따로 schema를 만들어 원래 schema에 extend함.

<br/>

---

## Prepare dataset


```python
from pyspark.ml.feature import RFormula

print(f"the number of column: {len(df.columns)}")

transformer = RFormula(formula="cover_type ~ .").fit(df)

prepared_df = transformer.transform(df).select("features", "label")
prepared_df.show(5, False) # features와 label이 추가됨

train_df, test_df = prepared_df.randomSplit([0.7, 0.3], seed=42)
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    the number of column: 55
    +--------------------------------------------------------------------------------------------------------+-----+
    |features                                                                                                |label|
    +--------------------------------------------------------------------------------------------------------+-----+
    |(54,[0,1,2,3,5,6,7,8,9,10,42],[2596.0,51.0,3.0,258.0,510.0,221.0,232.0,148.0,6279.0,1.0,1.0])           |5.0  |
    |(54,[0,1,2,3,4,5,6,7,8,9,10,42],[2590.0,56.0,2.0,212.0,-6.0,390.0,220.0,235.0,151.0,6225.0,1.0,1.0])    |5.0  |
    |(54,[0,1,2,3,4,5,6,7,8,9,10,25],[2804.0,139.0,9.0,268.0,65.0,3180.0,234.0,238.0,135.0,6121.0,1.0,1.0])  |2.0  |
    |(54,[0,1,2,3,4,5,6,7,8,9,10,43],[2785.0,155.0,18.0,242.0,118.0,3090.0,238.0,238.0,122.0,6211.0,1.0,1.0])|2.0  |
    |(54,[0,1,2,3,4,5,6,7,8,9,10,42],[2595.0,45.0,2.0,153.0,-1.0,391.0,220.0,234.0,150.0,6172.0,1.0,1.0])    |5.0  |
    +--------------------------------------------------------------------------------------------------------+-----+
    only showing top 5 rows
    
</div>
</details>  

<br/>

- RFormula는 Estimator이다. (Estimator와 Transformer에 대해 자세한건 다음 챕터..)
- `.select`를 통해 prepared_df에는 `features`와 `label`만 넘겨줌. 
- prepared_df의 show의 결과를 보게 되면 sparse vector에 대한 저장공간을 줄이기 위해 압축해서 저장한것을 알 수 있다.
   - 원래라면 schema columns의 총 개수인 55개와 값이 나와야한다.
   - 데이터에 0 이 많은 sparse matrix(vector) 라면 다음과 같이 저장이 된다.
   - [0이 아닌 컬럼갯수, 0이 아닌 컬럼의 인덱스, 0이 아닌 컬럼의 값]

<br/>

---

## Modeling


```python
from pyspark.ml import Pipeline
from pyspark.ml.feature import StringIndexer, VectorIndexer
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

def build_pipeline(ml_model):
    # Index labels, adding metadata to the label column.
    # Fit on whole dataset to include all labels in index.
    label_indexer = StringIndexer(inputCol="label", outputCol="indexedLabel").fit(prepared_df)

    # Automatically identify categorical features, and index them.
    # We specify maxCategories so features with > 4 distinct values are treated as continuous.
    feature_indexer = VectorIndexer(inputCol="features", outputCol="indexedFeatures", maxCategories=8).fit(prepared_df)

    # Chain indexers and tree in a Pipeline
    stages = [label_indexer, feature_indexer, ml_model]
    pipeline = Pipeline(stages=stages)
    return pipeline

def predict(train_df, test_df, pipeline, summarize_result=False, summarize_model=False):
    model = pipeline.fit(train_df)
    # Make predictions.
    predictions = model.transform(test_df)
    if summarize_result:
        predictions.select("indexedLabel", "prediction", "probability").show(5, False)
    if summarize_model:
        treeModel = model.stages[2]
        print(treeModel)
    return predictions

def evaluate(predictions):
    evaluator = MulticlassClassificationEvaluator(labelCol="indexedLabel",
                                                  predictionCol="prediction",
                                                  metricName="accuracy")
    accuracy = evaluator.evaluate(predictions)
    print("Test Error = %g " % (1.0 - accuracy))
    return accuracy
```

- stringindexer 를 통해 string으로 지정되어있는 label 값들을 numeric value로 바꿔줌.
- vectorindexer를 통해 categorical columns을 정리한다.
  - `soil_type` 같은 경우 40개의 categorical columns가 있고, 이 모든 columns를 쓰는건 비효율적이기 때문에 maxCategories 값인 8로 reduce 한다.

<br/>

### Decision Tree (Spark Documents)


```python
from pyspark.ml.classification import DecisionTreeClassifier

# Train a DecisionTree model.
dt = DecisionTreeClassifier(labelCol="indexedLabel",
                            featuresCol="indexedFeatures",
                            predictionCol="prediction",
                            seed=42)
#print(dt.explainParams())

pipeline = build_pipeline(dt)
predictions = predict(train_df, test_df, pipeline, True, True)
evaluate(predictions)
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 

    +------------+----------+------------------------------------------------------------------------------------------------------------------------------+
    |indexedLabel|prediction|probability                                                                                                                   |
    +------------+----------+------------------------------------------------------------------------------------------------------------------------------+
    |0.0         |0.0       |[0.917094017094017,0.03632478632478633,0.0,0.0,0.0,0.04658119658119658,0.0]                                                   |
    |0.0         |0.0       |[0.777401916890525,0.14035087719298245,0.015247239080688488,3.316419593406958E-5,0.019649786090936226,0.04731701654893377,0.0]|
    |0.0         |0.0       |[0.777401916890525,0.14035087719298245,0.015247239080688488,3.316419593406958E-5,0.019649786090936226,0.04731701654893377,0.0]|
    |0.0         |0.0       |[0.777401916890525,0.14035087719298245,0.015247239080688488,3.316419593406958E-5,0.019649786090936226,0.04731701654893377,0.0]|
    |0.0         |0.0       |[0.777401916890525,0.14035087719298245,0.015247239080688488,3.316419593406958E-5,0.019649786090936226,0.04731701654893377,0.0]|
    +------------+----------+------------------------------------------------------------------------------------------------------------------------------+
    only showing top 5 rows
    
    DecisionTreeClassificationModel: uid=DecisionTreeClassifier_83e630d42475, depth=5, numNodes=37, numClasses=7, numFeatures=54
    Test Error = 0.298203 





    0.7017968488871985

</div>
</details>  

- Decision tree 모델을 구성할때, Label column Feature column, predicion column을 다 구성한다. 이때 action은 일어나지 않는다. 
- Decision tree 모델을 build_pipeline에 넣어 pipeline을 구성한다.

<br/>

### Confusion Matrix
#### Spark API


```python
from pyspark.mllib.evaluation import MulticlassMetrics
import pandas as pd

# important: need to cast to float type, and order by prediction, else it won't work
# select only prediction and label columns
preds_and_labels = predictions.select('prediction','indexedLabel').orderBy('prediction') # 컬럼순서 주의(pred, real 순) 
metrics = MulticlassMetrics(preds_and_labels.rdd.map(tuple))

cm = metrics.confusionMatrix().toArray() # python list
pd.DataFrame(cm)
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>64968.0</td>
      <td>16608.0</td>
      <td>2592.0</td>
      <td>225.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>24.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>18337.0</td>
      <td>43329.0</td>
      <td>92.0</td>
      <td>1633.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>742.0</td>
      <td>0.0</td>
      <td>9750.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>199.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>79.0</td>
      <td>2643.0</td>
      <td>11.0</td>
      <td>3415.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1078.0</td>
      <td>0.0</td>
      <td>4011.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>144.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2559.0</td>
      <td>24.0</td>
      <td>249.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>496.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>318.0</td>
    </tr>
  </tbody>
</table>
</div>

</div>
</details>  


#### Custom


```python
cm = (predictions.select('prediction', 'indexedLabel')
      .groupby("indexedLabel")
      .pivot("prediction", list(range(0, 7)))
      .count()
      .fillna(0)
      .sort("indexedLabel")
      .toPandas())
cm = cm.reindex(cm["indexedLabel"], axis=0).drop("indexedLabel", axis=1)
cm
```


<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
    </tr>
    <tr>
      <th>indexedLabel</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0.0</th>
      <td>64968</td>
      <td>16608</td>
      <td>2592</td>
      <td>225</td>
      <td>0</td>
      <td>0</td>
      <td>24</td>
    </tr>
    <tr>
      <th>1.0</th>
      <td>18337</td>
      <td>43329</td>
      <td>92</td>
      <td>1633</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2.0</th>
      <td>742</td>
      <td>0</td>
      <td>9750</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>199</td>
    </tr>
    <tr>
      <th>3.0</th>
      <td>79</td>
      <td>2643</td>
      <td>11</td>
      <td>3415</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4.0</th>
      <td>1078</td>
      <td>0</td>
      <td>4011</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>144</td>
    </tr>
    <tr>
      <th>5.0</th>
      <td>2559</td>
      <td>24</td>
      <td>249</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6.0</th>
      <td>0</td>
      <td>0</td>
      <td>496</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>318</td>
    </tr>
  </tbody>
</table>
</div>


</div>
</details>  

- 예측값과 label값을 한눈에 보기 위해 confusion matrix 구성.

<br/>

## Hyperparameter Tuning


```python
from pyspark.ml.tuning import ParamGridBuilder, TrainValidationSplit

param_grid = (ParamGridBuilder()
              .addGrid(dt.impurity, ["entropy"])
              .addGrid(dt.maxDepth, [30]) # must <= 30
              .addGrid(dt.maxBins, [470])
              .addGrid(dt.minInfoGain, [0.0])
              .build())

evaluator = MulticlassClassificationEvaluator(labelCol="indexedLabel",
                                              predictionCol="prediction",
                                              metricName="accuracy")

validator = TrainValidationSplit(estimator=pipeline, estimatorParamMaps=param_grid, evaluator=evaluator, parallelism=4, seed=42)
validated_model = validator.fit(train_df)

validated_model.bestModel.stages[-1].extractParamMap()
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 



    {Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='cacheNodeIds', doc='If false, the algorithm will pass trees to executors to match instances with nodes. If true, the algorithm will cache node IDs for each instance. Caching can speed up training of deeper trees. Users can set how often should the cache be checkpointed or disable it by setting checkpointInterval.'): False,
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='checkpointInterval', doc='set checkpoint interval (>= 1) or disable checkpoint (-1). E.g. 10 means that the cache will get checkpointed every 10 iterations. Note: this setting will be ignored if the checkpoint directory is not set in the SparkContext.'): 10,
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='featuresCol', doc='features column name.'): 'indexedFeatures',
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='impurity', doc='Criterion used for information gain calculation (case-insensitive). Supported options: entropy, gini'): 'entropy',
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='labelCol', doc='label column name.'): 'indexedLabel',
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='leafCol', doc='Leaf indices column name. Predicted leaf index of each instance in each tree by preorder.'): '',
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='maxBins', doc='Max number of bins for discretizing continuous features.  Must be >=2 and >= number of categories for any categorical feature.'): 470,
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='maxDepth', doc='Maximum depth of the tree. (>= 0) E.g., depth 0 means 1 leaf node; depth 1 means 1 internal node + 2 leaf nodes.'): 30,
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='maxMemoryInMB', doc='Maximum memory in MB allocated to histogram aggregation. If too small, then 1 node will be split per iteration, and its aggregates may exceed this size.'): 256,
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='minInfoGain', doc='Minimum information gain for a split to be considered at a tree node.'): 0.0,
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='minInstancesPerNode', doc='Minimum number of instances each child must have after split. If a split causes the left or right child to have fewer than minInstancesPerNode, the split will be discarded as invalid. Should be >= 1.'): 1,
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='minWeightFractionPerNode', doc='Minimum fraction of the weighted sample count that each child must have after split. If a split causes the fraction of the total weight in the left or right child to be less than minWeightFractionPerNode, the split will be discarded as invalid. Should be in interval [0.0, 0.5).'): 0.0,
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='predictionCol', doc='prediction column name.'): 'prediction',
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='probabilityCol', doc='Column name for predicted class conditional probabilities. Note: Not all models output well-calibrated probability estimates! These probabilities should be treated as confidences, not precise probabilities.'): 'probability',
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='rawPredictionCol', doc='raw prediction (a.k.a. confidence) column name.'): 'rawPrediction',
     Param(parent='DecisionTreeClassifier_3f5fdea9e4a0', name='seed', doc='random seed.'): 42}


</div>
</details> 

- **`ParamGridBuilder` 는 지정된 모든 hyperparameter에 대해 grid search함**
- `TrainVildationSplit`에서 estimatorParamMaps를 설정하면 bestmodel attribute 생성. 

<br/>

```python
predictions = validated_model.bestModel.transform(test_df)
evaluate(predictions)

cm = (predictions.select('prediction', 'indexedLabel')
      .groupby("indexedLabel")
      .pivot("prediction", list(range(0, 7)))
      .count()
      .fillna(0)
      .sort("indexedLabel")
      .toPandas())
cm = cm.reindex(cm["indexedLabel"], axis=0).drop("indexedLabel", axis=1)
cm
```

    Test Error = 0.0644591 


<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
    </tr>
    <tr>
      <th>indexedLabel</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0.0</th>
      <td>80172</td>
      <td>3747</td>
      <td>224</td>
      <td>53</td>
      <td>174</td>
      <td>356</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1.0</th>
      <td>3840</td>
      <td>59434</td>
      <td>7</td>
      <td>289</td>
      <td>6</td>
      <td>57</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2.0</th>
      <td>223</td>
      <td>2</td>
      <td>10029</td>
      <td>0</td>
      <td>410</td>
      <td>29</td>
      <td>91</td>
    </tr>
    <tr>
      <th>3.0</th>
      <td>55</td>
      <td>300</td>
      <td>0</td>
      <td>5726</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4.0</th>
      <td>200</td>
      <td>22</td>
      <td>453</td>
      <td>0</td>
      <td>4509</td>
      <td>8</td>
      <td>37</td>
    </tr>
    <tr>
      <th>5.0</th>
      <td>416</td>
      <td>46</td>
      <td>26</td>
      <td>0</td>
      <td>19</td>
      <td>2381</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6.0</th>
      <td>0</td>
      <td>0</td>
      <td>89</td>
      <td>0</td>
      <td>44</td>
      <td>0</td>
      <td>680</td>
    </tr>
  </tbody>
</table>
</div>


</div>
</details> 

<br/>


## Feature Engineering & Random forest 


```python
cols = [col for col in df.columns if (col.find("wilderness_area_") == -1)] 
new_df = df.select(*cols, F.array(*wilderness_area_cols).alias("wilderness_area"))

print(f"the number of column: {len(new_df.columns)}")
new_df.select("wilderness_area").show(5)
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 


    the number of column: 52
    +---------------+
    |wilderness_area|
    +---------------+
    |   [1, 0, 0, 0]|
    |   [1, 0, 0, 0]|
    |   [1, 0, 0, 0]|
    |   [1, 0, 0, 0]|
    |   [1, 0, 0, 0]|
    +---------------+
    only showing top 5 rows
    

</div>
</details> 


```python
from typing import Iterator
import numpy as np
import pandas as pd

# Declare the function and create the UDF
@F.pandas_udf(T.LongType())
def unhot_udf(arrs: pd.Series) -> pd.Series:
    return pd.Series(np.where(arr==1)[0][0] for arr in arrs)

# Execute function as a Spark vectorized UDF
new_df = new_df.select(*cols, unhot_udf(F.col("wilderness_area")).alias("wilderness_area"))
transformer = RFormula(formula="cover_type ~ .").fit(new_df)
prepared_df = transformer.transform(new_df).select("features", "label")
prepared_df.show(5, False)

train_df, test_df = prepared_df.randomSplit([0.7, 0.3], seed=42)
train_df.cache()
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 


    +-------------------------------------------------------------------------------------------------+-----+
    |features                                                                                         |label|
    +-------------------------------------------------------------------------------------------------+-----+
    |(51,[0,1,2,3,5,6,7,8,9,38],[2596.0,51.0,3.0,258.0,510.0,221.0,232.0,148.0,6279.0,1.0])           |5.0  |
    |(51,[0,1,2,3,4,5,6,7,8,9,38],[2590.0,56.0,2.0,212.0,-6.0,390.0,220.0,235.0,151.0,6225.0,1.0])    |5.0  |
    |(51,[0,1,2,3,4,5,6,7,8,9,21],[2804.0,139.0,9.0,268.0,65.0,3180.0,234.0,238.0,135.0,6121.0,1.0])  |2.0  |
    |(51,[0,1,2,3,4,5,6,7,8,9,39],[2785.0,155.0,18.0,242.0,118.0,3090.0,238.0,238.0,122.0,6211.0,1.0])|2.0  |
    |(51,[0,1,2,3,4,5,6,7,8,9,38],[2595.0,45.0,2.0,153.0,-1.0,391.0,220.0,234.0,150.0,6172.0,1.0])    |5.0  |
    +-------------------------------------------------------------------------------------------------+-----+
    only showing top 5 rows
    





    DataFrame[features: vector, label: double]


</div>
</details> 


- onehot으로 분리된 열들을 한 열로 통합
- pandas udf를 사용함
- soil_type은 indexer 사용 불가(카디널리티가 30 이하여야함)


```python
from pyspark.ml.classification import RandomForestClassifier

rf = RandomForestClassifier(labelCol="indexedLabel",
                            featuresCol="indexedFeatures",
                            predictionCol="prediction",
                            numTrees=100)
pipeline = build_pipeline(rf)

param_grid = (ParamGridBuilder()
              .addGrid(dt.maxDepth, [1, 10, 20]) # must <= 30
              .addGrid(dt.maxBins, [10, 20, 30, 40, 50])
              .addGrid(dt.minInfoGain, [0.0, 0.05])
              .build())

evaluator = MulticlassClassificationEvaluator(labelCol="indexedLabel",
                                              predictionCol="prediction",
                                              metricName="accuracy")

validator = TrainValidationSplit(estimator=pipeline, estimatorParamMaps=param_grid, evaluator=evaluator, parallelism=1, seed=42)
validated_model = validator.fit(train_df)
validated_model.bestModel.stages[2].extractParamMap()
```

<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 


    {Param(parent='RandomForestClassifier_ba60fe650fb8', name='bootstrap', doc='Whether bootstrap samples are used when building trees.'): True,
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='cacheNodeIds', doc='If false, the algorithm will pass trees to executors to match instances with nodes. If true, the algorithm will cache node IDs for each instance. Caching can speed up training of deeper trees. Users can set how often should the cache be checkpointed or disable it by setting checkpointInterval.'): False,
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='checkpointInterval', doc='set checkpoint interval (>= 1) or disable checkpoint (-1). E.g. 10 means that the cache will get checkpointed every 10 iterations. Note: this setting will be ignored if the checkpoint directory is not set in the SparkContext.'): 10,
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='featureSubsetStrategy', doc="The number of features to consider for splits at each tree node. Supported options: 'auto' (choose automatically for task: If numTrees == 1, set to 'all'. If numTrees > 1 (forest), set to 'sqrt' for classification and to 'onethird' for regression), 'all' (use all features), 'onethird' (use 1/3 of the features), 'sqrt' (use sqrt(number of features)), 'log2' (use log2(number of features)), 'n' (when n is in the range (0, 1.0], use n * number of features. When n is in the range (1, number of features), use n features). default = 'auto'"): 'auto',
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='featuresCol', doc='features column name.'): 'indexedFeatures',
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='impurity', doc='Criterion used for information gain calculation (case-insensitive). Supported options: entropy, gini'): 'gini',
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='labelCol', doc='label column name.'): 'indexedLabel',
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='leafCol', doc='Leaf indices column name. Predicted leaf index of each instance in each tree by preorder.'): '',
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='maxBins', doc='Max number of bins for discretizing continuous features.  Must be >=2 and >= number of categories for any categorical feature.'): 32,
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='maxDepth', doc='Maximum depth of the tree. (>= 0) E.g., depth 0 means 1 leaf node; depth 1 means 1 internal node + 2 leaf nodes.'): 5,
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='maxMemoryInMB', doc='Maximum memory in MB allocated to histogram aggregation. If too small, then 1 node will be split per iteration, and its aggregates may exceed this size.'): 256,
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='minInfoGain', doc='Minimum information gain for a split to be considered at a tree node.'): 0.0,
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='minInstancesPerNode', doc='Minimum number of instances each child must have after split. If a split causes the left or right child to have fewer than minInstancesPerNode, the split will be discarded as invalid. Should be >= 1.'): 1,
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='minWeightFractionPerNode', doc='Minimum fraction of the weighted sample count that each child must have after split. If a split causes the fraction of the total weight in the left or right child to be less than minWeightFractionPerNode, the split will be discarded as invalid. Should be in interval [0.0, 0.5).'): 0.0,
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='numTrees', doc='Number of trees to train (>= 1).'): 100,
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='predictionCol', doc='prediction column name.'): 'prediction',
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='probabilityCol', doc='Column name for predicted class conditional probabilities. Note: Not all models output well-calibrated probability estimates! These probabilities should be treated as confidences, not precise probabilities.'): 'probability',
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='rawPredictionCol', doc='raw prediction (a.k.a. confidence) column name.'): 'rawPrediction',
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='seed', doc='random seed.'): -6894470631950539095,
     Param(parent='RandomForestClassifier_ba60fe650fb8', name='subsamplingRate', doc='Fraction of the training data used for learning each decision tree, in range (0, 1].'): 1.0}


</div>
</details> 


<br/>

```python
predictions = validated_model.bestModel.transform(test_df)
evaluator.evaluate(predictions)

cm = (predictions.select('prediction', 'indexedLabel')
      .groupby("indexedLabel")
      .pivot("prediction", list(range(0, 7)))
      .count()
      .fillna(0)
      .sort("indexedLabel")
      .toPandas())
cm = cm.reindex(cm["indexedLabel"], axis=0).drop("indexedLabel", axis=1)
cm
```


<details>    
<summary> 실행 결과 </summary>
<div markdown="1"> 


    0.6724794294802965


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
    </tr>
    <tr>
      <th>indexedLabel</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0.0</th>
      <td>63756</td>
      <td>20307</td>
      <td>837</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1.0</th>
      <td>16782</td>
      <td>46768</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2.0</th>
      <td>4079</td>
      <td>0</td>
      <td>6593</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3.0</th>
      <td>28</td>
      <td>6120</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4.0</th>
      <td>2298</td>
      <td>0</td>
      <td>2936</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5.0</th>
      <td>2848</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6.0</th>
      <td>0</td>
      <td>0</td>
      <td>805</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>


</div>
</details> 

## Feature Importance


```python
best_model = validated_model.bestModel.stages[-1]
importances = [(k, v) for v, k in zip(best_model.featureImportances.toArray(), new_df.columns)] # 마지막은 wilderness_area
pd.DataFrame(importances, columns=["feature", "importance"]).sort_values("importance", ascending=False)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>feature</th>
      <th>importance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>elevation</td>
      <td>0.389976</td>
    </tr>
    <tr>
      <th>50</th>
      <td>cover_type</td>
      <td>0.188755</td>
    </tr>
    <tr>
      <th>19</th>
      <td>soil_type_9</td>
      <td>0.068208</td>
    </tr>
    <tr>
      <th>31</th>
      <td>soil_type_21</td>
      <td>0.063238</td>
    </tr>
    <tr>
      <th>21</th>
      <td>soil_type_11</td>
      <td>0.046120</td>
    </tr>
    <tr>
      <th>13</th>
      <td>soil_type_3</td>
      <td>0.040963</td>
    </tr>
    <tr>
      <th>47</th>
      <td>soil_type_37</td>
      <td>0.030435</td>
    </tr>
    <tr>
      <th>5</th>
      <td>horz_dist_to_road</td>
      <td>0.023659</td>
    </tr>
    <tr>
      <th>32</th>
      <td>soil_type_22</td>
      <td>0.023002</td>
    </tr>
    <tr>
      <th>48</th>
      <td>soil_type_38</td>
      <td>0.020977</td>
    </tr>
    <tr>
      <th>11</th>
      <td>soil_type_1</td>
      <td>0.020692</td>
    </tr>
    <tr>
      <th>9</th>
      <td>horz_dist_to_fire</td>
      <td>0.019589</td>
    </tr>
    <tr>
      <th>15</th>
      <td>soil_type_5</td>
      <td>0.012061</td>
    </tr>
    <tr>
      <th>49</th>
      <td>soil_type_39</td>
      <td>0.006948</td>
    </tr>
    <tr>
      <th>2</th>
      <td>slope</td>
      <td>0.006745</td>
    </tr>
    <tr>
      <th>38</th>
      <td>soil_type_28</td>
      <td>0.005238</td>
    </tr>
    <tr>
      <th>3</th>
      <td>horz_dist_to_hydro</td>
      <td>0.004976</td>
    </tr>
    <tr>
      <th>7</th>
      <td>hillshade_noon</td>
      <td>0.004722</td>
    </tr>
    <tr>
      <th>6</th>
      <td>hillshade_9am</td>
      <td>0.004299</td>
    </tr>
    <tr>
      <th>1</th>
      <td>aspect</td>
      <td>0.003177</td>
    </tr>
    <tr>
      <th>12</th>
      <td>soil_type_2</td>
      <td>0.002707</td>
    </tr>
    <tr>
      <th>22</th>
      <td>soil_type_12</td>
      <td>0.002553</td>
    </tr>
    <tr>
      <th>8</th>
      <td>hillshade_3pm</td>
      <td>0.002431</td>
    </tr>
    <tr>
      <th>4</th>
      <td>vert_dist_to_hydro</td>
      <td>0.002378</td>
    </tr>
    <tr>
      <th>10</th>
      <td>soil_type_0</td>
      <td>0.002299</td>
    </tr>
    <tr>
      <th>39</th>
      <td>soil_type_29</td>
      <td>0.001036</td>
    </tr>
    <tr>
      <th>20</th>
      <td>soil_type_10</td>
      <td>0.000800</td>
    </tr>
    <tr>
      <th>42</th>
      <td>soil_type_32</td>
      <td>0.000605</td>
    </tr>
    <tr>
      <th>41</th>
      <td>soil_type_31</td>
      <td>0.000586</td>
    </tr>
    <tr>
      <th>40</th>
      <td>soil_type_30</td>
      <td>0.000241</td>
    </tr>
    <tr>
      <th>27</th>
      <td>soil_type_17</td>
      <td>0.000191</td>
    </tr>
    <tr>
      <th>26</th>
      <td>soil_type_16</td>
      <td>0.000089</td>
    </tr>
    <tr>
      <th>44</th>
      <td>soil_type_34</td>
      <td>0.000085</td>
    </tr>
    <tr>
      <th>46</th>
      <td>soil_type_36</td>
      <td>0.000067</td>
    </tr>
    <tr>
      <th>33</th>
      <td>soil_type_23</td>
      <td>0.000053</td>
    </tr>
    <tr>
      <th>30</th>
      <td>soil_type_20</td>
      <td>0.000039</td>
    </tr>
    <tr>
      <th>14</th>
      <td>soil_type_4</td>
      <td>0.000031</td>
    </tr>
    <tr>
      <th>23</th>
      <td>soil_type_13</td>
      <td>0.000026</td>
    </tr>
    <tr>
      <th>25</th>
      <td>soil_type_15</td>
      <td>0.000006</td>
    </tr>
    <tr>
      <th>29</th>
      <td>soil_type_19</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>28</th>
      <td>soil_type_18</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>24</th>
      <td>soil_type_14</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>37</th>
      <td>soil_type_27</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>18</th>
      <td>soil_type_8</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>43</th>
      <td>soil_type_33</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>36</th>
      <td>soil_type_26</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>45</th>
      <td>soil_type_35</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>35</th>
      <td>soil_type_25</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>17</th>
      <td>soil_type_7</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>16</th>
      <td>soil_type_6</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>34</th>
      <td>soil_type_24</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>

<br/>



## 마무리
- feature들을 VectorIndexer를 통해 categorical feature로 변환
- sparse vector는 저장공간을 줄이기 위해 값과, 인덱스 번호만 저장되어있는 형태로 따로 저장 
- pipeline으로 모델을 구성하는건 익숙하진 않지만 Data의 흐름에 따라 코드 읽기가 쉽다.
- **ParamGridBuilder 을 통해 hyperparameter set를 찾기!!**

<br/>

---



## Referrence
- pandas udf: https://docs.microsoft.com/ko-kr/azure/databricks/spark/latest/spark-sql/udf-python-pandas