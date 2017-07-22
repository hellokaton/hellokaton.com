---
title: 跟上Java8 - 日期和时间实用技巧
date: 2017-07-20
category: ["跟上Java8"]
tags: ["Java8", "datetime"]
---

当你开始使用Java操作日期和时间的时候，会有一些棘手。你也许会通过`System.currentTimeMillis()`
来返回`1970年1月1日`到今天的毫秒数。或者使用`Date`类来操作日期；当遇到加减月份、天数的时候
你又需要用到`Calendar`类；当需要格式化日期的时候需要使用`java.text.DateFormat`类。
总而言之在Java中操作日期不是很方便，以至于很多开发者不得不使用第三方库，比如: [joda-time](http://joda-time.sourceforge.net)

<!-- more -->

{% image /static/img/article/java8-clock.jpeg 600 380 时间 %}

## 现有API存在的问题

- 线程安全: `Date`和`Calendar`不是线程安全的，你需要编写额外的代码处理线程安全问题
- API设计和易用性: 由于`Date`和`Calendar`的设计不当你无法完成日常的日期操作
- `ZonedDate`和`Time`: 你必须编写额外的逻辑处理时区和那些旧的逻辑

好在[JSR 310](http://jcp.org/en/jsr/detail?id=310)规范中为Java8添加了新的API，
在`java.time`包中，新的API纠正了过去的缺陷，

## 新的日期API

- `ZoneId`: 时区ID，用来确定Instant和LocalDateTime互相转换的规则
- `Instant`: 用来表示时间线上的一个点
- `LocalDate`: 表示没有时区的日期, LocalDate是不可变并且线程安全的
- `LocalTime`: 表示没有时区的时间, LocalTime是不可变并且线程安全的
- `LocalDateTime`: 表示没有时区的日期时间, LocalDateTime是不可变并且线程安全的
- `Clock`: 用于访问当前时刻、日期、时间，用到时区
- `Duration`: 用秒和纳秒表示时间的数量

最常用的就是`LocalDate`、`LocalTime`、`LocalDateTime`了，从它们的名字就可以看出是操作日期
和时间的。这些类是主要用于当时区不需要显式地指定的上下文。在本章节中我们将讨论最常用的api。

### LocalDate

`LocalDate`代表一个IOS格式(yyyy-MM-dd)的日期，可以存储 `生日`、`纪念日`等日期。
获取当前的日期：

```java
LocalDate localDate = LocalDate.now();
System.out.println("localDate: " + localDate);
```

```bash
localDate: 2017-07-20
```

`LocalDate`可以指定特定的日期，调用`of`或`parse`方法返回该实例：

```java
LocalDate.of(2017, 07, 20);

LocalDate.parse("2017-07-20");
```

当然它还有一些其他方法，我们一起来看看：

**为今天添加一天，也就是获取明天**

```java
LocalDate tomorrow = LocalDate.now().plusDays(1);
```

**从今天减去一个月**

```java
LocalDate prevMonth = LocalDate.now().minus(1, ChronoUnit.MONTHS);
```

下面写两个例子，分别解析日期 `2017-07-20`，获取每周中的`星期`和每月中的`日`：

```java
DayOfWeek thursday = LocalDate.parse("2017-07-20").getDayOfWeek();
System.out.println("周四: " + thursday);

int twenty = LocalDate.parse("2017-07-20").getDayOfMonth();
System.out.println("twenty: " + twenty);
```

试试今年是不是闰年:

```java
boolean leapYear = LocalDate.now().isLeapYear();
System.out.println("是否闰年: " + leapYear);
```

判断是否在日期之前或之后:

```java
boolean notBefore = LocalDate.parse("2017-07-20")
                .isBefore(LocalDate.parse("2017-07-22"));
System.out.println("notBefore: " + notBefore);

boolean isAfter = LocalDate.parse("2017-07-20").isAfter(LocalDate.parse("2017-07-22"));
System.out.println("isAfter: " + isAfter);
```

获取这个月的第一天:

```java
LocalDate firstDayOfMonth = LocalDate.parse("2017-07-20")
                .with(TemporalAdjusters.firstDayOfMonth());
System.out.println("这个月的第一天: " + firstDayOfMonth);

firstDayOfMonth = firstDayOfMonth.withDayOfMonth(1);
System.out.println("这个月的第一天: " + firstDayOfMonth);
```

判断今天是否是我的生日，例如我的生日是 `2009-07-20`

```java
LocalDate birthday = LocalDate.of(2009, 07, 20);
MonthDay birthdayMd = MonthDay.of(birthday.getMonth(), birthday.getDayOfMonth());
MonthDay today = MonthDay.from(LocalDate.now());
System.out.println("今天是否是我的生日: " + today.equals(birthdayMd));
```

### LocalTime

`LocalTime`表示一个时间，而不是日期，下面介绍一下它的使用方法。

获取现在的时间，输出`15:01:22.144`

```java
LocalTime now = LocalTime.now();
System.out.println("现在的时间: " + now);
```

将一个字符串时间解析为`LocalTime`，输出`15:02`

```java
LocalTime nowTime = LocalTime.parse("15:02");
System.out.println("时间是: " + nowTime);
```

使用静态方法`of`创建一个时间

```java
LocalTime nowTime = LocalTime.of(15, 02);
System.out.println("时间是: " + nowTime);
```

使用解析字符串的方式并添加一小时，输出`16:02`

```java
LocalTime nextHour = LocalTime.parse("15:02").plus(1, ChronoUnit.HOURS);
System.out.println("下一个小时: " + nextHour);
```

获取时间的`小时`、`分钟`

```java
int hour = LocalTime.parse("15:02").getHour();
System.out.println("小时: " + hour);
int minute = LocalTime.parse("15:02").getMinute();
System.out.println("分钟: " + minute);
```

我们也可以通过之前类似的API检查一个时间是否在另一个时间之前、之后

```java
boolean isBefore = LocalTime.parse("15:02").isBefore(LocalTime.parse("16:02"));
boolean isAfter = LocalTime.parse("15:02").isAfter(LocalTime.parse("16:02"));
System.out.println("isBefore: " + isBefore);
System.out.println("isAfter: " + isAfter);
```

输出 `isBefore: true`, `isAfter: false`。

在`LocalTime`类中也将每天的开始和结束作为常量供我们使用:

```java
System.out.println(LocalTime.MAX);
System.out.println(LocalTime.MIN);
```

输出:

```java
23:59:59.999999999
00:00
```

`LocalTime`就这些了，下面我们来了解一下`LocalDateTime`

### LocalDateTime

`LocalDateTime`是用来表示日期和时间的，这是一个最常用的类之一。

获取当前的日期和时间:

```java
LocalDateTime now = LocalDateTime.now();
System.out.println("现在: " + now);
```

输出

```bash
现在: 2017-07-20T15:17:19.926
```

下面使用静态方法和字符串的方式分别创建`LocalDateTime`对象

```java
LocalDateTime.of(2017, Month.JULY, 20, 15, 18);

LocalDateTime.parse("2017-07-20T15:18:00");
``

同时`LocalDateTime`也提供了相关API来对日期和时间进行增减操作:

```java
LocalDateTime tomorrow = now.plusDays(1);
System.out.println("明天的这个时间: " + tomorrow);

LocalDateTime minusTowHour = now.minusHours(2);
System.out.println("两小时前: " + minusTowHour);
```

这个类也提供一系列的`get`方法来获取特定单位:

```java
Month month = now.getMonth();
System.out.println("当前月份: " + month);
```

## 日期格式化

在日常开发中我们用到最多的也许就是日期、时间的格式化了，那在Java8种该如何操作呢？

```java
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

System.out.println("默认格式化: " + now);
System.out.println("自定义格式化: " + now.format(dateTimeFormatter));

LocalDateTime localDateTime = LocalDateTime.parse("2017-07-20 15:27:44", dateTimeFormatter);
System.out.println("字符串转LocalDateTime: " + localDateTime);
```

也可以使用`DateTimeFormatter`的`format`方法将日期、时间格式化为字符串

```java
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
String dateString = dateTimeFormatter.format(LocalDate.now());
System.out.println("日期转字符串: " + dateString);
```

## 日期周期

`Period`类用于修改给定日期或获得的两个日期之间的区别。

给初始化的日期添加5天:

```java
LocalDate initialDate = LocalDate.parse("2017-07-20");
LocalDate finalDate   = initialDate.plus(Period.ofDays(5));
System.out.println("初始化日期: " + initialDate);
System.out.println("加日期之后: " + finalDate);
```

周期API中提供给我们可以比较两个日期的差别，像下面这样获取差距天数:

```java
long between = ChronoUnit.DAYS.between(initialDate, finalDate);
System.out.println("差距天数: " + between);
```

上面的代码会返回5，当然你想获取两个日期相差多少小时也是简单的。

## 与遗留代码转换

在之前的代码中你可能出现了大量的`Date`类，如何将它转换为Java8种的时间类呢？

`Date`和`Instant`互相转换

```java
Date date = Date.from(Instant.now());
Instant instant = date.toInstant();
```

`Date`转换为`LocalDateTime`

```java
LocalDateTime localDateTime = LocalDateTime.from(new Date());
System.out.println(localDateTime);
```

`LocalDateTime`转`Date`

```java
Date date =
    Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
```

`LocalDate`转`Date`

```java
Date date =
    Date.from(LocalDate.now().atStartOfDay().atZone(ZoneId.systemDefault()).toInstant());
```
