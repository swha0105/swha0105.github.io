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
- Other balanced trees
- Data structure in general
- Lower bounds 

# Importance of being balanced

![balanced tree](https://swha0105.github.io/assets/intro_algorithm/image/lec_6_balanced_tree.png)   

- Binary Search Tree에서 모든 연산은 시간 복잡도 O(h) (log n < h <  n, 트리의 높이)를 가진다.
- BST에서 연산에 대한 시간 복잡도를 줄이러면 h를 작게 즉, log n 에 가깝게 가져가야한다.  

h (height)의 정의: root 로드에서 마지막 노드까지 가장 긴 경로

<br/>

---

# AVL trees
- Binary Search Tree의 일종.
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

### insert

1. insert as in simple BST (O (log n))
2. fix AVL property

height를 저장해야됨.

<br />
 
# 강의 듣는중..

### rotation



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
