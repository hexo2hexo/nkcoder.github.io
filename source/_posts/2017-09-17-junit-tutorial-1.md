title: 单元测试系列：使用Junit
categories: Backend
date: 2017-09-17 10:43:09
tags: [单元测试, Junit]
---

    Junit版本：4.12

本文简单介绍`Junit`在实际工作中的常见的一些用法和API。

添加gradle依赖：

    ```bash
    testCompile("junit:junit:4.12") {
        exclude group: 'org.hamcrest'
    }
    testCompile "org.hamcrest:hamcrest-library:1.3"
    ```

## 1. 基本注解

    ```java
    @BeforeClass: 在class中的所有测试方法执行之前运行一次
    @AfterClass：在class中的所有测试方法执行之后运行一次
    @Before：在每一个测试方法执行之前运行一次
    @After：在每一个测试方法执行之后运行一次
    @Test：表示一个测试方法
    ```

<!--more-->

其中`@BeforeClass`和`@AfterClass`修饰的方法签名为`public static void`，`@Before`、`@After`和`@Test`修饰的方法的签名为`public void`。

    ```java
    @BeforeClass
    public static void run_before_class() {
        log.info("run once before class.");
    }

    @AfterClass
    public static void run_after_class() {
        log.info("run once after class.");
    }

    @Before
    public void run_before_test_method() {
        log.info("run before every test method.");
    }

    @After
    public void run_after_test_method() {
        log.info("run before every test method.");
    }

    @Test
    public void test_method1() {
        log.info("test method 1");
    }

    @Test
    public void test_method2() {
        log.info("test method 2");
    }
    ```

## 2. 测试异常

测试异常主要有三种方式：

- 通过`@Test`的`expect`属性
- 通过`try...catch...`搭配`fail()`方法使用，使用`fail()`的原因是，如果测试的方法没有抛出指定的异常，则该单元测试就会通过
- 通过`@Rule`注解

    ```java
    @Test(expected = IndexOutOfBoundsException.class)
    public void test_exception_with_expect_attribute() {
        new ArrayList<>().get(0);
    }

    @Test
    public void test_exception_with_try_catch_fail() {
        try {
            new ArrayList<>().get(0);
            fail();
        } catch (IndexOutOfBoundsException e) {
            assertThat(e.getMessage(), is("Index: 0, Size: 0"));
        }
    }

    @Rule
    public ExpectedException expectedException = ExpectedException.none();

    @Test
    public void test_exception_with_rule() {
        expectedException.expect(IndexOutOfBoundsException.class);
        expectedException.expectMessage(is("Index: 0, Size: 0"));
        expectedException.expectMessage(containsString("Index: 0, Size: 0"));
        new ArrayList<>().get(0);
    }
    ```

## 3. 忽略测试

通过`@Ignore`可以忽略单元测试，如果用在方法上，表示该方法不作为单元测试被执行，如果用在类上，表示该类中的所有单元测试方法都不被执行。

为什么要用`@Ignore`忽略单元测试，而不是直接注释掉单元测试或者注释掉`@Test`注解呢？因为被`@Ignore`的单元测试会显示在最后的测试结果中，另外，在
多人协作的多模块项目中，ignore掉别的模块中执行不通过的单元测试，可以避免整个项目都无法运行。

    ```java
    @Ignore("will be add later!")
    public class JunitException {
        @Test(expected = IndexOutOfBoundsException.class)
        @Ignore
        public void test_exception_with_expect_attribute() {
            new ArrayList<>().get(0);
        }
    }
    ```

## 4. 设置超时

`@Test`的`tiimeout`属性可以设置超时，单位是毫秒。

    ```java
    @Test(timeout = 1000)
    public void test_timeout() {
        try {
            TimeUnit.SECONDS.sleep(2000);
        } catch (InterruptedException e) {
            log.error(e.getMessage(), e);
        }
    }
    ```

## 5. 测试list的常用方法

    ```java
    List<String> actual = Arrays.asList("a", "b", "c");
    List<String> expected = Arrays.asList("a", "b", "c");
    List<Integer> numList = Arrays.asList®(1, 2, 3);

    assertThat(actual, is(expected));
    assertThat(actual, hasItem("a"));
    assertThat(actual, hasItems("c", "b"));
    assertThat(actual, containsInAnyOrder("c", "b", "a"));

    assertThat(actual.size(), is(3));
    assertThat(actual, hasSize(3));

    assertThat(numList, everyItem(greaterThanOrEqualTo(1)));
    ```

## 6. 测试map的常用方法

    ```java
    Map<String, Integer> actual = new HashMap<>();
    actual.put("00001", 1);
    actual.put("00002", 2);
    actual.put("00003", 3);

    Map<String, Integer> expected = new HashMap<>();
    expected.put("00001", 1);
    expected.put("00002", 2);
    expected.put("00003", 3);

    assertThat(actual, is(expected));
    assertThat(actual, hasEntry("00001", 1));
    assertThat(actual, not(hasEntry("00004", 4)));
    assertThat(actual, hasKey("00002"));
    assertThat(actual, hasValue("00003"));
    ```

### 参考

- [junit4](http://junit.org/junit4/)
- [JUnit Tutorial](http://www.mkyong.com/tutorials/junit-tutorials/)