---
layout: post
title: "scala how to"
date: "2014-12-15 10:59"
---

##joda-time编译报错
```
[WARNING] warning: Class org.joda.convert.ToString not found - continuing with a stub.
[WARNING] warning: Caught: java.lang.NullPointerException while parsing annotations in /Users/du00/.m2/repository/joda-time/joda-time/2.6/joda-time-2.6.jar(org/joda/time/base/AbstractInstant.class)
[ERROR] error: error while loading AbstractInstant, class file '/Users/du00/.m2/repository/joda-time/joda-time/2.6/joda-time-2.6.jar(org/joda/time/base/AbstractInstant.class)' is broken
[INFO] (class java.lang.RuntimeException/bad constant pool tag 10 at byte 10)
```
提示里面已经说了convert找不到，增加convert依赖（[参考](http://stackoverflow.com/questions/13856266/class-broken-error-with-joda-time-using-scala)）
```xml
<dependency>
  <groupId>org.joda</groupId>
  <artifactId>joda-convert</artifactId>
  <version>1.6</version>
</dependency>
```

## Scala Int => Java Integer
转换很简单，但是不知道就很要命。
```scala
i:Int => i:java.lang.Integer
111
```
