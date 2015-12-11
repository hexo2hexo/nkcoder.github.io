---
layout: post
title: 【Guava】避免使用null
date: 2014-07-08 22:36:58 +0800
comments: true
tags: guava
categories: Backend
---

> 本系列文章是在学习[Google Guava 17.0](https://code.google.com/p/guava-libraries/)库的过程中的整理和笔记，不是完全翻译，欢迎交流。

## 1. 避免使用null

- 集合中不要接受null，对于null采取快速失败的策略；

- NULL的含义是模糊的，如Map.get(key)，如果返回null，可能是该map中不存在该key，也可能是存在该key，但对应的value为null；

- 如果打算在map中存放key对应的value为null的entry，不要这么做，宁愿另外用一个Set保存value非null的key（或value为null的key），分开操作，在你的上下文中，value为null的key应该具有明确的含义；

<!-- more -->

## 2. Optional<T>

- Optional是一个引用，如果其中有值，则为'present'，如果没有值，则为'absent'，但不会“包含null”；Optional可以看做是对null的友好的包装，在设置或取值时，会提醒你判断它的引用状态，而null很容易让人忘记检查其合法性；

## 3. Optional API

构造Optional的静态方法：

- `Optional.of(T)`		根据非null的参数，构造一个Optional实例；

- `Optional.absent()`		构造一个不包含引用的(absent)的Optional实例；

- `Optional.fromNullable(T)`	从可为null的参数中构造一个Optional实例，如果参数不为null，则状态为present，否则状态为absent；

Optional对象的方法：

- `boolean isPresent()`	如果该optional实例有非null引用，返回true，否则返回false；

- `T get()`				如果该optional实例有非null引用，返回引用对象的值，否则，抛出IllegalStateException；

- `T or(T)`				如果该optional实例有非null引用，返回引用对象的值，否则，返回参数指定的默认值；

- `T orNull()`			如果该optional实例有非null引用，返回引用对象的值，否则，返回null；

- `Set<T> asSet()`		如果该optional实例包含非null引用，以该实例的值构造一个Set并返回，否则返回一个空Set；

## 4. Strings中相关方法

注意，不要将null串与空串混为一谈，Strings工具类中相关的static方法有：

- `emptyToNull(String)`		如果参数不为空，返回参数值，否则返回null；

- `nullToEmpty(String)`		如果参数不为null，返回该参数，否则返回空串("")；

- `isNullOrEmpty(String)`	如果参数为null或为空，返回true；

### 参考

- [Using and avoiding null](https://code.google.com/p/guava-libraries/wiki/UsingAndAvoidingNullExplained)
- [Optional doc](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html)
