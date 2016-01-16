title: Java 8 Lambda表达式入门
categories: Backend
date: 2016-01-16 18:03:04
tags: java
---

> 本文是在学习[Java SE 8 for the Really Impatient](http://www.amazon.cn/Java-SE8-for-the-Really-Impatient-A-Short-Course-on-the-Basics-Horstmann-Cay-S/dp/0321927761/ref=sr_1_2)过程中整理而来的，顺便推荐一下这本书！

## 1.1 为什么使用Lambda表达式

先看几个例子：

第一个例子，在一个独立的线程中执行某项任务，我们通常这么实现：

	class Worker implements Runnable {
		public void run() {
			for (int i = 0; i < 100; i++)
				doWork();
		}
		...
	}

	Worker w = new Worker();
	new Thread(w).start();

<!-- more -->

第二个例子，自定义字符串比较的方法（通过字符串长度），一般这么做：

	class LengthComparator implements Comparator<String> {
		public int compare(String first, String second) {
			return Integer.compare(first.length(), second.length());
		}
	}
	Arrays.sort(strings, new LengthComparator());

第三个例子，在JavaFX中，给一个button添加一个callback：

	button.setOnAction(new EventHandler<ActionEvent>() {
		public void handle(ActionEvent event) {
			System.out.println("Thanks for clicking!");
		}
	});

这些例子有一个共同点，就是：先定义一段代码块，传给某个对象或方法，然后被执行。在Lambda表达式之前，Java是不允许直接传递代码块的，因为Java是面向对象的，因此必须传递一个对象，将要执行的代码块封装到对象里。

## 1.2 Lambda表达式的语法

将上面第二个例子中的`LengthComparator`，用Lambda表达式表示为：

	(String first, String second) -> Integer.compare(first.length(), second.length());

`->`前为参数列表，其后为表达式语句体；

如果表达式语句体不止一行，则将语句体写在{}中，与普通的函数一样：

	(String first, String second) -> {
        if (first.length() > second.length()) {
            return 1;
        } else if (first.length() == second.length()) {
            return 0;
        } else {
            return -1;
        }
    };

如果没有参数，`()`还是需要带上，比如上面的第一个例子，可以表示为：

	() -> {
        for (int i = 0; i < 1000; i ++) {
            doWork();
        }
    }

如果参数的类型可以从上下文自动推断，则可以省略：

	Comparator<String> comp
		= (first, second) // Same as (String first, String second)
		-> Integer.compare(first.length(), second.length());

如果参数只有一个，且类型可以自动推断，则小括号`()`也可以省略：

	// Instead of (event) -> or (ActionEvent event) ->
	eventHandler<ActionEvent> listener = event -> System.out.println("Thanks for clicking!");

lambda表达式的返回值的类型是自动推断的，因此不需要指明；在lambda表达式，某些条件分支中有返回值，而其它分支没有返回值，是不允许的，如：

	(x) -> {
		if (x >= 0) {
			return 1;
		}
	}

另外，`expression lambda`和`statement lambda`的区别是，`expression lambda`不需要写`return`关键字，Java runtime会将表达式的结果作为返回值返回，而`statement lambda`是写在`{}`中的表达式，需要使用`return`关键字，比如：

	// expression lambda
	Comparator<String> comp1 = (first, second) -> Integer.compare(first.length(), second.length());

	// statement lambda
	Comparator<String> comp2 = (first, second) -> { return Integer.compare(first.length(), second.length());};

## 1.3 Functional Interface

如果一个接口（interface）仅有一个抽象方法（abstract method），就称为`Functional Interface`，比如`Runnable`、`Comparator`等。
在任何需要一个`Functional Interface`对象的地方，都可以使用lambda表达式：

	Arrays.sort(words, (first, second) -> Integer.compare(first.length(), second.length()));

这里，`sort()`的第二个参数需要的是一个`Comparator`对象，而`Comparator`是`Functional Interface`，因此可以直接传入lambda表达式，在调用该对象的`compare()`方法时，就是执行该lambda表达式中的语句体；

如果lambda表达式的语句体会抛出异常，则对应的`Functional Interface`中的抽象方法必须抛出了该异常，否则就需要在lambda表达式中显式捕获异常：

	Runnable r = () -> {
	   System.out.println("------");
	   try {
		   Thread.sleep(10);
	   } catch (InterruptedException e) {
		   // catch exception
	   }
   };

   Callable<String> c = () -> {
	   System.out.println("--------");
	   Thread.sleep(10);
	   return "";
   };

## 1.4 Method Reference

如果将lambda表达式的参数作为参数传递给一个方法，他们的执行效果是相同的，则该lambda表达式可以使用`Method Reference`表达，以下两种方式是等价的：

	(x) -> System.out.println(x)
	System.out::println

其中`System.out::println`被称为`Method Reference`。

`Method Reference`主要有三种形式：

	object::instanceMethod
	Class::staticMethod
	Class::instanceMethod

对于前两种方式，对应的lambda表达式的参数和method的参数是一致的，比如：

	System.out::println
	(x) -> System.out.println(x)

	Math::pow  
	(x, y) -> Math.pow(x, y)

对于第三种方式，对应的lambda表达式的语句体中，第一个参数作为对象，调用method，将其它参数作为method的参数，比如：

	String::compareToIgnoreCase
	(s1, s2) -> s1.compareToIgnoreCase(s2)

## 1.5 Constructor Reference

`Constructor Reference`与`Method Reference`类似，只不过是特殊的method：`new`，具体调用的是哪个构造函数，由上下文环境决定，比如：

	List<String> labels = ...;
	Stream<Button> stream = labels.stream().map(Button::new);

`Button::new`等价于`(x) -> Button(x)`，所以调用的构造函数是：`Button(x)`;

除了创建单个对象，也可以创建对象数组，如下面两种方式等价：

	int[]::new  
	(x) -> new int[x]

## 1.6 变量作用域

lambd表达式会捕获当前作用域下可用的变量，比如：

	public void repeatMessage(String text, int count) {
		Runnable r = () -> {
			for (int i = 0; i < count; i ++) {
				System.out.println(text);
				Thread.yield();
			}
		};
		new Thread(r).start();
	}

但是这些变量必须是不可变的，为什么呢？看下面这个例子：

	int matches = 0;
	for (Path p : files)
		new Thread(() -> { if (p has some property) matches++; }).start(); // Illegal to mutate matches

因为可变的变量在lambda表达式中不是线程安全的，这和内部类的要求是一致的，内部类中只能引用外部定义的`final`变量；

lambda表达式的作用域与嵌套代码块的作用域是一样的，所以在lambd表达式中的参数名或变量名不能与局部变量冲突，如：

	Path first = Paths.get("/usr/bin");
	Comparator<String> comp = (first, second) -> Integer.compare(first.length(), second.length()); // Error: Variable first already defined

如果在lambda表达式中引用`this`变量，则引用的是创建该lambda表达式的方法的`this`变量，如：

	public class Application() {
		public void doWork() {
			Runnable runner = () -> {
				...;
				System.out.println(this.toString());
				...
			};
		}
	}

所以这里的`this.toString()`调用的是`Application`对象的`toString()`，而不是`Runnable`对象的。

## 1.7 Default Method

接口中只能有抽象方法，如果在已有的接口中新增一个方法，则该接口所有的实现类都需要实现该方法。Java 8中引入了`Default Method`的概念，在接口中新增一个`default`方法，不会破坏已有的接口规则，接口的实现类可以选择重写或直接继承该`default`方法，比如：

	interface Person {
		long getId();
		default String getName() { return "John Q. Public"; }
	}

Java是允许多继承的，如果一个类的父类中定义的方法和接口中定义的`default`方法完全相同，或者一个类的两个接口中定义了完全相同的方法， 则如何处理这种冲突呢？处理规则如下：

- 如果是父类和接口的方法冲突：以父类中的方法为准，接口中的方法被忽略；
- 如果两个接口中的`default`方法冲突，则需要重写该方法解决冲突；

## 1.8 Static Method

Java 8之前，接口中只能定义`static`变量，Java 8开始，接口中可以添加`static`方法，比如`Comparator`接口新增了一系列`comparingXXX`的`static`方法，比如：

	public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
		Objects.requireNonNull(keyExtractor);
		return (Comparator<T> & Serializable)
		  (c1, c2) -> Integer.compare(keyExtractor.applyAsInt(c1), keyExtractor.applyAsInt(c2));
	}

使用这个`static`方法，以下两种方式也是等价的：

	Arrays.sort(cities, (first, second) -> Integer.compare(first.length(), second.length()));
	Arrays.sort(cities, Comparator.comparingInt(String::length));

所以，以后我们在设计自己的接口时，不需要再定义单独的工具类（如Collections/Collection)，在接口中使用`static`方法就行了。

### 参考

- [Java SE8 for the Really Impatient](http://www.amazon.cn/Java-SE8-for-the-Really-Impatient-A-Short-Course-on-the-Basics-Horstmann-Cay-S/dp/0321927761/ref=sr_1_2)
- [示例代码](https://gist.github.com/nkcoder/50c115a96c4e67164580#file-lambda-note-java)

