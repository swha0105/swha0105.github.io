---
layout: post
title:  "[Intro CS] Object Oriented Programming & Inheritance"
subtitle:   "Introduction to Computer Science and Programming"
categories: programming
tags: cs
comments: False
---
# 8 & 9. Object Oriented Programming & Inheritance
- Object & Object Oriented Programming (OOP)
- Class
- Inheritance  

<br/>

***

# Object & Object Oriented Programming

- **Object Oriented Programming (OOP):**

    **Object들을 데이터와 그것들을 처리 할 수 있는 Method의 집합으로 보고 구성하는 프로그래밍 방법**

- **Object**

    **Everything in Python is an object  (파이썬의 모든것들은 Object이다)**

    Object는 다음의 것들을 가지고 있다.

    1. Type
    2. Internal representation 의 존재
    3. Interface의 존재  (functions)

    그리고 모든 Python의 object들은 Abstract data type을 하고 있다. 

    **Abstract data type**: Object와 그들을 다룰 수 있는 연산들의 집합체

    a = [1,2] 를 선언했다고 치자. 이것이 어떻게 object이고 위의 두 성질을 만족하는지 알아보자.

     a 는 list의 Type을 가지고 안에 존재하는 값들은 linked 되어있다. a[0] = 1, a[1] = 2 와 같이 리스트 안에서 인덱스 순서대로 데이터들이 구성되어있음을 확인할 수 있다. 이와 같이 클래스 안에서 데이터들을 표현하는 방법을 **Internal representation**이라 한다. 
    (이게 너무 당연하다고 느껴지면 Dictionary 특징인 `Unordered` 를 떠올려라) 

    또한 list는 .append, .extend 와 같은 함수들을 사용 할 수 있는데 이것들은 리스트 클래스에 선언되어있는 Method들이다 (아래에 설명). 이 method들은 해당 클래스 내에서 생성된 객체에만 사용해야 함으로 메인 프로그램과 격리되어 사용되어야 한다. 메인 프로그램에서는 method들을 사용할수 있지만 소스코드를 볼 수 없다. 이러한 가상의 벽을 `Interface` 라고 하고 이를 통해 함수의 기본 성질인 `abstraction`을 구현 할 수 있다.

 
 <br/>
 
 ***

# Class

앞서 설명한 Object들 중 같은 속성인 것들을 묶어서 처리하면 편하지 않을까?

나는 개인적으로 약간 정리벽(?)이 있어 기능과 용도에 따라 분류하고 정리하는걸 습관적으로 하고 안되어있음 약간 미치는 편이다.  코드도 마찬가지인데 코드내에서 Object들을 묶어서 처리할때 Class 개념이 필요하다.

- **Class는 object들을 만들어 낼 수 있는 함수와 변수의 집합체**

Class를 통해 만들어진 객체들은 Class의 `Instance` 라고 하며 클래스를 이용해 객체를 만드는 과정을 `instantiation` 라고 한다.

- **Class는 같은 Attribute를 가지는 객체들을 만들어 낸다.**
    1. **Data attribute**: Object를 결정하는 속성
    2. **Procedural attribute:** Object를 다루는 함수 (Method)

예를 들어, 유기묘 보호소에서 유기묘들의 정보를 정리하는 일을 했다고 가정하자. 유기묘라는 Class안에 냥이들은 제각각의 특성을 가지고 있지만 분류하기 쉽게 나이, 털 색깔, 품종으로 분류한다고 가정한다. 이때 나이, 털 색깔, 품종은 유기묘들의 **`data attribute`**가 된다. (냥이들의 정보를 결정하는 속성) 그리고 유기묘들이 좋은 집사를 만나 입양을 갈때 우리가 가진 유기묘들의 정보에서 제외시켜줘야 한다. 그때 정보를 삭제하는 함수를 class안에 선언하면 그것들은 `**Method**`라 할 수 있다. (냥이들의 정보를 처리하는 함수)

파이썬의 내장된 type말고 다른 형태의 변수가 필요하다면 class를 사용해서 정의를 해야한다.

예제를 통해 개념을 설명하는게 가장 빠를듯 하다. 다음과 같이 a,b의 거리를 구하는 코드를 보자. 

```python
import numpy as np

a = [1,1]
b = [4,4]

class Coordinate():
	def __init__(self,coords):
		self.x = coords[0]  #data attribute
		self.y = coords[1]

	def __str__(self):
		return 'x = ' + str(self.x) + ' y = ' + str(self.y)

	def get_distance(self,other_point):
		return np.sqrt( (self.x - other_point.x)**2 + (self.y - other_point.y)**2)

coords_a = Coordinate(a) #coords_a 는 Coordinate의 Instance!
coords_b = Coordinate(b)

coords_a.get_distance(coords_b)
Coordinate.get_distance(coords_a,coords_b)
```

Class Coordinate():  넘어가도록 하겠다.. 

**def __init__ (self, coords):**

>- __function__ 과 같은 형태는 파이썬에 내장되어있는 특수한 함수를 불러온다.
>- **__init__** 은 `class`가 선언되어 `Instance` 객체를 만들때 자동으로 불러오는 특수한 함수
>- **self는 선언되는 Instance 객체를 Formal parameter로 받는 특수한 Actual parameter**

coords_a = Coordinate(a) 를 선언하면 자동으로 __init__ 이 실행 된다. 이때 coords_a는 self 인자로 들어가고, a는 coords로 들어가게 된다.  

**def __str__ (self):**

> - 특수한 함수로 print() 문을 실행시켰을때 return되는 값을 정의한다.

예를들어, __str__ 함수가 없다고 가정하고 coords_a = Coordinate(a) 한 뒤,  **print(coords_a)**를 실행 시키면 해당 Instance 객체의 메모리 주소값을 리턴한다. 메모리주소 대신 유용한 값을 리턴받고 싶을때  __str__ 에서 return할 값을 정의하면 **해당 Instance 객체의** **리턴값이** 된다. 따라서 디버깅할때 유용한 값들을 리턴 시키도록 하자. 

**def get_distance(self, other_point):**

> - Coordinate의 **Method**중 하나이며 **Coordinate로 생성된 Instance 객체**를 연산하는 정보를 담고 있다.

**coords_a.get_distance(coords_b)**

> - coords_a는 coordinate **class**의 **instance** 객체이고, 해당 class의 **method**들도 함께 담고 있다.
> - get_distance의 self인자는 coords_a가 되고, other_point는 coords_b가 된다.

**Coordinate.get_distance(coords_a,coords_b)**

> - **Instance 객체**에 포함된 **method**를 통한 연산이 아닌, **Class**에서 직접 **Method**들을 불러와 연산 함
> - instance 객체를 받을 수 없으므로 get_distance의 self인자는 Formal parameter 중 첫번째와 bound 됨 (이 경우 coords_a가 self 인자로 bound됨)

<br/>

***

# Inheritance

클래스의 목적이 객체들을 분류하는거라고 생각한다면 객체가 하나의 분류기준으로 분류가 안될때는 어떻게 할까? 혹은 객체들을 좀 더 세밀하게 분류 해야된다고 하면 어떻게 해야할까?

- **Inheritance**: Hierarchy에 상위에 존재하는 Class의 Data attribute, Method 등이 아래에 존재하는 class들에게 상속되는 현상
- **Hierarchy:**  Class의 계층 구조
    - Super class (Parent class): (상대적) 상위 class
    - Sub class (Child class)     : (상대적) 하위 class
        1. Sub class는 Super class의 모든 속성을 물려받는다
        2. Sub class는 Super class에서 물려받은 속성외 다른 추가 속성을 추가, 삭제가 가능하다
        3. Sub class는 Super class에서 물려받은 속성에 대해 수정이 가능하다  (Overriding)

        **Overriding:** Super class와 Sub class의 Method, 즉 함수 이름이 같으면 Sub class의 함수로 실행된다. 이와 같은 개념을 Super class의 함수를 덮어썼다고 하여 Overriding이라 한다

역시 예제가 가장 빠르다.

수업내용에서 나온 Animal이라는 예제를 간추려 보았다.

```python
class Animal(object):
    def __init__(self, age):
        self.age = age
        self.name = None
    def get_age(self):
        return self.age
    def get_name(self):
        return self.name
    def set_age(self, newage):
        self.age = newage
    def set_name(self, newname=""):
        self.name = newname
    def __str__(self):
        return "animal:"+str(self.name)+":"+str(self.age)

class Person(Animal):
    def __init__(self, name, age):
        Animal.__init__(self, age)
        self.set_name(name)
        self.friends = []
    def get_friends(self):
        return self.friends
    def speak(self):
        print("hello")
    def add_friend(self, fname):
        if fname not in self.friends:
            self.friends.append(fname)
    def __str__(self):
        return "person:"+str(self.name)+":"+str(self.age)

class Student(Person):
    def __init__(self, name, age, major=None):
        Person.__init__(self, name, age)
        self.major = major
    def __str__(self):
        return "student:"+str(self.name)+":"+str(self.age)+":"+str(self.major)
    def change_major(self, major):
        self.major = major
    def speak(self):
        r = random.random()
        if r < 0.25:
            print("i have homework")
        elif 0.25 <= r < 0.5:
            print("i need sleep")
        elif 0.5 <= r < 0.75:
            print("i should eat")
        else:
            print("i am watching tv")

class Rabbit(Animal):
    # a class variable, tag, shared across all instances
    tag = 1
    def __init__(self, age, parent1=None, parent2=None):
        Animal.__init__(self, age)
        self.parent1 = parent1
        self.parent2 = parent2
        self.rid = Rabbit.tag
        Rabbit.tag += 1
    def get_rid(self):
        # zfill used to add leading zeroes 001 instead of 1
        return str(self.rid).zfill(3)
    def get_parent1(self):
        return self.parent1
    def get_parent2(self):
        return self.parent2
    def __add__(self, other):
        # returning object of same type as this class
        return Rabbit(0, self, other)
    def __eq__(self, other):
        # compare the ids of self and other's parents
        # don't care about the order of the parents
        # the backslash tells python I want to break up my line
        parents_same = self.parent1.rid == other.parent1.rid \
                       and self.parent2.rid == other.parent2.rid
        parents_opposite = self.parent2.rid == other.parent1.rid \
                           and self.parent1.rid == other.parent2.rid
        return parents_same or parents_opposite
    def __str__(self):
        return "rabbit:"+ self.get_rid()
```

구조를 잘 보게 되면 Animal → (Person, Rabbit) 이 되고 Person → Student가 된다.

먼저 Person 클래스부터 살펴보자

**class Person(Animal):**
> - Actual Parameter로 지정되어있는 Animal은 Animal 클래스에 속하는 모든 속성을 가져 온다.

**def __init__(self, name, age):**
> -  Super class인 Animal과 함수 이름이 겹치기 때문에 Animal의 __init__ 함수가 작동하지 않고 Person class의 __init__ 만 작동한다.  (**Overriding**)
> -  Super 클래스인 Animal의 **Instance variable** 들의 속성을 물려받으러면 Animal.init 과 같이 따로 실행해야한다.
> - self.set_name은 Person class의 Super class인 Animal에서 상속받은 함수를 사용한다


>> **Instance variable**: Instantiation 과정을 거쳐 생기는 변수로서 해당 Class에만 존재한다 (상속 되지않음)  
>> **Class variable**: Class 내에 존재하는 변수로서 상속되는 해당 클래스로 만들어진 모든 Instance (sub class포함) 에서 해당 값을 참조 & 수정이 가능하다 
"""

**def speak(self): ,  def add_friend(self, fname): , def age_diff(self, other):**

> - 이와 같은 함수들은 Super class인 Animal과 관계없이 Person 클래스 내 작동하는 함수 Method들이다.  
> - 이를 물려받을 Sub class인 Student의 class에서는 이를 참조 가능하지만 Super class인 Animal은 참조가 불가능하다.  



그 다음 Rabbit 클래스를 보자. 

**tag = 1**

> - **Class 변수**로써 Rabbit 클래스를 이용해 만든 모든 Instance 들이 참조 가능

**def __init__(self, age, parent1=None, parent2=None):**

> - formal parameter인 parent1,2 의 Default 값은 None 이다. (따로 지정안하면 None)
> - Super class인 Animal의 **Instance Variable**들을 상속 받는다
> - Class Variable인 Rabbit.tag를 가져와 해당 Instance에 적용한다.

**def __add__(self, other):**
> - return Rabbit(0, self, other)
> - 특수함수로써 객체를 `+` 연산 했을시 수행해하는 연산을 포함하고 있다.

~~너무 힘들다~~

~~Dr. Ana Bell 진짜 설명잘한다..~~


<br/>

*** 

## Reference

[Lecture pdf 1](https://github.com/swha0105/swha0105.github.io/blob/gh-pages/assets/intro_cs/material/Lec8.pdf)   
[Lecture pdf 2](https://github.com/swha0105/swha0105.github.io/blob/gh-pages/assets/intro_cs/material/Lec9.pdf) 