title: python学习笔记之Native Datatypes
categories: Backend
date: 2015-12-25 23:43:17
tags: python
---

> 本文来自[Dive Into Python 3-Native Datatypes](http://www.diveintopython3.net/native-datatypes.html)的学习笔记，记录重点，用于加深印象。

    - 所有变量都是有数据类型，但是不需要声明，python会在第一次赋值时确定变量的类型；

## 1.1 Boolean

  	- True/False

<!-- more -->

## 1.2 Number

    - `isinstance(1, int)`方法可以用来确定变量的类型；
    - 整数和浮点数可以通过`int(float1), float(int1)`进行转换；
    - 运算符：`/, //, %, **`;
    - 分数：`fractions.Fraction(1, 3)`;
    - 三角函数：`math.pi; math.sin(2)`;
    - 条件判断：0(包括0.0, Fraction(0, n))-> False；非0 -> True；
    - 注意：python没有运算符++/--，可以通过num += 1实现自增；

## 1.3 List

    - `list1 = []; list2 = [2, 'a']`;
    - 子list：`list2[0:3]; list2[:-1]; list2[1:]; list2[:]`;
    - 增加元素：+操作符；`append('a'); extend([2, 3, 4]); insert(0, 'a')`;
    - 查找元素：in操作；`index('b'); len(list1)`;
    - 删除元素：`del list1[1]; list1.remove('a'); list1.pop()`; `remove()`方法仅会删除第一个元素；
    - 判断list是否为空：`if not list1:`
    - 条件判断：空list（[]） -> False；非空list -> True;
    - 将list作为queue使用，可以通过`append(x); pop(0)`实现;
      但是当数据量较大时，通过`collections.deque()`效率会更高一些：`append(x); popleft()`

## 1.4 Tuple

    - tuple1(); tuple2(3,);(只有一个元素时必须加逗号)；tuple3('a', 'b', 3);
    - 可以看成是不可变的list；tuple比list更快；
    - list和tuple可以通过tuple(list1)和list(tuple1)相互转换；
    - 查找：in操作；len(set1);
    - 条件判断：空tuple(()) -> False; 非空tuple -> True;

## 1.5 Set

    - `set1 = {1, 2}`;
    - 创建空set：`set1 = set()`; 而`set1 = {}`创建的是一个空dict；
    - set和list可以通过`set(list1), list(set1)`相互转换；
    - 修改：`add()/update()/discard()/remove()/clear()`;
    - 集合操作: `union()/intersection()/difference()/symmetric_difference()/issubset()/issuperset()`;
    - 查找：in操作；`len(set1)`;
    - 条件判断：空set（set()) -> False, 非空set -> True

## 1.6 dictionary

    - `dict1 = {}; dict2 = {'key1': 'value1'}`
    - dict支持in操作，判断key是否存在；
    - dict的key和value都可以是任意类型；
    - 方法：len();
    - 空dict({}) -> False, 非空dict -> True

## 1.7 None

    - None是python的null，仅与None比较时为True，与其它任意值比较都为False；
    - 在条件判断时，None -> False; not None -> True;
