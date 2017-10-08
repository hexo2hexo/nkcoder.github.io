title: 什么是空指针异常，如何避免？
categories: Backend
date: 2017-10-08 14:41:04
tags: [java, 问答系列]
---

**什么是空指针异常？**

对于引用类型的变量，如果在使用的时候，它的值是空的(null)，就会报空指针异常。当变量的值为null的时候，表示这个引用不指向内存中的任何区域，所以使用（解引用）时就会触发空指针异常。

**常见发生场景**

- 直接使用值为null的变量，如：

```java
Integer num = null;
System.out.println("result: " + (num + 10));
```

<!-- more -->

- 调用值为null的对象的方法：

```java
SomeClass someObject = null;
someObject.someMethod();
```

- 值为null的集合类型变量，遍历集合元素或者调用方法：

```java
List<String> names = null;
names.forEach(System.out::println);
names.add("daniel");
```

**如何避免**

最简单直接的办法就是，如果对象可能为null，使用之前先判断：

```java
String value = null;

if (value == null) {
  // ...
}

if (Objects.isNull(value)) {
  // ...
}

if (StringUtils.isEmpty(value)) {
  // ...
}

checkArgument(value != null, "invalid value");
```

另外，可以参考以下建议：

- 如果返回的引用类型的变量可能为null，则返回Optional<>，如：

```java
public Optional<String> getResult(boolean valid) {
  String result = null;
  if (valid) {
    result = getResultFromRemote();
  }
  return Optional.ofNullable(result);
}
```

- 如果返回值是集合类型(List或Map)，如果可以，尽量返回空集合，而不是null，如：

```java
public List<String> getResults(boolean valid) {
  List<String> results = getResultsFromRemote();
  if (results != null) {
    return results;
  }
  return new ArrayList<>();
}
```

- 使用注解`@Nullable`或`@NonNull`

- 使用`final`修饰变量，提醒自己做合理的初始化

- 对象比较时，将有值的变量放在前面，如：

```java
if ("name".equals(someName)) {
  // ... 
}
```

**如何修复**

空指针异常的修复比较简单，根据异常堆栈定位发生异常的代码行，然后进一步确定可能为null的变量即可。

```java
@Test
public void test_npe() {
  List<String> names = null;
  names.forEach(System.out::println);
}
```

```text
java.lang.NullPointerException
	at NullPointerExceptionTest.test_primitive_type(NullPointerExceptionTest.java:12)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
```

异常堆栈的重要信息为：

    java.lang.NullPointerException
      at NullPointerExceptionTest.test_primitive_type(NullPointerExceptionTest.java:12)

表示异常发生在方法`NullPointerExceptionTest`的12行，则可以进一步定位是变量names为null。

## 参考

- [NullPointerException](http://docs.oracle.com/javase/8/docs/api/java/lang/NullPointerException.html)