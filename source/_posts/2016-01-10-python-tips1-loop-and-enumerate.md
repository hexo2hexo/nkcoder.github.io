title: 'python tips 1: 循环与enumerate'
categories: Backend
date: 2016-01-10 22:07:42
tags: python
---

开发的时候，经常会有这样的需求：遍历一个集合（比如数组），索引不是从0开始，而是从某一个具体的值开始，在Java或Swift中，可以直接用for循环实现：

Java代码：

    for (int i = 3; i < dataList.size(); i++) {
        // do someting with dataList[i]
    }
<!-- more -->

Swift代码：

    for var i = 3; i < dataList.count; i++ {
        // do someting with dataList[i]
    }

    for i in 3 ..< dataList.count {
        // do someting with dataList[i]
    }

但是python的for循环没有类似的形式，当然可以用while循环实现，但总感觉不够简洁：

    i = 3
    while i < len(data_list):
        // do someting with data_list[i]
        i += 1

借助`range()`函数，使用`for...in...`循环也是可以实现的：

    for i in range(3, len(data_list)):
        // do someting with data_list[i]

另外一种实现，可以使用`enumerate()`方法，起始索引通过参数设置：

    i = 3
    for (i, value) in enumerate(data_list[i:], start=i):
        // do someting with value

这里需要说明一下的是，`enumerate()`的第一个参数表示的数组，必须和第二个参数表示的索引是对应的，如果是`enumerate(data_list, start=i)`，那么i的值从3开始，而value的值还是从data_list[0]开始的，而不是从data_list[3]开始的。

