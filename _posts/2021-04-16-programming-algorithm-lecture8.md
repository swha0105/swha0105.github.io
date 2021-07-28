---
layout: post
title:  "[Intro Algorithms] Hashing with Chaining"
subtitle:   "Introduction to Algorithms"
categories: programming
tags: algorithms
comments: true
---

# 8. Hashing with Chaining
- Dictionaries 
- Hash & Chaining
- Simple Uniform Hashing & Hash Function

<br/>

---

# Dictionary
Abstract Data Type(ADT)으로 나열한 Dictionary 기능

- **Insert item**: add item to set
- **Delete item**: remove item from set
- **Search key** : return item with key if it exists

**Goal**: 모든 연산을 상수 시간 ( O(1) ) 에 수행한다.

### Dictionary 사용 용도
- C,C++,Java 등 거의 모든 현대의 컴퓨터 언어에 구현되어있음
- Doclist (word count, lec 3: document distance)
- database 구축
- compilers & interpreters (name -> variable)
- network router (ip address -> wire)



### Dictionary 구현:

Dictionary 자료 구조의 goal인 **상수시간에 모든 연산**을 하기 위해 **`key-value`**를 pair로 저장한다. 이후 `value`에 해당되는 값을 `key`로 접근한다.

**direct-access table (simple approach)**

|![direct-access table](https://swha0105.github.io/assets/intro_algorithm/image/lec_8_direct_access_table.png)   
|:--:| 
| direct-access table |

- Dictionary의 item을 key로 인덱싱한 array에 저장

### Problems
사용하지 않는 key 값들도 저장해야되기 때문에 메모리가 비효율적으로 사용됨.  
위의 그림에서, item이 없는 key값들에 대한 array도 존재함. (keys = array[0:max_key_value])

<br/>

---

# Hash & Chaining

### Hash
- key가 될 수 있는 모든 값들의 집합들 중 데이터(해시 테이블) 크기에 적절하게 key값의 정의역을 축소한다.

- Simple uniform hashing을 제외한 hash mapping(**`key-hash, hash-value pair`**) 은 `hash function`을 통해 다양한 방법으로 한다.  ( `key-value`방법을 사용하는 **`direct-access table`** 방법과 차이)

**이러한 방법을 통해 memory를 효율적으로 사용한다.**

|![hash_mapping](https://swha0105.github.io/assets/intro_algorithm/image/lec_8_hash_mapping.png)   
|:--:| 
| hash mapping |

> U: key 값이 될 수 있는 모든 값들의 집합  
> k: key 값에 될 수 있는 축소된 정의역 (k1: key 값  )
> T: hash table  
> h: hash function  
> h(k1) = 1: hash
> m: 데이터의 값.

`데이터의 값`(m) 과 `해시 h`(k1)을 함께 저장하는 자료구조를 **`해시 테이블`** 이라 한다.


### Chaining 

만약 여러개의 **key가 같은 hash을 가르킨다면 `collision`**이 일어난다. 이럴 경우 `chaining` 개념을 통해 mapping 한다

|![Chaining](https://swha0105.github.io/assets/intro_algorithm/image/lec_8_chaining.png)   
|:--:| 
| Chaining |


<br/>

---

# Simple Uniform Hashing & Hash Function

### Simple Uniform Hashing
그렇다면 **key-hashing 을 pair하는 방법**을 알아보자.  
  
가장 간단한 `simple uniform hashing`은 두가지 가정을 가진다.

- 각각의 키는 hash table에 지정될 확률은 동일하다. (랜덤하게 지정)
- 각각의 키들은 독립적으로 hash mapping 된다. (잘못된 가정)

2번째 조건은 현실적으로 불가능하다고 언급한다. 하지만 여기서는 2번째 조건이 맞다고 가정 한 뒤, 진행한다.  

<br/>

key의 갯수가 n, hash table의 공간 (slot)이 m 이라고 하면 **`load factor`** $$(\alpha)$$ 는 $$\frac{n}{m}$$ (chain의 길이) 이 된다.   
이럴 경우, 모든 연산에 대한 시간 복잡도는 O(1 + $$(\alpha)$$) 가 된다.  (1은 hash function이 slot에 랜덤하게 접근하는 시간.)  
worst case일 경우, hash가 지정될 확률이 랜덤이기 때문에 한 slot에 모든 hash들이 mapping될 수 있다. 이 경우 $$(\alpha)$$가 = **n** 이 된다.  

<br/>

## hashing function
`key`를 `hash table`에 `random` 하게 `mapping` 하는 방법말고, **`hash function`**을 이용한 수학적으로 좋은방법들이 있다. 


1. Division method

h(k) = k mod m

> k: key  
> m: 사용자가 지정하는 숫자.

**단점:**
- k,m 이 서로소가 아닐 경우 hashing되는 map이 너무 작다.


2. Mulitiplication method
h(k) = [(a · k) mod 2w] >> (w − r)
> a: random
> k: key, and length is w bits (w bit machine)
> m: 2^r

# 작성중
..?

3. universal hashing
은 나중 강의에 나온다고 자세한 설명은 생략했다.

---
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
