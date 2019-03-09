---
title: 谈谈Java中的时间API
date: 2019-03-09 21:50:29
author: 陈松夏
tags:
  - Java基础
categories: 
  - Java基础
---


---------------

![时区](/images/2019/02/time/时区.jpg)


--------------

<!-- more -->

### 1. 时区概念

> 国际经度会议（又称国际子午线会议）上，规定将全球划分为24个时区（东、西各12个时区）。规定英国的格林尼治天文台旧址为中时区（零时区）、东1-12区，西1-12区。每个时区横跨经度15度，时间正好是1小时。最后的东、西第12区各跨经度7.5度，以东、西经180度为界。每个时区的中央经线上的时间就是这个时区内统一采用的时间，称为区时，相邻两个时区的时间相差1小时。英国（格林尼治天文台旧址）为本初子午线，即零度经线。

时区的表格划分
![时区范围](/images/2019/02/time/时区范围.PNG)

**为什么全世界不使用统一的时间**

于各个地区所在地球位置不同，所处地球的经度不同，故其日出日落的时间也不相同。在地球上划定不同的时区，是为了使时间和自然现象（白天黑夜）有固定的对应。如果全世界用统一的时间，那么中国的8：00是早上，而美国的8：00却是晚上了。这样容易扰乱人自身生理的节律和对日常生活的安排也及其不方便。人是不能改变白天黑夜的，只好改变衡量白天黑夜的办法了。


### 2. 几种常见的时间

1. GMT
GMT(格林威治标准时间)：也就是0时区的时间，理论上来说，格林尼治标准时间的正午是指当太阳横穿格林尼治子午线时（也就是在格林尼治上空最高点时）的时间。由于地球在它的椭圆轨道里的运动速度不均匀，这个时刻可能和实际的太阳时相差16分钟。地球每天的自转是有些不规则的，而且正在缓慢减速。所以，格林尼治时间已经不再被作为标准时间使用。现在的标准时间——协调世界时（UTC）——由原子钟提供。

2.  UTC
UTC（世界协调时间），协调世界时是以原子时秒长为基础，国际原子时的准确度为每日数纳秒，而世界时的准确度为每日数毫秒，**现在我们使用的互联网就采用该计时标准**。UTC是GMT微调过的时间，我们可以认为两者是一致的。

3. CET
欧洲中部时间(CET)比世界标准时间(UTC)早一个小时，部分欧洲国家和北非国家使用，这个标准是地理加政治的产物；

4. CST
北京时间

```
    CET=UTC(GMT) + 1小时
    CST=UTC(GMT) + 8小时
```

### 3. 时间戳

> 时间戳是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的总秒数(Java中获得的秒数是以毫秒为单位的)。
> 
> 例如现在北京时间2015-12-31 17:00:00的时间戳是1451552400，就是指从北京时间1970-01-01 08:00:00到2015-12-31 17:00:00已经过去了1451552400秒。

使用时间戳有如下好处：

- 时间戳没有时区概念，比如如果用'2015-12-31 17:00:00'这么一个字符串表示时间的话，北京时间和美国时间是不一样的，但是用时间戳1451552400来表示的话，那就是一定是唯一的时间，不会有歧义；
- 时间戳在编程语言中一般是长整形数据类型，无论何种编程语言都能认识时间戳，如果用字符串表示时间，还需要转换。

### 4. Java中的时间API
在Java8以前操作时间的常见API有：

- java.util.Date:表示Java中的日期，但是能够操作到时间级别，如今这个类中的很多方法都已经被废弃，不建议使用；
- java.sql.Date:表示数据库时间，只能操作到日期，不能读取和修改时间；
- java.sql.Time:表示数据库时间；
- java.sql.Timestamp:时间戳；
- Calendar:工具类，提供时间的加减等复杂操作，支持时区；
- TimeZone:表示时区；
- SimpleDateFormat：日期格式化类，非常常用。


Date主要负责存储一个绝对时间，并对两边提供操作接口。Calendar负责对Date中特定信息，比如这个时间是该年的第几个星期，此外，还可以通过set,add,roll接口来进行日期时间的增减。SimpleDateFormat主要作为一些格式化的输入输出。

#### 4.1 Date的简单列子
Date类比较简单，支持两种构造函数。不建议用这个类进行复杂的操作。如果使用的是Java8，建议使用LocalDate。Date类也提供了和Java 8 API相互转换的接口。

```java
Date date1 = new Date();
Thread.sleep(1000);
long nowTime = System.currentTimeMillis();
Date date2 = new Date(nowTime);
//Date类和Java 8 API相互转换的接口
Instant instant = date2.toInstant();
Date date3 = Date.from(instant);

System.out.println("nowTime:"+nowTime);
System.out.println(date1);
System.out.println(date2);
System.out.println(date3);
System.out.println(date2.getTime());
```

#### 4.2 SimpleDateFormat使用列子
SimpleDateFormat接收一个String pattern和Local参数来构造对象。其中pattern是预定义的：

```
G    年代标志符
y    年
M   月
d    日
h    时 在上午或下午 (1~12)
H    时 在一天中 (0~23)
m   分
s    秒
S    毫秒
E    星期
D    一年中的第几天
F    一月中第几个星期几
w   一年中第几个星期
W   一月中第几个星期
a    上午 / 下午 标记符 
k    时 在一天中 (1~24)
K   时 在上午或下午 (0~11)
z    时区
```

```java
//默认的方言
SimpleDateFormat sf = new SimpleDateFormat("G yyyy-MM-dd a z");
SimpleDateFormat sf1 = new SimpleDateFormat("G yyyy-MM-dd a z", Locale.US);
SimpleDateFormat sf2 = new SimpleDateFormat("G yyyy-MM-dd a z", Locale.KOREA);
String format1 = sf.format(date1);
String format2 = sf1.format(date1);
String format3 = sf2.format(date1);
System.out.println("format1:"+format1);
System.out.println("format2:"+format2);
System.out.println("format3:"+format3);
//解析Date
Date date = sf.parse(format1);
```

#### 4.3 Calendar的使用
Calendar还有许多其他API，比如获得当前月天数，当前年天数等等，需要时可以查询使用。
```java
Calendar calendar = Calendar.getInstance();
int year = calendar.get(Calendar.YEAR);
// 获取月，这里需要需要月份的范围为0~11，因此获取月份的时候需要+1才是当前月份值
int month = calendar.get(Calendar.MONTH) + 1;
// 获取日
int day = calendar.get(Calendar.DAY_OF_MONTH);
// 获取时
int hour = calendar.get(Calendar.HOUR);
// 获取分
int minute = calendar.get(Calendar.MINUTE);
// 获取秒
int second = calendar.get(Calendar.SECOND);
System.out.println("现在是" + year + "年" + month + "月" + day + "日" + hour
		+ "时" + minute + "分" + second + "秒");
//获取一个月后的今天
calendar.add(Calendar.MONTH,1);
```

#### 4.4 存在的问题

- java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。
- java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
- 对于时间、时间戳、格式化以及解析，并没有一些明确定义的类。对于格式化和解析的需求，我们有java.text.DateFormat抽象类，但通常情况下，SimpleDateFormat类被用于此类需求。
- 所有的日期类都是可变的，因此他们都不是线程安全的，这是Java日期类最大的问题之一。
- 日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。

### 5. Java8中新添加的时间API

- java.time包：这是新的Java日期/时间API的基础包，所有的主要基础类都是这个包的一部分，如：LocalDate, LocalTime, LocalDateTime, Instant, Period, Duration等等。所有这些类都是不可变的和线程安全的，在绝大多数情况下，这些类能够有效地处理一些公共的需求。
- java.time.chrono包：这个包为非ISO的日历系统定义了一些泛化的API，我们可以扩展AbstractChronology类来创建自己的日历系统。
- java.time.format包：这个包包含能够格式化和解析日期时间对象的类，在绝大多数情况下，我们不应该直接使用它们，因为java.time包中相应的类已经提供了格式化和解析的方法。
- java.time.temporal包：这个包包含一些时态对象，我们可以用其找出关于日期/时间对象的某个特定日期或时间，比如说，可以找到某月的第一天或最后一天。你可以非常容易地认出这些方法，因为它们都具有“withXXX”的格式。
- java.time.zone包：这个包包含支持不同时区以及相关规则的类。

#### 5.1 LocalDate

```java
//Current Date
LocalDate today = LocalDate.now();
System.out.println("Current Date="+today);

//Creating LocalDate by providing input arguments
LocalDate firstDay_2014 = LocalDate.of(2014, Month.JANUARY, 1);
System.out.println("Specific Date="+firstDay_2014);

//Try creating date by providing invalid inputs
//LocalDate feb29_2014 = LocalDate.of(2014, Month.FEBRUARY, 29);
//Exception in thread "main" java.time.DateTimeException:
//Invalid date 'February 29' as '2014' is not a leap year

//Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
LocalDate todayKolkata = LocalDate.now(ZoneId.of("Asia/Kolkata"));
System.out.println("Current Date in IST="+todayKolkata);

//Getting date from the base date i.e 01/01/1970
LocalDate dateFromBase = LocalDate.ofEpochDay(365);
System.out.println("365th day from base date= "+dateFromBase);

//2014年的第100天
LocalDate hundredDay2014 = LocalDate.ofYearDay(2014, 100);
System.out.println("100th day of 2014="+hundredDay2014);
```

#### 5.2 LocalTime使用

```java
//Current Time
LocalTime time = LocalTime.now();
System.out.println("Current Time="+time);

//Creating LocalTime by providing input arguments
LocalTime specificTime = LocalTime.of(12,20,25,40);
System.out.println("Specific Time of Day="+specificTime);

//Try creating time by providing invalid inputs
//LocalTime invalidTime = LocalTime.of(25,20);
//Exception in thread "main" java.time.DateTimeException:
//Invalid value for HourOfDay (valid values 0 - 23): 25

//Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
LocalTime timeKolkata = LocalTime.now(ZoneId.of("Asia/Kolkata"));
System.out.println("Current Time in IST="+timeKolkata);

//java.time.zone.ZoneRulesException: Unknown time-zone ID: IST
//LocalTime todayIST = LocalTime.now(ZoneId.of("IST"));

//Getting date from the base date i.e 01/01/1970
LocalTime specificSecondTime = LocalTime.ofSecondOfDay(10000);
System.out.println("10000th second time= "+specificSecondTime);
```

#### 5.3 LocalDateTime
```java
//Current Date
LocalDateTime today = LocalDateTime.now();
System.out.println("Current DateTime="+today);

//Current Date using LocalDate and LocalTime
today = LocalDateTime.of(LocalDate.now(), LocalTime.now());
System.out.println("Current DateTime="+today);

//Creating LocalDateTime by providing input arguments
LocalDateTime specificDate = LocalDateTime.of(2014, Month.JANUARY, 1, 10, 10, 30);
System.out.println("Specific Date="+specificDate);

//Try creating date by providing invalid inputs
//LocalDateTime feb29_2014 = LocalDateTime.of(2014, Month.FEBRUARY, 28, 25,1,1);
//Exception in thread "main" java.time.DateTimeException:
//Invalid value for HourOfDay (valid values 0 - 23): 25

//Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
LocalDateTime todayKolkata = LocalDateTime.now(ZoneId.of("Asia/Kolkata"));
System.out.println("Current Date in IST="+todayKolkata);

//java.time.zone.ZoneRulesException: Unknown time-zone ID: IST
//LocalDateTime todayIST = LocalDateTime.now(ZoneId.of("IST"));

//Getting date from the base date i.e 01/01/1970
LocalDateTime dateFromBase = LocalDateTime.ofEpochSecond(10000, 0, ZoneOffset.UTC);
System.out.println("10000th second time from 01/01/1970= "+dateFromBase);
```
#### 5.4 Instant
Instant可以理解为时间戳。

```java
//Current timestamp
Instant timestamp = Instant.now();
System.out.println("Current Timestamp = "+timestamp);

//Instant from timestamp
Instant specificTime = Instant.ofEpochMilli(timestamp.toEpochMilli());
System.out.println("Specific Time = "+specificTime);

//Duration example
Duration thirtyDay = Duration.ofDays(30);
System.out.println(thirtyDay);
```

#### 5.5 格式化
需要格式化日期时，我们直接调用相关类的format方法即可，如下：

```java
//DateTimeFormatter支持的模式和SimpleDateFormat支持的一致
today.format(DateTimeFormatter.ofPattern("yyyy年MM月dd日"));
```

#### 5.6 对旧时间API的支持
```java
    Date date= new Date();Instant instant = date.toInstant();LocalDateTime pst = LocalDateTime.ofInstant(instant, 
    ZoneId.of(ZoneId.SHORT_IDS.get("PST")));Calendar calendar = Calendar.getInstance();Instant instant1 = calendar.toInstant();LocalDateTime pst1 = LocalDateTime.ofInstant(instant, 
    ZoneId.of(ZoneId.SHORT_IDS.get("PST")));
```

### 6. 在东八区的机器上获得美国时间
美国属于西5区，比北京时间晚13个小时。