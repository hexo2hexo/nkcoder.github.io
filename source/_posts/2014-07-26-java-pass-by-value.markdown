---
layout: post
title: Java的值传递
keywords: java pass-by-value
date: 2014-07-26 23:53:27 +0800
comments: true
tags: java
categories: Backend
---

`java pass by value or reference`，我用这句话向google求助，第一条结果就是我想要的，StackOverflow上早有大神解决了我等菜鸟的困惑。[排名第一的回答](http://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value)虽简短，但精辟，两个小例子非常准确地说明了问题的本质。我想，如果再配两幅小图，那就完美了，故作图如下。

简单地回答，Java中只有值传递，没有引用传递之说；对象以引用的形式传递，但引用本身是值传递的。当将一个对象传递给函数时，该对象的内存地址将按位复制给临时变量，即函数内的临时变量和函数外的变量中都是对象的地址，都指向对象在内存中的位置，对这两个引用本身的改变是相互独立的，但通过引用去修改引用的对象则是相互影响的，因为修改的是同一个对象。

StackOverflow上作者的例子太经典了，这里还是借用一下，这里仅配图：

	Dog aDog = new Dog("Max");
	foo(aDog);
	aDog.getName().equals("Max"); // true

	public void foo(Dog d) {
		d.getName().equals("Max"); // true
		d = new Dog("Fifi");
		d.getName().equals("Fifi"); // true
	}

在刚调用函数时：

![value-ref-eg1-before-call](/images/post/value-ref-eg1-before-call.png)

执行`d = new Dog("Fifi");`后：

![value-ref-eg1-after-call](/images/post/value-ref-eg1-after-call.png)

	Dog aDog = new Dog("Max");
	foo(aDog);
	aDog.getName().equals("Fifi"); // true

	public void foo(Dog d) {
		d.getName().equals("Max"); // true
		d.setName("Fifi");
	}

在刚函数调用时：

![value-ref-eg2-before-call](/images/post/value-ref-eg2-before-call.png)

调用`d.getName("Fifi")`后：

![value-ref-eg2-after-call](/images/post/value-ref-eg2-after-call.png)


### 参考：

- [Is Java “pass-by-reference” or “pass-by-value”?](http://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value)
