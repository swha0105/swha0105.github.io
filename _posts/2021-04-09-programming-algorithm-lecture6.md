---
layout: post
title:  "[Intro Algorithms] Balanced Binary Search Tree"
subtitle:   "Introduction to Algorithms"
categories: programming
tags: cs
comments: true
---

# 6. Balanced Binary Search Tree
- Importance of being balanced 
- AVL trees
- AVL insert & Rotation

# Importance of being balanced

![balanced tree](https://swha0105.github.io/assets/intro_algorithm/image/lec_6_balanced_tree.png)   

- Binary Search Tree에서 모든 연산은 시간 복잡도 O(h) (log n < h <  n, 트리의 높이)를 가진다.
- BST에서 연산에 대한 시간 복잡도를 줄이러면 h를 작게 즉, log n 에 가깝게 가져가야한다.  

h (height)의 정의: root 로드에서 마지막 노드까지 가장 긴 경로

<br/>

---

# AVL trees
- Binary Search Tree의 일종.
- BST의 h를 log n으로 만들기 위해 나온 개념.
- 모든 노드에 대해, `BF`의 조건을 만족해야한다.


### BF(Balanced Factor) = $$ h_{l} - h_{r} $$
> BF는 항상 $$\rvert h_{l} - h_{r} \rvert < 1$$ 을 만족한다.  
> $$h_{l}$$: 왼쪽 subtree의 height  
> $$h_{r}$$: 오른쪽 subtree의 height  
> empty node의 BF는 -1

|![BF](https://swha0105.github.io/assets/intro_algorithm/image/lec_6_BF.png)   
|:--:| 
| Balanced Factor in Tree [출처](https://ratsgo.github.io/data%20structure&algorithm/2017/10/27/avltree/) |

<br/>

### Worst case일때 높이와 노드의 상관계수 
모든 노드에 대해 오른쪽 subtree의 높이가 왼쪽 subtree보다 1 높을때 **AVL 트리의 높이와 노드의 상관계수는** 다음과 같다
  
$$ h \approx 1.440 \log N_{h} \lt 2 \log N_{h}  $$

$$N_{h} = 1 + N_{h-1} + N_{h-2}$$

> $$N_{h}$$ = 높이 h의 AVL tree를 구성하는데 최소한의 노드 수  
 
여기서 $$N_{h}$$ 식은 +1 부분만 뺸다면 Fibonacci 수열과 일치한다. ($$N_{h} = F_{h+1} - 1 $$)
> $$F_{h}: \frac{1}{\sqrt{5}}( (\frac{1+\sqrt{5}}{2})^h - (\frac{1-\sqrt{5}}{2})^h ) = int(\frac{1}{\sqrt{5}}(\frac{1+\sqrt{5}}{2})^h )$$

피보나치 일반항을 $$N_{h}$$ 에 대입하여 트리의 높이와 노드의 수의 관계식을 얻으면 다음과 같다.

$$ h \approx 1.440 \log N_{h}$$

다음과 같이 증명하는 방법도 있다.

$$N_{h} = 1 + N_{h-1} + N_{h-2} \; \gt \; 1 + 2 N_{h-2} \; \gt \; 2 N_{h-2} \; \gt \; 4 N_{h-4} ...$$  
$$N_{h} \gt 2^{\frac{h}{2}}  \quad \quad h \approx 2 \log N_{h}$$


<br/> 

---

# AVL insert & Rotation
BST에서 데이터를 insert 할때, 말단 노드에 추가가 된다고 저번 강의에서 설명하였다.  
이럴경우 `모든 노드에 대해 BF < 1` 을 만족하라는 AVL의 특성이 깨질 수 있기에 데이터를 insert한 다음 정렬해주는 과정이 필요하다.  

#### AVL insert 과정.
1. 일반 BST처럼 node를 insert (O (log n))
2. fix AVL property By **Rotation**

<br/>


### Sigle Rotation
앞서 언급한것과 같이 새로운 노드가 말단에 추가될 경우 AVL의 특성이 깨질 수 있다. 이럴 경우 AVL 특성을 만족하기위해 `Rotation`개념을 통해 트리 높이의 밸런스를 맞춘다.

![Left Roation](https://swha0105.github.io/assets/intro_algorithm/image/lec_6_left_rotate.png)   
> k 값은 높이를 나타낸다.  

- 특정 노드의 abs(BF) >= 2 일때 시행.

위의 왼쪽그림에서 left subtree의 높이는 k-1, 오른쪽 subtree의 높이는 k+1이므로 BF = 2에 해당된다. 

이때, **오른쪽 subtree의 높이가 더 높으므로** (무게가 크다), 높이 밸런스를 맞춰주기 위해 **`Left Rotation`**을 시행한다.

**Left Rotation**
- abs(BF) >= 2에 해당되는 노드 (X), 오른쪽 subtree의 root node (Y) 이라 할 때,
- Y를 기준으로 X를 아래로 잡아 당긴다는 개념 
- 이때, subtree B의 모든 노드는 X보다 크기 때문에 X의 오른쪽 subtree가 된다! 

당연히 왼쪽 subtree의 높이가 높을때는 `Right Rotation`을 시행해주면 된다.

<br/>

### Double Rotation

Rotation한번으로는 AVL 성질을 만족 못하는 경우가 있다.  
![Double Roation](https://swha0105.github.io/assets/intro_algorithm/image/lec_6_double_rotate.png)   

위 그림에서, BF가 2인 U를 기준으로 `Right Rotation` 을 수행하면,  
U는 V의 오른쪽 subtree, W는 U의 왼쪽 subtree가 되고, V의 BF는 2가 되기때문에 여전이 AVL의 특성을 만족하지 못한다.  이럴 경우 `Double Rotation`을 수행한다.

- 특정 노드의 abs(BF) >= 2 일때 (노드 X라 가정)
- **노드 X에 대해 zigzag path로 노드가 삽입될 때**
위 두가지 조건을 만족할 경우 Double Rotation을 시행한다.

zigzag path란, BF를 만족하지 못하는 U를 기준으로, **왼쪽 subtree**의 root노드인 V에 **오른쪽 subtree**에 노드가 추가되는 현상이라 정의한다. U를 기준으로 내려가는 화살표가 왼쪽으로 일정하지 않고 오른쪽으로 꺾일경우를 말하는 것인데, 시각적으로 이렇게 이해하는게 편하다. 직접 예제로 그려서 이해해보면, 노드들 끼리의 대소관계가 정리되지 않아 한번의 Rotation으로 높이의 밸런스가 맞지 않는 경우일 때 Double rotation이 필요하다.

**Double Rotation**
- Zigzag path에서 마지막으로 꺾이는 node (위의 그림에서 W)를 기준으로
- Left(Right) Rotation 시행.
- AVL 성질을 만졳할 때 까지, W기준으로 rotation 시행.

~~예제로 그려보면 정말신기하다~~

**Double rotation은 Single Rotation을 2번하는 개념이 아닌, 노드가 추가되는 트리의 구조에 따라 다른개념**




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
