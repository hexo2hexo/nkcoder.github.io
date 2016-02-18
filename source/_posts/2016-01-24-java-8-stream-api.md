title: 使用Java 8中的Stream
categories: Backend
date: 2016-01-24 20:45:15
tags: java
---

Stream是Java 8 提供的高效操作集合类（Collection）数据的API。

## 1. 从Iterator到Stream

有一个字符串的list，要统计其中长度大于7的字符串的数量，用迭代来实现：

	List<String> wordList = Arrays.asList("regular", "expression", "specified", "as", "a", "string", "must");

    int countByIterator = 0;
    for (String word: wordList) {
        if (word.length() > 7) {
            countByIterator++;
        }
    }

<!-- more -->

用Stream实现：

	long countByStream= wordList.stream().filter(w -> w.length() > 7).count();

显然，用stream实现更简洁，不仅如此，stream很容易实现并发操作，比如：

	long countByParallelStream = wordList.parallelStream().filter(w -> w.length() > 7).count();

stream遵循的原则是：告诉我做什么，不用管我怎么做。比如上例：告诉stream通过多线程统计字符串长度，至于以什么顺序、在哪个线程中执行，由stream来负责；而在迭代实现中，由于计算的方式已确定，很难优化了。

Stream和Collection的区别主要有：

1. stream本身并不存储数据，数据是存储在对应的collection里，或者在需要的时候才生成的；
2. stream不会修改数据源，总是返回新的stream；
3. stream的操作是懒执行(lazy)的：仅当最终的结果需要的时候才会执行，比如上面的例子中，结果仅需要前3个长度大于7的字符串，那么在找到前3个长度符合要求的字符串后，`filter()`将停止执行；

使用stream的步骤如下：

1. 创建stream；
2. 通过一个或多个中间操作(intermediate operations)将初始stream转换为另一个stream；
3. 通过中止操作(terminal operation)获取结果；该操作触发之前的懒操作的执行，中止操作后，该stream关闭，不能再使用了；

在上面的例子中，`wordList.stream()`和`wordList.parallelStream()`是创建stream，`filter()`是中间操作，过滤后生成一个新的stream，`count()`是中止操作，获取结果。

## 2. 创建Stream的方式

1) 从array或list创建stream：

	Stream<Integer> integerStream = Stream.of(10, 20, 30, 40);
    String[] cityArr = {"Beijing", "Shanghai", "Chengdu"};
    Stream<String> cityStream = Stream.of(cityArr);
	Stream<String> nameStream = Arrays.asList("Daniel", "Peter", "Kevin").stream();
    Stream<String> cityStream2 = Arrays.stream(cityArr, 0, 1);
    Stream<String> emptyStream = Stream.empty();

2) 通过`generate`和`iterate`创建无穷stream：

	Stream<String> echos = Stream.generate(() -> "echo");
    Stream<Integer> integers = Stream.iterate(0, num -> num + 1);

3) 通过其它API创建stream：

	Stream<String> lines = Files.lines(Paths.get("test.txt"))

	String content = "AXDBDGXC";
    Stream<String> contentStream = Pattern.compile("[ABC]{1,3}").splitAsStream(content);

## 3. Stream转换

1) `filter()`用于过滤，即使原stream中满足条件的元素构成新的stream：

	List<String> langList = Arrays.asList("Java", "Python", "Swift", "HTML");
	Stream<String> filterStream = langList.stream().filter(lang -> lang.equalsIgnoreCase("java"));

2) `map()`用于映射，遍历原stream中的元素，转换后构成新的stream：

	List<String> langList = Arrays.asList("Java", "Python", "Swift", "HTML");
	Stream<String> mapStream = langList.stream().map(String::toUpperCase);

3) `flatMap()`用于将`[["ABC", "DEF"], ["FGH", "IJK"]]`的形式转换为`["ABC", "DEF", "FGH", "IJK"]`：

	Stream<String> cityStream = Stream.of("Beijing", "Shanghai", "Shenzhen");
	// [['B', 'e', 'i', 'j', 'i', 'n', 'g'], ['S', 'h', 'a', 'n', 'g', 'h', 'a', 'i'], ...]
    Stream<Stream<Character>> characterStream1 = cityStream.map(city -> characterStream(city));

    Stream<String> cityStreamCopy = Stream.of("Beijing", "Shanghai", "Shenzhen");
	// ['B', 'e', 'i', 'j', 'i', 'n', 'g', 'S', 'h', 'a', 'n', 'g', 'h', 'a', 'i', ...]
    Stream<Character> characterStreamCopy = cityStreamCopy.flatMap(city -> characterStream(city));

> 其中，`characterStream()`返回有参数字符串的字符构成的Stream<Character>;

4)  `limit()`表示限制stream中元素的数量，`skip()`表示跳过stream中前几个元素，`concat`表示将多个stream连接起来，`peek()`主要用于debug时查看stream中元素的值：

	Stream<Integer> limitStream = Stream.of(18, 20, 12, 35, 89).sorted().limit(3);
	Stream<Integer> skipStream = Stream.of(18, 20, 12, 35, 89).sorted(Comparator.reverseOrder()).skip(1);
	Stream<Integer> concatStream = Stream.concat(Stream.of(1, 2, 3), Stream.of(4, 5, 6));
	concatStream.peek(i -> System.out.println(i)).count();

> `peek()`是**intermediate operation**，所以后面需要一个**terminal operation**，如`count()`才能在输出中看到结果；

5) 有状态的(stateful)转换，即元素之间有依赖关系，如`distinct()`返回由唯一元素构成的stream，`sorted()`返回排序后的stream：

	Stream<String> distinctStream = Stream.of("Beijing", "Tianjin", "Beijing").distinct();
	Stream<String> sortedStream = Stream.of("Beijing", "Shanghai", "Chengdu").sorted(Comparator.comparing
		(String::length).reversed());

## 4. Stream reduction

`reduction`就是从stream中取出结果，是`terminal operation`，因此经过`reduction`后的stream不能再使用了。

### 4.1 Optional

Optional<T>表示或者有一个T类型的对象，或者没有值；

1) 创建Optional对象：

直接通过Optional的类方法：`of()`/`empty()`/`ofNullable()`：

	Optional<Integer> intOpt = Optional.of(10);
    Optional<String> emptyOpt = Optional.empty();
    Optional<Double> doubleOpt = Optional.ofNullable(5.5);

2) 使用Optional对象：

你当然可以这么使用：

	if (intOpt.isPresent()) {
        intOpt.get();
    }

但是，最好这么使用：

	doubleOpt.orElse(0.0);
    doubleOpt.orElseGet(() -> 1.0);
    doubleOpt.orElseThrow(RuntimeException::new);
    List<Double> doubleList = new ArrayList<>();
    doubleOpt.ifPresent(doubleList::add);

`map()`方法与`ifPresent()`用法相同，就是多个返回值，`flatMap()`用于Optional的链式表达：

	Optional<Boolean> addOk = doubleOpt.map(doubleList::add);
	Optional.of(4.0).flatMap(num -> Optional.ofNullable(num * 100)).flatMap(num -> Optional.ofNullable(Math.sqrt
		(num)));

### 4.2 简单的reduction

主要包含以下操作： `findFirst()`/`findAny()`/`allMatch`/`anyMatch()`/`noneMatch`，比如：

	Optional<String> firstWord = wordStream.filter(s -> s.startsWith("Y")).findFirst();
	Optional<String> anyWord = wordStream.filter(s -> s.length() > 3).findAny();
	wordStream.allMatch(s -> s.length() > 3);
	wordStream.anyMatch(s -> s.length() > 3);
	wordStream.noneMatch(s -> s.length() > 3);

### 4.3 reduce方法

1) `reduce(accumulator)`：参数是一个执行双目运算的`Functional Interface`，假如这个参数表示的操作为op，stream中的元素为x, y, z, ...，则`reduce()`执行的就是`x op y op z ...`，所以要求op这个操作具有结合性(associative)，即满足：`(x op y) op z = x op (y op z)`，满足这个要求的操作主要有：求和、求积、求最大值、求最小值、字符串连接、集合并集和交集等。另外，该函数的返回值是Optional的：

	Optional<Integer> sum1 = numStream.reduce((x, y) -> x + y);

2) `reduce(identity, accumulator)`：可以认为第一个参数为默认值，但需要满足`identity op x = x`，所以对于求和操作，`identity`的值为0，对于求积操作，`identity`的值为1。返回值类型是stream元素的类型：

	Integer sum2 = numStream.reduce(0, Integer::sum);

## 5. collect结果

1) `collect()`方法：

`reduce()`和`collect()`的区别是：

- `reduce()`的结果是一个值；
- `collect()`可以对stream中的元素进行各种处理后，得到stream中元素的值；

`Collectors`接口提供了很方便的创建`Collector`对象的工厂方法：

	// collect to Collection
    Stream.of("You", "may", "assume").collect(Collectors.toList());
    Stream.of("You", "may", "assume").collect(Collectors.toSet());
    Stream.of("You", "may", "assume").collect(Collectors.toCollection(TreeSet::new));
	// join element
    Stream.of("You", "may", "assume").collect(Collectors.joining());
    Stream.of("You", "may", "assume").collect(Collectors.joining(", "));
	// summarize element
    IntSummaryStatistics summary = Stream.of("You", "may", "assume").collect(Collectors.summarizingInt(String::length));
    summary.getMax();

2) `foreach()`方法：

`foreach()`用于遍历stream中的元素，属于`terminal operation`；
`forEachOrdered()`是按照stream中元素的顺序遍历，也就无法利用并发的优势；

	Stream.of("You", "may", "assume", "you", "can", "fly").parallel().forEach(w -> System.out.println(w));
    Stream.of("You", "may", "assume", "you", "can", "fly").forEachOrdered(w -> System.out.println(w));

3) `toArray()`方法：

得到由stream中的元素得到的数组，默认是Object[]，可以通过参数设置需要结果的类型：

	Object[] words1 = Stream.of("You", "may", "assume").toArray();
	String[] words2 = Stream.of("You", "may", "assume").toArray(String[]::new);

4) `toMap()`方法：

`toMap`: 将stream中的元素映射为<key, value>的形式，两个参数分别用于生成对应的key和value的值。比如有一个字符串stream，将首字母作为key，字符串值作为value，得到一个map：

	Stream<String> introStream = Stream.of("Get started with UICollectionView and the photo library".split(" "));
    Map<String, String> introMap = introStream.collect(Collectors.toMap(s -> s.substring(0, 1), s -> s));

如果一个key对应多个value，则会抛出异常，需要使用第三个参数设置如何处理冲突，比如仅使用原来的value、使用新的value，或者合并：

	Stream<String> introStream = Stream.of("Get started with UICollectionView and the photo library".split(" "));
	Map<Integer, String> introMap2 = introStream.collect(Collectors.toMap(s -> s.length(), s -> s, (existingValue, newValue) -> existingValue));

如果value是一个集合，即将key对应的所有value放到一个集合中，则需要使用第三个参数，将多个value合并：

	Stream<String> introStream3 = Stream.of("Get started with UICollectionView and the photo library".split(" "));
    Map<Integer, Set<String>> introMap3 = introStream3.collect(Collectors.toMap(s -> s.length(),
        s -> Collections.singleton(s), (existingValue, newValue) -> {
            HashSet<String> set = new HashSet<>(existingValue);
            set.addAll(newValue);
            return set;
        }));
    introMap3.forEach((k, v) -> System.out.println(k + ": " + v));

如果value是对象自身，则使用`Function.identity()`，如：

	Map<Integer, Person> idToPerson = people.collect(Collectors.toMap(Person::getId, Function.identity()));

`toMap()`默认返回的是HashMap，如果需要其它类型的map，比如TreeMap，则可以在第四个参数指定构造方法：

	Map<Integer, String> introMap2 = introStream.collect(Collectors.toMap(s -> s.length(), s -> s, (existingValue, newValue) -> existingValue, TreeMap::new));

## 6. Grouping和Partitioning

1)  `groupingBy()`表示根据某一个字段或条件进行分组，返回一个Map，其中key为分组的字段或条件，value默认为list，`groupingByConcurrent()`是其并发版本：

	Map<String, List<Locale>> countryToLocaleList = Stream.of(Locale.getAvailableLocales())
            .collect(Collectors.groupingBy(l -> l.getDisplayCountry()));

2) 如果`groupingBy()`分组的依据是一个bool条件，则key的值为true/false，此时与`partitioningBy()`等价，且`partitioningBy()`的效率更高：

	// predicate
    Map<Boolean, List<Locale>> englishAndOtherLocales = Stream.of(Locale.getAvailableLocales())
        .collect(Collectors.groupingBy(l -> l.getDisplayLanguage().equalsIgnoreCase("English")));

    // partitioningBy
    Map<Boolean, List<Locale>> englishAndOtherLocales2 = Stream.of(Locale.getAvailableLocales())
        .collect(Collectors.partitioningBy(l -> l.getDisplayLanguage().equalsIgnoreCase("English")));

3) `groupingBy()`提供第二个参数，表示`downstream`，即对分组后的value作进一步的处理：

返回set，而不是list：

	Map<String, Set<Locale>> countryToLocaleSet = Stream.of(Locale.getAvailableLocales()).collect(Collectors.groupingBy(l -> l.getDisplayCountry(), Collectors.toSet()));

返回value集合中元素的数量：

	Map<String, Long> countryToLocaleCounts = Stream.of(Locale.getAvailableLocales())
		.collect(Collectors.groupingBy(l -> l.getDisplayCountry(), Collectors.counting()));

对value集合中的元素求和：

	Map<String, Integer> cityToPopulationSum = Stream.of(cities)
            .collect(Collectors.groupingBy(City::getName, Collectors.summingInt(City::getPopulation)));

对value的某一个字段求最大值，注意value是Optional的：

	Map<String, Optional<City>> cityToPopulationMax = Stream.of(cities)
            .collect(Collectors.groupingBy(City::getName, Collectors.maxBy(Comparator.comparing(City::getPopulation))));

使用mapping对value的字段进行map处理：

	Map<String, Optional<String>> stateToNameMax = Stream.of(cities)
		.collect(Collectors.groupingBy(City::getState, Collectors.mapping(City::getName, Collectors.maxBy
			(Comparator.comparing(String::length)))));

	Map<String, Set<String>> stateToNameSet = Stream.of(cities)
        .collect(Collectors.groupingBy(City::getState, Collectors.mapping(City::getName, Collectors.toSet())));

通过`summarizingXXX`获取统计结果：

	Map<String, IntSummaryStatistics> stateToPopulationSummary = Stream.of(cities)
		.collect(Collectors.groupingBy(City::getState, Collectors.summarizingInt(City::getPopulation)));

`reducing()`可以对结果作更复杂的处理，但是`reducing()`却并不常用：

	Map<String, String> stateToNameJoining = Stream.of(cities)
		.collect(Collectors.groupingBy(City::getState, Collectors.reducing("", City::getName,
			(s, t) -> s.length() == 0 ? t : s + ", " + t)));

比如上例可以通过mapping达到同样的效果：

	Map<String, String> stateToNameJoining2 = Stream.of(cities)
            .collect(Collectors.groupingBy(City::getState, Collectors.mapping(City::getName, Collectors.joining(", ")
            )));

## 7. Primitive Stream

`Stream<Integer>`对应的Primitive Stream就是`IntStream`，类似的还有`DoubleStream`和`LongStream`。

1) Primitive Stream的构造：`of()`, `range()`, `rangeClosed()`, `Arrays.stream()`:

	IntStream intStream = IntStream.of(10, 20, 30);
	IntStream zeroToNintyNine = IntStream.range(0, 100);
	IntStream zeroToHundred = IntStream.rangeClosed(0, 100);
	double[] nums = {10.0, 20.0, 30.0};
	DoubleStream doubleStream = Arrays.stream(nums, 0, 3);

2) Object Stream与Primitive Stream之间的相互转换，通过`mapToXXX()`和`boxed()`：

	// map to
	Stream<String> cityStream = Stream.of("Beijing", "Tianjin", "Chengdu");
	IntStream lengthStream = cityStream.mapToInt(String::length);

	// box
	Stream<Integer> oneToNine = IntStream.range(0, 10).boxed();
3) 与Object Stream相比，Primitive Stream的特点：

`toArray()`方法返回的是对应的Primitive类型：

	int[] intArr = intStream.toArray();

自带统计类型的方法，如：`max()`, `average()`, `summaryStatistics()`:

	OptionalInt maxNum = intStream.max();
	IntSummaryStatistics intSummary = intStream.summaryStatistics();

## 8. Parallel Stream

1) Stream支持并发操作，但需要满足以下几点：

构造一个paralle stream，默认构造的stream是顺序执行的，调用`paralle()`构造并行的stream：

	IntStream scoreStream = IntStream.rangeClosed(10, 30).parallel();

要执行的操作必须是可并行执行的，即并行执行的结果和顺序执行的结果是一致的，而且必须保证stream中执行的操作是线程安全的：

	int[] wordLength = new int[12];
	Stream.of("It", "is", "your", "responsibility").parallel().forEach(s -> {
		if (s.length() < 12) wordLength[s.length()]++;
	});

这段程序的问题在于，多线程访问共享数组`wordLength`，是非线程安全的。解决的思路有：1）构造AtomicInteger数组；2）使用`groupingBy()`根据length统计；

2) 可以通过并行提高效率的常见场景：

使stream无序：对于`distinct()`和`limit()`等方法，如果不关心顺序，则可以使用并行：

	LongStream.rangeClosed(5, 10).unordered().parallel().limit(3);
    IntStream.of(14, 15, 15, 14, 12, 81).unordered().parallel().distinct();

在`groupingBy()`的操作中，map的合并操作是比较重的，可以通过`groupingByConcurrent()`来并行处理，不过前提是parallel stream：

    Stream.of(cities).parallel().collect(Collectors.groupingByConcurrent(City::getState));

在执行stream操作时不能修改stream对应的collection；

stream本身是不存储数据的，数据保存在对应的collection中，所以在执行stream操作的同时修改对应的collection，结果是未定义的：

	// ok
	Stream<String> wordStream = wordList.stream();
	wordList.add("number");
	wordStream.distinct().count();

	// ConcurrentModificationException
	Stream<String> wordStream = wordList.stream();
	wordStream.forEach(s -> { if (s.length() >= 6) wordList.remove(s);});

## 9. Functional Interface

仅包含一个抽象方法的interface被成为`Functional Interface`，比如：`Predicate`, `Function`, `Consumer`等。此时我们一般传入一个lambda表达式或`Method Reference`。

常见的`Functional Interface`有：

	Functional Interface 	Parameter 	Return Type 	Description Types
	Supplier<T> 			None 		T				Supplies a value of type T
	Consumer<T> 			T 			void			Consumes a value of type T
	BiConsumer<T, U> 		T,U 		void			Consumes values of types T and U
	Predicate<T> 			T			boolean			A Boolean-valued function
	ToIntFunction<T> 		T 			int				An int-, long-, or double-valued function
	ToLongFunction<T> 		T			long
	ToDoubleFunction<T> 	T			double
	IntFunction<R> 			int 		R				A function with argument of type int, long, or double
	LongFunction<R> 		long
	DoubleFunction<R> 		double
	Function<T, R> 			T 			R				A function with argument of type T
	BiFunction<T, U, R> 	T,U 		R				A function with arguments of types T and U
	UnaryOperator<T> 		T 			T				A unary operator on the type T
	BinaryOperator<T> 		T,T 		T				A binary operator on the type T

## 参考

- [Java SE8 for the Really Impatient](http://www.amazon.cn/Java-SE8-for-the-Really-Impatient-A-Short-Course-on-the-Basics-Horstmann-Cay-S/dp/0321927761/ref=sr_1_2)
- [Stream示例代码](https://gist.github.com/nkcoder/50c115a96c4e67164580#file-java-8-stream-api-java)
