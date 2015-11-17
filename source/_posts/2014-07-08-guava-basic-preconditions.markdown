---
layout: post
title: 【Guava】使用前置条件进行合法性检查
date: 2014-07-08 22:55:59 +0800
comments: true
tags: guava
categories: Backend
---

> 本系列文章是在学习[Google Guava 17.0](https://code.google.com/p/guava-libraries/)库的过程中的整理和笔记，不是完全翻译，欢迎交流。

## 1. 前置条件检查

- Preconditions类提供了很多static方法，用于对方法或构造函数的参数进行合法性检查，强烈推荐使用；

- 关于性能：Preconditions是为了提高代码的可读性，但有些情况下，可能会影响系统性能；在性能敏感的环境下，总是可以使用如下形式：

	if (value < 0.0) {
		throw new IllegalArgumentException("negative value: " + value);
	}

- 在使用com.google.common时，应尽量避免使用Objects.requireNonNull(Object)，推荐使用checkNotNull(Object)或者Verify.verifyNotNull(Object)，而且它们支持%s的格式化信息；

- checkNotNull(Object) 含义明确，不易混淆，而且参数合法时返回原值，可以用于‘检查并赋值‘的场合，如：this.field = checkNotNull(field)；

- 推荐将多个前置条件分开检查，并在异常消息中提供有用的信息，这样有利于程序的调试；

## 2. Preconditions API

每一类方法都有三种变体，除了要检查的参数外，额外的参数组合有：

- 不带参数，抛出异常时不带异常信息；

- 带一个Object参数，抛出异常时，异常信息为：Ojbect.toString()；

- 带一个String参数，和不定个数的Object参数；行为类似于printf，但String中仅支持%s格式符，如：

	checkArgument(i >= 0, "Argument was %s but expected nonnegative", i);
	checkArgument(i < j, "Expected i < j, but %s > %s", i, j);

主要的static方法有：

- `checkArgument(boolean)`		检查参数条件是否为true；

- `checkNotNull(T)`				检查参数是否不为null，如果不为null，直接返回参数值；

- `checkState(boolean)`			检查对象的状态，不依赖于方法的参数；

- `checkElementIndex(int index, int size)`	检查index是否在[0, size)之间，其中size可以表示String、List或Array的长度；

- `checkPositionIndex(int index, int size)`	检查index是否在[0, size]之间，其中size可以表示String、List或Array的长度；

- `checkPositionIndexes(int start, int end, int size)`	检查区间[start, end)是否为[0, size]的子区间；(该方法仅此一种形式，无其它参数版)。

### 参考

- [PreconditionsExplained](https://code.google.com/p/guava-libraries/wiki/PreconditionsExplained)
- [Preconditions API](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html)
