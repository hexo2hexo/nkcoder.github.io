---
layout: post
title: 阻塞队列之ArrayBlockingQueue
date: 2014-09-05 23:06:18 +0800
comments: true
tags: java
categories: Backend
---

ArrayBlockingQueue是一个有界阻塞队列，队列根据FIFO的顺序组织元素，从队列的头部取元素，新的元素被插入到队列的尾部。插入元素时，如果队列满，线程阻塞(也有不阻塞的方法)，取元素时，如果队列为空，线程阻塞(有不阻塞的方法)。

<!-- more -->

内部使用固定大小的数组实现，在构造ArrayBlockingQueue对象时，指定数组的容量compacity，之后该容量不能被修改。

支持公平策略fairness，根据FIFO的顺序选择等待的线程；该参数在构造对象的时候开启，默认禁用；启用fairness会降低吞吐量，但是可以避免线程饥饿。

同步：内部使用ReentrantLock和two-condition算法实现

	/** Main lock guarding all access */
    final ReentrantLock lock;
    /** Condition for waiting takes */
    private final Condition notEmpty;
    /** Condition for waiting puts */
    private final Condition notFull;

> notEmpty表示队列不为空，可以插入元素；notFull表示队列没有满，可以取出元素。

构造函数：

	ArrayBlockingQueue(int capacity)
	ArrayBlockingQueue(int capacity, boolean fair)

主要方法及区别：

	boolean add(E e)
	boolean offer(E e)
	boolean offer(E e, long timeout, TimeUnit unit)
	void put(E e)

> add()插入元素，如果队列不满，则立即返回true，否则抛出IllegalStateException；offer()也是插入元素，在队列满的时候，返回false，也可以指定超时，超时后，队列还是满的，则返回false；put()插入元素，队列满时等待，直到队列不满或者被中断。

	E take()
	E poll(long timeout, TimeUnit unit)
	E poll()

> take()表示从队列取元素，如果队列为空，则等待，直到队列不空或者被中断；poll()取出元素，如果队列为空，返回null，可以指定超时，如果超时后队列还是为空，则返回null。

注意：合理地处理这些方法抛出的异常，否则线程就挂了，线程池也就崩溃了。比如add(),take(), put()等都可能会被中断，从而抛出中断异常，poll()可能返回null，如果将返回结果赋值给基本类型的变量，会抛出空指针异常。对线程中可能出现的异常，一般有两种处理方式：1)捕获所有的异常；2)给线程注册一个UncaughtExceptionHandler；

参考：

+ [ArrayBlockingQueue](http://fuseyism.com/classpath/doc/java/util/concurrent/ArrayBlockingQueue-source.html)
