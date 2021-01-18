# Apache Spark

오픈소스 분산 쿼리 및 처리 엔진, set of libraries 집합.

`Spark API`는 자바,스칼라,파이썬,R,SQL를 이용해 접근


# 스파크 잡과 API 

실행프로세스:
모든 Spark 애플리케이션은 **여러 개의 잡**을 가질 수 있는 하나의 `드라이버 프로세스` 를 `마스터 노드`에서 실행합니다.

`드라이버 프로세스`는 `실행 프로세스 (Executor Process)` 들을 컨트롤 합니다. 

태스크를 포함하고 있는 `실행 프로세스`들은 여러개의 워커노드로 태스크를 분산.

`드라이버 프로세스`는 `테스크 프로세스`의 개수와 구성을 결정. 

`테스크 프로세스`는 하나의 `잡 그래프`에 기반해 실행 노드에 의해 컨틀로 됨. 

ref: https://enterone.tistory.com/255 



# RDD
스파크는 RDD (Resilient Disstributed Datasets)라고 불리는 자바 가상 머신 객체들의 집합.  데이터가 저장되는곳. 
메모리상에서 캐시, 저장, 계산됨.


RDD에서 병렬 연산
- 새로운 RDD에 리턴하는 Tranformation (게으른 연산)
- 연산 후에 값을 리턴하는 Action

# 데이터 프레임
클라스터상 여러 노드에 분산된 이뮤터블 데이터 집합. 
칼럼명으로 이뤄져있으며 큰 데이이셋을 쉽게 처리하기 위해 디자인 됨.




apache hadoop

hadoop common
hadoop distributed file system
hadoop YARN
hadoop mapreduce

하둡 클러스터, 하둡 환경 


HDFS
구조
master 
: single namenode for managing metadata
, keeps the metadata, the name, location and directory 
slave
: multiple datanodes for storing data


HDFS files are divided into blocks (basic unit of read/write)


checkpointing

accumulator

broadcast: large readonly variable