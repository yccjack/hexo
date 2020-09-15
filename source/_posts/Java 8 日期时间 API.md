---
title: Java 8 日期时间 API
categories: java 
tags: [java]
date: 2019-09-15 
cover: https://mysticalyu.gitee.io/pic/img/20200407235917-www-ycfcg-com-3.jpg

---

![](https://mysticalyu.gitee.io/pic/img/20200407235917-www-ycfcg-com-3.jpg)

java 8 通过发布新的Date-Time API (JSR 310)来进一步加强对日期和时间的处理。



 <!-- more -->



#  **Java 8 日期时间 API**



在旧版本的Java中，日期时间API存在诸多问题，其中有:

+  **非线程安全** - java.util.Date 是非线程安全的，所有的日期类都是可变的，这是Java日期类最大的问题之一。
+ **设计很差** - Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。java.Date同时包含日期和时间，而java.Date仅包含日期，将其纳入java.sql包并不合理，另外这两个类都有相同的名字，本身就是一个非常糟糕的设计。
+ **时区处理麻烦** - 日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calenda和java.util.TimeZone类，单他们同样存在上述的所有问题。

Java 8 在java.time包下提供了很多新的API。以下为两个比较重要的API：
+ **Local(本地)** - 简化了日期时间的处理，没有时区的问题。
+ **Zoned(时区)** - 通过制定的时区处理日期时间。

新的java.time包涵盖了所有处理日期，时间，日期/时间，时区，时刻(instants),过程(during),与时钟(clock)的操作。

##  **1.本地化日期时间 API**

LocalDate/LocalTime和LocalDateTime类可以在处理时区不是必须的情况。代码如下

```java
public class Java8Tester{
    public static void main(String args[]){
        Java8Tester java8Tester = new Java8Tester();
        java8Tester.testLocalDateTime();
    }

    public void testLocalDateTime(){
        //获取当前日期时间
        LocalDateTime currentTime = LocalDateTime.now();
        System.out.println("当前时间:"+currentTime);
        LocalDate date1 = currentTime.toLocalDate();
        System.out.println("date1: "+date1);
        Month month = currentTime.getMonth();
        int day = currentTime.getDayOfMonth();
        int senconds = currentTime.getSecond();
        System.out.println("月："+ month + "，日：" + day + ",秒：" + senconds);

        //指定年日 2019/09/10
        LocalDateTime date2  = currentTime.withDayOfMonth(10).withYear(2019);
        System.out.println("date2: "+ date2);

        //指定年月日   2019-11-10
        LocalDate date3 = LocalDate.of(2019,Month.NOVEMBER,10);
        System.out.println("date3: "+ date3);

        //22时15分钟
        LocalTime date4 = LocalTime.of(22,10);
        System.out.println("date4: "+ date4);

        //解析字符串
        LocalTime date5 = LocalTime.parse("20:15:30");
        System.out.println(date5);
    }
}

```

执行以上脚本，输出结果为：

```java
当前时间: 2018-06-08T15:19:16.910

date1:2018-06-08

月: JUNE, 日: 8, 秒: 16

date2:2012-06-10T15:19:16.910

date3:2014-12-12

date4:22:15

date5:20:15:30
```

## **2 使用时区的日期时间API**

如果我们需要考虑到时区，就可以使用时区的日期时间API：

```java
public class Java8Tester {
    public static void main(String args[]) {
        Java8Tester java8Tester = new Java8Tester();
        java8Tester.testZonedDateTime();
    }

    public void testZonedDateTime() {
        // 获取当前时间日期
        ZonedDateTime date1 = ZonedDateTime.parse("2019-12-03T10:15:30+05:30[Asia/Shanghai]");
        System.out.println("date1: " + date1);
        ZoneId id = ZoneId.of("Europe/Paris");
        System.out.println("ZoneId: " + id);
        ZoneId currentZone = ZoneId.systemDefault();
        System.out.println("当期时区: " + currentZone);
    }
}
```

执行以上脚本，输出结果为：

```java
date1:2015-12-03T10:15:30+08:00[Asia/Shanghai]

ZoneId:Europe/Paris

当期时区: Asia/Shanghai
```



## **3.常用代码**

```java
		LocalDate today = LocalDate.now();
        System.out.println("今天的日期是："+today);
        System.out.println("-----------------------------");
        int year = today.getYear();
        int month = today.getMonthValue();
        int day = today.getDayOfMonth();
        System.out.println("年："+year+",月："+month+",日："+day);
        System.out.println("-----------------------------");
        LocalDate birthday = LocalDate.of(2011, 11, 11);
        System.out.println("特定日期："+birthday);
        System.out.println("-----------------------------");
        System.out.println("今天的日期是2011-11-11吗？"+today.equals(birthday));
        System.out.println("-----------------------------");
        LocalDate dayofbirth = LocalDate.of(2008, 11, 20);
        LocalDate tday = LocalDate.now();
	    // 获取生日的月、日
        MonthDay birthMonthDay = MonthDay.of(dayofbirth.getMonth(), dayofbirth.getDayOfMonth());
        MonthDay currentMonthDay = MonthDay.from(tday);
        if (currentMonthDay.equals(birthMonthDay)) {
            System.out.println("今天是你的生日");
        }else{
            System.out.println("对不起，今天不是你的生日");
        }
        System.out.println("-----------------------------");
        LocalTime localTime = LocalTime.now();
        System.out.println("现在的时间是"+localTime);
        System.out.println("-----------------------------");
        LocalTime twoHourLaterTime = localTime.plusHours(2);
        System.out.println("当前时间两小时后的时间是"+twoHourLaterTime);
        System.out.println("-----------------------------");
        LocalDate oneWeekLaterDate = today.plus(1,ChronoUnit.WEEKS);
        System.out.println("两周后的日期是："+oneWeekLaterDate);
        System.out.println("-----------------------------");
        LocalDate oneYearBeforeDate = today.minus(1, ChronoUnit.YEARS);
        System.out.println("一年前的日期是："+oneYearBeforeDate);
        LocalDate oneYearLaterDate = today.plus(1, ChronoUnit.YEARS);
        System.out.println("一年后的日期是："+oneYearLaterDate);
        System.out.println("-----------------------------");
        Clock clock = Clock.systemUTC();
        System.out.println("Clock:"+clock);
        Clock.systemDefaultZone();
        System.out.println("Clock:"+clock.millis());
        System.out.println("-----------------------------");
        LocalDate day1 = LocalDate.of(2011, 12, 15);
        LocalDate day2 = LocalDate.of(2011, 9, 17);
        System.out.println("day1是否在day2之后："+day1.isAfter(day2));
        System.out.println("day1是否在day2之前："+day1.isBefore(day2));
        System.out.println("-----------------------------");
        LocalDateTime todaytime = LocalDateTime.now();
        System.out.println("当前的日期时间："+todaytime);
        ZoneId zone = ZoneId.of(ZoneId.SHORT_IDS.get("ACT"));
        ZonedDateTime dateandtimeinNewYork = ZonedDateTime.of(todaytime, zone);
        System.out.println("现在时区的时间在特定时区的时间："+dateandtimeinNewYork);
        System.out.println("-----------------------------");
        YearMonth currentYearMonth = YearMonth.now();
        System.out.println("今年的当前月"+currentYearMonth+"有"+currentYearMonth.lengthOfMonth()+"天");
        YearMonth creditCardExpiry = YearMonth.of(2018, Month.FEBRUARY);
        System.out.println("您输入的年月日期是："+creditCardExpiry);
        System.out.println("-----------------------------");
        System.out.println("今年"+today+"是否是闰年："+today.isLeapYear());
        System.out.println("-----------------------------");
        LocalDate day3 = LocalDate.of(2002, 12, 10);
        LocalDate day4 = LocalDate.of(2001, 9, 12);
        Period period = Period.between(day3, day4);
        System.out.println(day3+"和"+day4+"之间相差"+period.getMonths()+"月");
        System.out.println("-----------------------------");
        LocalDateTime datetime = LocalDateTime.of(2016, Month.APRIL, 14, 14, 02, 24);
        ZoneOffset offset = ZoneOffset.of("+05:30");
        OffsetDateTime offsetdatetime = OffsetDateTime.of(datetime, offset);
        System.out.println("日期和时间在时区上的偏移时间："+offsetdatetime);
        System.out.println("-----------------------------");
        Instant timestamp = Instant.now();
        System.out.println("当前时间戳："+timestamp);
        System.out.println("-----------------------------");
        String dayaftertommrow = "20160205";
        LocalDate formatdate = LocalDate.parse(dayaftertommrow,DateTimeFormatter.BASIC_ISO_DATE);
        System.out.println(dayaftertommrow+"格式化后的日期是"+formatdate);
        System.out.println("-----------------------------");
        String goodFriday = "04 14 2016";
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("MM dd yyyy");
        LocalDate holiday = LocalDate.parse(goodFriday,dateTimeFormatter);
        System.out.println(goodFriday+"自定义格式化后的日期是"+holiday);
        System.out.println("-----------------------------");
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MM dd yyyy HH:mm a");
        String datestr = todaytime.format(formatter);
        System.out.println("自定义格式化后的当前日期时间是"+datestr);
```



## **4.注意点**

+ Instant 它代表的是时间戳，比如2016-04-14T14:20:13.592Z，这可以从java.time.Clock类中获取，像这样： Instant current = Clock.system(ZoneId.of("Asia/Tokyo")).instant();

+ LocalDate 它表示的是不带时间的日期，比如2016-04-14。它可以用来存储生日，周年纪念日，入职日期等。

- LocalTime - 它表示的是不带日期的时间

- LocalDateTime - 它包含了时间与日期，不过没有带时区的偏移量

- ZonedDateTime - 这是一个带时区的完整时间，它根据UTC/格林威治时间来进行时区调整

- 这个库的主包是java.time，里面包含了代表日期，时间，瞬时以及持续时间的类。它有两个子package，一个是java.time.foramt，这个是什么用途就很明显了，还有一个是java.time.temporal，它能从更低层面对各个字段进行访问。

- 时区指的是地球上共享同一标准时间的地区。每个时区都有一个唯一标识符，同时还有一个地区/城市(Asia/Tokyo)的格式以及从格林威治时间开始的一个偏移时间。比如说，东京的偏移时间就是+09:00。

- OffsetDateTime类实际上包含了LocalDateTime与ZoneOffset。它用来表示一个包含格林威治时间偏移量（+/-小时：分，比如+06:00或者 -08：00）的完整的日期（年月日）及时间（时分秒，纳秒）。

- DateTimeFormatter类用于在Java中进行日期的格式化与解析。与SimpleDateFormat不同，它是不可变且线程安全的，如果需要的话，可以赋值给一个静态变量。DateTimeFormatter类提供了许多预定义的格式器，你也可以自定义自己想要的格式。当然了，根据约定，它还有一个parse()方法是用于将字符串转换成日期的，如果转换期间出现任何错误，它会抛出DateTimeParseException异常。类似的，DateFormatter类也有一个用于格式化日期的format()方法，它出错的话则会抛出DateTimeException异常。

- 再说一句，“MMM d yyyy”与“MMm dd yyyy”这两个日期格式也略有不同，前者能识别出"Jan 2 2014"与"Jan 14 2014"这两个串，而后者如果传进来的是"Jan 2 2014"则会报错，因为它期望月份处传进来的是两个字符。为了解决这个问题，在天为个位数的情况下，你得在前面补0，比如"Jan 2 2014"应该改为"Jan 02 2014"。