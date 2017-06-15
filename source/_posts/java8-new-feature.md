---
title: Java 8 之默认方法和静态接口方法
date: 2017-02-17 18:16:32
categories:
  - Java
tags:
  - Java
  - Android
---

> 转载自[[30分钟入门Java8之默认方法和静态接口方法](http://www.cnblogs.com/JohnTsai/p/5598036.html)](http://www.cnblogs.com/JohnTsai/p/5598036.html)

## 默认方法
> 默认方法让我们能给我们的软件库的接口增加新的方法，并且能保证对使用这个接口的老版本代码的兼容性

下面通过一个简单的例子来深入理解下默认方法:

一天，PM说我们的产品需要获取时间和日期。于是我们就写了一个设置和获取日期时间的接口类 `TimeClient`

```Java
public interface TimeClient {
    void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year,
                        int hour, int minute, int second);
    LocalDateTime getLocalDateTime();
}
```

以及这个接口的实现类 `SimpleTimeClient`

```Java
public class SimpleTimeClient implements TimeClient {

    private LocalDateTime localDateTime;

    public SimpleTimeClient(){
        localDateTime = LocalDateTime.now();
    }

    @Override
    public void setTime(int hour, int minute, int second) {
        LocalTime localTime = LocalTime.of(hour, minute, second);
        LocalDate localDate = LocalDate.from(localDateTime);
        localDateTime = LocalDateTime.of(localDate,localTime);
    }

    @Override
    public void setDate(int day, int month, int year) {
        LocalDate localDate = LocalDate.of(day, month, year);
        LocalTime localTime = LocalTime.from(localDateTime);
        localDateTime = LocalDateTime.of(localDate, localTime);
    }

    @Override
    public void setDateAndTime(int day, int month, int year, int hour, int minute, int second) {
        LocalDate localDate = LocalDate.of(day, month, year);
        LocalTime localTime = LocalTime.of(hour, minute, second);
        localDateTime = LocalDateTime.of(localDate, localTime);
    }

    @Override
    public LocalDateTime getLocalDateTime() {
        return localDateTime;
    }

    @Override
    public String toString() {
        return localDateTime.toString();
    }

    public static void main(String[] args) {
        TimeClient timeClient = new SimpleTimeClient();
        System.out.println(timeClient.toString());
    }
}
```

<!-- more -->


可是PM说我们这个产品呐，不光国内用，各种其他时区的顾客也会使用。于是给你增加了新的需求：获取指定时区的日期和时间

以往我们都会这么做: 重写接口，增加方法

```Java
public interface TimeClient {
    void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year,
        int hour, int minute, int second);
    LocalDateTime getLocalDateTime(); 
    //新增的方法                          
    ZonedDateTime getZonedDateTime(String zoneString);
}
```
这样我们的实现类也要相应的进行重写。

```Java
public class SimpleTimeClient implements TimeClient {

    private LocalDateTime localDateTime;
    ...
    ZonedDateTime getZonedDateTime(String zoneString){
       return ZonedDateTime.of(getLocalDateTime(), getZoneId(zoneString));
    }
    
     static ZoneId getZoneId (String zoneString) {
        try {
            return ZoneId.of(zoneString);
        } catch (DateTimeException e) {
            System.err.println("Invalid time zone: " + zoneString +
                "; using default time zone instead.");
            return ZoneId.systemDefault();
        }
    }
    
 }
```
这样写会导致我们要去重写每个实现了 `TimeClient` 接口的类。而这大大增加了我们的实现需求的负担。

正是为了解决Java接口中只能定义抽象方法的问题。Java8新增加了默认方法的特性。下面让我们来使用默认方法实现需求。

```Java
public interface TimeClient {
    void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year,
        int hour, int minute, int second);
    LocalDateTime getLocalDateTime();                          
     static ZoneId getZoneId (String zoneString) {
        try {
            return ZoneId.of(zoneString);
        } catch (DateTimeException e) {
            System.err.println("Invalid time zone: " + zoneString +
                "; using default time zone instead.");
            return ZoneId.systemDefault();
        }
    }
    //默认方法 
    default ZonedDateTime getZonedDateTime(String zoneString) {
        return ZonedDateTime.of(getLocalDateTime(), getZoneId(zoneString));
    }
}
```
默认方法关键字为 `default`，以往我们只能在接口中定义只有声明没有实现的方法。有了默认方法，我们就能编写完整的方法。

这样我们就不需要修改继承接口的实现类，就给接口添加了新的方法实现。
```Java
public static void main(String[] args) {
        TimeClient timeClient = new SimpleTimeClient();
        System.out.println(timeClient.toString());
        System.out.println(timeClient.getZonedDateTime("test"));
    }
```
## 继承含有默认方法的接口
当我们继承含有默认方法的接口时，一般有以下三种情况
### 不去管默认方法，继承的接口直接继承默认方法
```Java
//1.不去管默认方法

public interface AnotherTimeClient  extends  TimeClient{
}
```
通过下面的测试代码，我们知道 AnotherTimeClient 接口直接继承了 TimeClient 接口的默认方法 `getZonedDateTime`
```Java
 Method[] declaredMethods = AnotherTimeClient.class.getMethods();
        for(Method method:declaredMethods){
            System.out.println(method.toString());
        }
 
//output:
//public default java.time.ZonedDateTime xyz.johntsai.lambdademo.TimeClient.getZonedDateTime(java.lang.String)
```
### 重新声明默认方法，这样会使得这个方法变成抽象方法
```Java
//重新声明默认方法，使之变为抽象方法
public interface AbstractZoneTimeClient extends TimeClient{
    @Override
    ZonedDateTime getZonedDateTime(String zoneString);
}
```
测试可以发现getZonedDateTime方法由默认方法变为了抽象方法:
```Java
Method[] methods = AbstractZoneTimeClient.class.getMethods();
        for(Method method:methods){
            System.out.println(method.toString());
        }
//output:       
//public abstract java.time.ZonedDateTime xyz.johntsai.lambdademo.AbstractZoneTimeClient.getZonedDateTime(java.lang.String)
```
### 重新定义默认方法，这样会使得方法被重写
```Java
//3.重新定义默认方法
public interface HandleInvalidZoneTimeClient extends TimeClient {
    default ZonedDateTime getZonedDateTime(String zoneString){
        try {
            return ZonedDateTime.of(getLocalDateTime(), ZoneId.of(zoneString));
        } catch (DateTimeException e) {
            System.err.println("Invalid zone ID: " + zoneString +
                    "; using the default time zone instead.");
            return ZonedDateTime.of(getLocalDateTime(),ZoneId.systemDefault());
        }
    }
}
```
实现 `HandleInvalidZoneTimeClient` 接口的类将拥有重写过的 `getZonedDateTime` 方法。
## 静态方法
在Java8的接口中，我们不光能写默认方法，还能写静态方法。上面的例子中正好用到了静态方法。
```Java
public interface TimeClient {
    // ...
    static public ZoneId getZoneId (String zoneString) {
        try {
            return ZoneId.of(zoneString);
        } catch (DateTimeException e) {
            System.err.println("Invalid time zone: " + zoneString +
                "; using default time zone instead.");
            return ZoneId.systemDefault();
        }
    }

    default public ZonedDateTime getZonedDateTime(String zoneString) {
        return ZonedDateTime.of(getLocalDateTime(), getZoneId(zoneString));
    }    
}
```