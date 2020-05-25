---
title: Core Java&#58 Date and Time
author: Hamza Belmellouki
categories: [Java]
tags: [core-java]
---

## Introduction

Greetings!

Since Java 8, Oracle completely rebuilt its Date/Time API. The new API is supposed to replace the old one.

The new API which is located in the `java.time` package is thread-safe because most of the new classes are **immutable**, meaning that, after the object is constructed, it cannot be modified. This is especially useful when working in a multi-threaded environment where issues like thread interference and data corruption cannot happen thanks to immutability.

This article shows how to work with the new API.

## Creating Dates and Times

Java lets us create dates and times using static factory methods. Note that you cannot create date and time objects using a constructor (you can do so with the old API, but you shouldn’t) because it is made private.

Creating dates and times is straightforward; you’ll notice a pattern for creating dates and times:

### LocalDate

You create a `LocalDate` by using one of its static factory methods:

```java
    LocalDate now = LocalDate.now();
    System.out.println(now);// 2020-02-21
    
    LocalDate date = LocalDate.of(2020, 1, 23);
    System.out.println(date);// 2020-01-23
    
    LocalDate date2 = LocalDate.of(2020, Month.JANUARY, 23);
    System.out.println(date2);// 2020-01-23
```

Note that month indexes are one-based.

### LocalTime

Similarly, you create a `LocalTime` object like so:

```java
    LocalTime now = LocalTime.now();
    System.out.println(now);// 12:31:56.186
    
    LocalTime midnight = LocalTime.of(23, 0);
    System.out.println(midnight); // 23:00
```

The second line output a `toString` representation of `LocalTime`, which represents `[hour:minutes:seconds.milliseconds]`

### LocalDateTime

The `LocalDateTime` class represents date and time combined. You can create it with:

```java
    LocalDateTime now = LocalDateTime.now();
    System.out.println(now);// 2020-02-21T12:46:09.950
    
    LocalDateTime newYear = LocalDateTime.of(LocalDate.of(2021, 12, 31), LocalTime.of(11, 59));
    System.out.println(newYear);// 2021-12-31T11:59
```

Note that in the output date and time are separated with a `T`.

### ZonedDateTime

Use this class if you want to express in a date and time in a specific timezone. for example:

```java
    ZonedDateTime usPacific = ZonedDateTime.now(ZoneId.of("US/Pacific"));
    System.out.println(usPacific);// 2020-02-22T02:53:46.774455-08:00[US/Pacific]
    
    ZonedDateTime now = ZonedDateTime.now();// 2020-02-22T11:53:46.778363+01:00[Africa/Casablanca]
    System.out.println(now);
```

The format of the output consists of `LocalDateTime` followed by the `ZoneOffset`.

## Manipulating Dates and Times
```java
    LocalDate date = LocalDate.of(2020, Month.JANUARY, 20);
    System.out.println(date);// 2020–01–20
    
    date = date.plusDays(1);
    System.out.println(date);// 2020-01-21
    
    date = date.plusWeeks(3);
    System.out.println(date);// 2020-02-11
    
    date = date.plusMonths(4);
    System.out.println(date);// 2020-06-11
    
    date = date.plusYears(10);
    System.out.println(date);// 2030-06-11
```

You these methods return a `LocalDate` object. Therefore, you can chain them:

```java
    LocalDate date = LocalDate.of(2020, Month.JANUARY, 20)
            .plusDays(1).plusWeeks(3)
            .plusMonths(4).plusYears(10);
    System.out.println(date);// 2030-06-11
```

Using the same pattern, you can subtract dates/times from `LocalDate`, `LocalDateTime`, `LocalTime` and `ZonedDateTime` using `minus###()` method.

## Periods and Durations

### Periods

You create a period from the `Period` class. This class represents the amount of time in years, months, and days. These examples demonstrate the typical ways you would create a `Period`:

```java
    Period threeDays = Period.ofDays(3);
    System.out.println(threeDays);// P3D
    
    Period threeWeeks = Period.ofWeeks(3);
    System.out.println(threeWeeks);// P21D
    
    Period threeMonths = Period.ofMonths(3);
    System.out.println(threeMonths);// P3M
    
    Period threeYears = Period.ofYears(3);
    System.out.println(threeYears);// P3Y
    
    Period threeYearsAndFourMonthsAndTwoDays = Period.of(3, 4, 2);
    System.out.println(threeYearsAndFourMonthsAndTwoDays);// P3Y4M2D
```

These static factory methods are self-explanatory; they create an immutable `Period` instance.

In the output, the letter `P` stands for Period, `Y` for years, `M` for months, and `D` for days.

Note that you cannot chain methods as you’ve seen in the `LocalDate` example when you create `Period` because these methods are static, If you chain them you’ll get unexpected behavior:

```java
    Period oneWeekAndADay = Period.ofDays(1).ofWeeks(1);
    System.out.println(oneWeekAndADay); // unexpected result: P7D
```

Remember that a `Period` cannot be be used with some objects. Let’s look at some code:

```java
    Period period = Period.ofDays(1);
    LocalTime time = LocalTime.of(6, 15);
    time.plus(period); // UnsupportedTemporalTypeException
```
### Durations

You create a duration form the `Duration` class. This class represents the amount of time in seconds and nanoseconds. It can also be expressed using other duration-based units, such as minutes and hours. These examples demonstrate the typical ways you would create a `Duration`:
```java
    Duration oneNano = Duration.ofNanos(1);
    System.out.println(oneNano);// PT0.000000001S
    
    Duration oneMilli = Duration.ofMillis(1);
    System.out.println(oneMilli);// PT0.001S
    
    Duration oneSeconds = Duration.ofSeconds(1);
    System.out.println(oneSeconds);// PT1S
    
    Duration oneMinute = Duration.ofMinutes(1);
    System.out.println(oneMinute);// PT1M
    
    Duration oneHour = Duration.ofHours(1);
    System.out.println(oneHour);// PT1H
    
    Duration oneDay = Duration.ofDays(1);
    System.out.println(oneDay);// PT24H
```

Alternatively, you can create a `Duration` using the following method:
```java
    Duration fiveHours = Duration.of(5, ChronoUnit.HOURS);
    System.out.println(fiveHours);
```
This method takes 5 as an amount and a unit that the duration is measured in.

## Working with Instants

You create an instant from the `Instant` class. This class represents a single instantaneous point on the time-line in the GMT since January 1, 1970 (1970–01–01T00:00:00Z), a.k.a the [EPOCH](https://en.wikipedia.org/wiki/Epoch_(computing)). It may come in handy when you want to record event timestamps in the program. These examples demonstrate the typical ways you would create an `Instant`:

```java
    Instant now = Instant.now();
    System.out.println(now);//2020-02-25T11:14:46.032856Z
    
    ZonedDateTime zonedDateTime = ZonedDateTime.now(ZoneId.of("US/Eastern"));
    System.out.println(zonedDateTime);// 2020-02-25T06:27:27.572624-05:00[US/Eastern]
    Instant now2 = Instant.from(zonedDateTime);
    System.out.println(now2);// 2020-02-25T11:14:46.055857Z
    
    Instant instant = Instant.parse("2010-01-20T11:33:45Z");
    System.out.println(instant);// 2010-01-20T11:33:45Z
    
    Instant epoch = Instant.ofEpochMilli(0);
    System.out.println(epoch);// 1970-01-01T00:00:00Z
```

Note that the output of `toString` follows the [ISO-8601](http://www.iso.org/iso/home/standards/iso8601.htm) standard.

As you can see, when Java invoked `Instant.from(zonedDateTime);` it created an `Instant` object from `ZonedDateTime` object and converted the time from US/Eastern timezone to GMT.

Likewise, this class provides various ways to operate on instants. For example:
```java
    Instant tenMinutesLater = Instant.now().plus(10, ChronoUnit.MINUTES);
    System.out.println(tenMinutesLater);// print 10 minutes later from the current time
```
## Parsing and Formatting

### Formating

The JDK provides a new API to parse and format Temporal-based objects, using the `DateTimeFormatter` from the `java.time.format` package we can parse and format dates and times. Similar to most other new Date/Time API classes, `DateTimeFormatter` is immutable thus thread-safe.

The `format` method is provided by those classes for formatting temporal-based objects for display. For example, this snippet of code format date and time using a **predefined formatter**:

```java
    LocalDate date = LocalDate.of(2020, Month.MARCH, 17);
    LocalTime time = LocalTime.of(9, 15, 45);
    LocalDateTime dateTime = LocalDateTime.of(date, time);
    
    System.out.println(
    dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));// 2020-03-17T09:15:45
    
    System.out.println(
    date.format(DateTimeFormatter.ISO_LOCAL_DATE));// 2020-03-17
    
    System.out.println(
    time.format(DateTimeFormatter.ISO_LOCAL_TIME));// 09:15:45
    
    DateTimeFormatter shortF = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT);
    System.out.println(shortF.format(dateTime));// 3/17/20, 9:15 AM
    
    DateTimeFormatter mediumF = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM);
    System.out.println(mediumF.format(dateTime));// Mar 17, 2020, 9:15:45 AM
```

We can also define a custom formatter object using `DateTimeFormatter.ofPattern` method:

```java
    LocalDate date = LocalDate.of(2020, Month.MARCH, 17);
    LocalTime time = LocalTime.of(9, 15, 45);
    LocalDateTime dateTime = LocalDateTime.of(date, time);
    
    DateTimeFormatter f = DateTimeFormatter.ofPattern("dd-MMMM-yyyy | hh:mm");
    System.out.println(dateTime.format(f));// 17-March-2020 | 09:15
```

Make sure to take a look at the [reference documentation](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#predefined) if you want to know more about the syntax used in the `ofPattern` method argument.

### Parsing

Now you know how to convert Temporal-based classes into strings, let’s see how we can convert into the other direction using the `parse` method:

```java
    DateTimeFormatter f = DateTimeFormatter.ofPattern("dd MM yyyy");
    LocalDate date = LocalDate.parse("25 03 2020", f);
    LocalTime time = LocalTime.parse("09:30");
    
    System.out.println(date);// 2020-03-25
    System.out.println(time);// 09:30
```

`LocalDate.parse(“25 03 2020“, f)` returns a `LocalDate` by parsing the text string using a custom formatter object. While `LocalTime.parse(“09:30”)` parses the text string and returns a `LocalTime` using the default formatter ([DateTimeFormatter.ISO_LOCAL_TIME](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_LOCAL_TIME))

## A Peek on Dates and Times Before Java 8

Before Java 8, developers used the `Date` class, which represents the date and time altogether. There was no way to get a date or time separately, and it was all bundled in the `Date` class. Developers should not use this class anymore, as you’ve seen, there is a better way. This class exists to support backward compatibility.

Another disadvantage of the `Date` class is that it’s **mutable**. Therefore, you need to synchronize access to instances of this class when they’re accessed from multiple threads to avoid data corruption and unexpected behavior.

You may encounter the old Date API in legacy projects. You create a date by calling its constructor: `Date date = new Date()` , `date` refers to the current date/time. You can specify a specific date using the `Calendar` class:

```java
    Calendar cal = Calendar.getInstance();
    cal.set(2020, 0, 23);
    Date d = cal.getTime();
    System.out.println(d);// Wed Jan 01 11:36:37 WET 2020
```

You can see how much verbose this old API compared to the new one. Beware that Month indexes are zero-based instead of one-based, which is confusing. **The new API’s indexes are one-based**.

## Wrap Up

In this post, I attempted to demonstrate why you should work with the new Date/Time API, how to work with it, how to format and parse dates and times. And, peeked at how dates/times were handled before Java 8.

If you found this post useful, you know what to do now. Hit that clap button and follow me to get more articles and tutorials on your feed.
