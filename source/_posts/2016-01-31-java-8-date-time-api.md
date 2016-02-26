title: Java 8新的日期和时间API
categories: Backend
keywords: Java 8, Date Time API
date: 2016-01-31 16:23:29
tags: java
---

## 1. Instant 与 Duration

1) `Instant`表示某一个时间点的时间戳，可以类比于`java.uti.Date`。支持各种运算操作：

    Instant begin = Instant.now();
    begin.plus(5, ChronoUnit.SECONDS);
    begin.minusMillis(50);
    begin.isBefore(Instant.now());

    begin.toEpochMilli();

<!-- more -->

2) `Duration`表示`Instant`之间的时间差，可以用来统计任务的执行时间，也支持各种运算操作，比如：

	Instant begin = Instant.now();
	// do some work
	Instant end = Instant.now();
	Duration elapsed = Duration.between(begin, end);
	elapsed.toMillis()

	elapsed.dividedBy(10).minus(Duration.ofMillis(10)).isNegative();
	elapsed.isZero();
	elapsed.plusHours(3);

## 2. LocalDate 与 Period

1) `LocalDate`用于表示日期，与时区(TimeZone)无关。

创建`LocalDate`：

	LocalDate now = LocalDate.now();
	LocalDate today = LocalDate.of(2016, 1, 31);
	LocalDate today2 = LocalDate.of(2016, Month.JANUARY, 31);   // JANUARY = 1, ..., DECEMBER = 12

支持的操作：

	today2.getDayOfWeek().getValue();   // Monday = 1, ..., Sunday = 7
	LocalDate dayOfYear = Year.now().atDay(220);
	YearMonth april = Year.of(2016).atMonth(Month.APRIL);

注意，有些操作得到的日期可能是不存在的，比如`2016-01-31`增加1个月后为`2016-02-31`，该日期是不存在的，返回值为该月的最后一天，即`2016-02-29`:

	LocalDate nextMonth = LocalDate.of(2016, 1, 31).plusMonths(1);  // 2016-02-29

2) `Period`用来表示两个`LocalDate`之间的时间差，支持各种运算操作：

	LocalDate fiveDaysLater = LocalDate.now().plusDays(5);
	Period period = LocalDate.now().until(fiveDaysLater).plusMonths(2);
	period.isNegative();

3) `TemporalAdjusters`用于表示**某个月第一天、下个周一**等日期：

	LocalDate.now().with(TemporalAdjusters.firstDayOfMonth());
    LocalDate.now().with(TemporalAdjusters.lastInMonth(DayOfWeek.SUNDAY));
    LocalDate.now().with(TemporalAdjusters.nextOrSame(DayOfWeek.MONDAY));

## 3. LocalTime

1) `LocalTime`表示时间，没有日期，与时区(TimeZone)无关：

	LocalTime.now().isBefore(LocalTime.of(16, 2, 1));
    LocalTime.now().plusHours(2).getHour();

2) `LocalDateTime`表示日期和时间，适用于时区固定不变的场合(`LocalDateTime`使用系统默认的时区)，如果需要根据时区调整日期和时间，应该使用`ZonedDateTime`:

	LocalDateTime.now().plusDays(3).minusHours(5).isAfter(LocalDateTime.of(2016, 1, 30, 10, 20, 30));

## 4. ZonedDateTime

1) `ZonedDateTime`表示带时区的日期和时间，支持的操作与`LocalDateTime`非常类似：

	Set<String> zones = ZoneId.getAvailableZoneIds();
	ZonedDateTime.now(ZoneId.of("Asia/Shanghai")).plusMonths(1).minusHours(3)
        .isBefore(ZonedDateTime.now());

2) `ZonedDateTime`与`LocalDateTime`、`Instant`之间可以相互转换：

	ZonedDateTime nowOfShanghai = LocalDateTime.now().atZone(ZoneId.of("Asia/Shanghai"));
	ZonedDateTime.now(ZoneId.of("UTC")).toLocalDate();
	ZonedDateTime nowOfShanghai2 = Instant.now().atZone(ZoneId.of("Asia/Shanghai"));
	ZonedDateTime.of(LocalDate.now(), LocalTime.now(), ZoneId.of("UTC")).toInstant();

## 5. Formatting 与 Parsing

1) 要格式化或者解析日期时，需要使用到`DateTimeFormatter`，用来定义日期或时间的格式：

	// 2016-01-31T15:39:31.481
	DateTimeFormatter.ISO_LOCAL_DATE_TIME.format(LocalDateTime.now());
	// Jan 31, 2016 3:50:04 PM
	DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM).format(LocalDateTime.now());
	// Sun 2016-01-31 15:50:04
	DateTimeFormatter.ofPattern("E yyyy-MM-dd HH:mm:ss").format(LocalDateTime.now());

	LocalDateTime.parse("2016-01-31 15:51:00-0400", 
        DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ssxx"));
    LocalDate.parse("2016-01-31");

2) 日期和时间格式化的常见格式：

    年       yy: 16      yyyy: 2016
    月       M: 1        MM: 01
    日       d: 3        dd: 03
    周       e: 3        E:	Web
    时       H: 9        HH: 09
    钟       mm: 02
    秒       ss: 00
    纳秒      nnnnnn:000000
    时区偏移    x: -04     xx:-0400

## 参考

- [Java SE8 for the Really Impatient](http://www.amazon.cn/Java-SE8-for-the-Really-Impatient-A-Short-Course-on-the-Basics-Horstmann-Cay-S/dp/0321927761/ref=sr_1_2))
- [示例代码](https://gist.github.com/nkcoder/50c115a96c4e67164580)

