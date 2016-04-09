title: 'python tips 2: string模块与reversed关键字'
categories: Backend
date: 2016-01-12 19:56:14
tags: python
---

## 1. 使用string模块

如果需要构造这样一个dict：

    'A' -> 1
    'B' -> 2
    'C' -> 3
    ...
    'Z' -> 26

<!-- more -->

你当然不想写26遍赋值语句，我们知道，内置函数`ord()`和`chr()`可以在unicode字符和值之间进行转换，而对于ASCII字符，unicode编码和ASCII编码的值相同，所以，可以这样来实现：

    ascii_of_A = ord('A')

    ascii_dict = {}
    for index in range(26):
        c = chr(ascii_of_A + index)
        ascii_dict[c] = index + 1

其实，`string`模块有可以直接拿来用的函数：

    string.ascii_letters
    string.ascii_uppercase
    string.ascii_lowercase

所以，可以这样实现：

    ascii_dict = {}
    for index, c in enumerate(string.ascii_uppercase):
        ascii_dict[c] = index + 1

## 2. 使用reversed反转列表

现在，有个list或一个string，我们想逆序遍历，怎么实现呢？

最直觉简单的做法就是先拿到列表的长度，然后使用下标索引逆序遍历，像这样：

    num_list = [1, 2, 3, 4, 5, 6, 7, 8, 9]
    i = len(num_list) - 1
    while i >= 0:
        print(num_list[i])
        i -= 1

python的内置函数`reversed()`就是用来反转一个列表的（list和string都可以）：

    for x in reversed(num_list):
        print(x)

如果在逆序遍历的同时，还希望得到对应的索引呢，使用`enumerate()`:

    for i, x in reversed(list(enumerate(num_list))):
        print(i, x)

其实，除了使用`reversed()`函数，也可以使用list和string的**slicing**机制，这里简单介绍一下：

    a_list[start:end:step]

- 默认是顺序遍历，且挨个元素遍历，因为step的值默认是1；
- 遍历的范围是list[start, end)，包含前一个索引，不包含后一个索引；
- 当start省略的时候，值为0，当end省略的时候，值为list的长度，所以a_list[::]表示整个列表；
- start和end可以为负数，比如-1表示倒数第一个元素，可以认为此时的索引值为(list长度-1)；
- step大于0时，表示顺序遍历，step小于0时，表示逆序遍历，所以[::-1]表示逆序遍历列表；

明白了**slicing**的机制，上面的问题就很容易这样来实现了：

    for x in num_list[::-1]:
        print(x)

## 参考

- [Built-in Functions¶](https://docs.python.org/3/library/functions.html)
- [Traverse a list in reverse order in Python](http://stackoverflow.com/questions/529424/traverse-a-list-in-reverse-order-in-python)
