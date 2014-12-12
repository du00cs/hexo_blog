title: Spark/Scala极速入门材料
date: 2014-11-11 15:09:26
tags:
- Spark
- Scala
categories:
- Spark
---
Spark/Scala的一点入门材料，希望能对想快速了解的人有所帮助，对自己则是一个备忘。
## Scala极速入门
如果需要一些感性认识，可以先读一读Scala的[官方介绍](http://www.scala-lang.org/what-is-scala.html)。简单地说，Scala是一种更偏函数式的函数式、命令式的混合编程语言，同时也是面向对象的，函数是顶级对象，运行在Java虚拟机上，能与Java无缝结合。

```scala
package com.xiaomi.data.ctr.feature.analysis

/**
 * Scala极速入门材料，可以直接贴入ScalaIDE的worksheet
 */
object test {
  println("Welcome to the Scala worksheet")       //> Welcome to the Scala worksheet
  //val(ue) 是引用不变，不能改变val值变量的『值』
  val n = 8                                       //> n  : Int = 8
  //n += 1 : value += is not a member of Int

  //var(riable) 是变量，能用val用val，技穷用var
  var nn = 7                                      //> nn  : Int = 7
  nn += 1
  nn                                              //> res0: Int = 8

  //tuple - 使用得非常重的数据结构，同python，但是不能（显式地）按下标取到每个元素
  val t = (1, "a", None)                          //> t  : (Int, String, None.type) = (1,a,None)
  t._1                                            //> res1: Int = 1
  t._2                                            //> res2: String = a
  t._3                                            //> res3: None.type = None
  //纯语法，相信看过就不会忘记的
  val (no, name, score) = t                       //> no  : Int = 1
                                                  //| name  : String = a
                                                  //| score  : None.type = None

  //collections, 取下标用()
  val list = List(1, 2, 3)                        //> list  : List[Int] = List(1, 2, 3)
  list(0)                                         //> res4: Int = 1
  val m = Map(
    "a" -> 1,
    "c" -> 2,
    "b" -> 3,
    "d" -> 4)                                     //> m  : scala.collection.immutable.Map[String,Int] = Map(a -> 1, c -> 2, b -> 3
                                                  //| , d -> 4)
  //取前两个
  m.take(2)                                       //> res5: scala.collection.immutable.Map[String,Int] = Map(a -> 1, c -> 2)
  m("a")                                          //> res6: Int = 1
  //删除一个键，得到一个新的map
  m - "a"                                         //> res7: scala.collection.immutable.Map[String,Int] = Map(c -> 2, b -> 3, d ->
                                                  //| 4)

  //函数：变量名在前、类型在后，函数头到函数体之间有“=”号
  def isPalindrome(str: String) = (str == str.reverse.toString())
                                                  //> isPalindrome: (str: String)Boolean
  //函数：可以显式指定返回值类型，函数的返回值就是最后一行的值
  def isPalindromeDetail(str: String): Boolean = {
    println(str)
    str == str.reverse.toString
  }                                               //> isPalindromeDetail: (str: String)Boolean

  def isPalindromeDetailUn(str: String): Boolean = ???
                                                  //> isPalindromeDetailUn: (str: String)Boolean
  //函数式编程：map/filter/reduce，其它的基本都是变种
  val ab = List(1, 2, 3, 4, 5, 6)                 //> ab  : List[Int] = List(1, 2, 3, 4, 5, 6)
  //每个元素乘3
  ab.map(_ * 3)                                   //> res8: List[Int] = List(3, 6, 9, 12, 15, 18)
  //取出偶数
  ab.filter(_ % 2 == 0)                           //> res9: List[Int] = List(2, 4, 6)
  //reduce实现sum
  ab.reduce(_ + _)                                //> res10: Int = 21
  //reduce实现max
  ab.reduce((x, y) => if (x > y) x else y)        //> res11: Int = 6
}
```

一种是“对集合中的每个东西，东西在哪儿，取出来，执行某个操作”，另一种是“对一个集合中的每一个元素执行操作”。
具体的函数式编程与命令式编程语言的区别网上铺天盖地的，推荐一篇精简（是否得当就不评论了）的介绍（我自己参考写的……）《[课程学习Couresera - Functional Programming Principles in Scala - Week 1](/2014/11/11/课程学习Couresera-Functional-Programming-Principles-in-Scala-Week-1)》

## Scala相关资料
- 以后最常用的是会是这个：[Scala Standard Library API Docs](http://www.scala-lang.org/api/current/#package)，也许你会觉得看完也不知道毡哪个，但是你确实得依赖它。
- 书籍：多看上面是有本书的——《[Scala程序设计：Java虚拟机多核编程实战](http://www.duokan.com/book/68639)》，其它的还有很多，如《**Scala for the Impatient**》，《**Programming in Scala: A comprehensive Step-by-step Guide**》
- [Coursera](https://class.coursera.org)上有一门用Scala讲的函数式编程语言的课——*Functional Programming Principles in Scala*，需要注意的是可能从头学到尾都不知道还有`var`这个东西，因为这门课真的只讲函数式编程。另外，请不要惊讶做作业需要花很长时间。
如果你是跟我一样的懒人，还是去Coursera上面上一课吧，系统地学一学对整体把握有好处。
- 方方同学补充： [typesafe activetor](http://www.typesafe.com/)上有不少代码模板，Twitter内部大量使用Scala并且开办了[Scala School](https://twitter.github.io/scala_school/index.html)

## Spark入门
![RDD操作](http://du00.qiniudn.com/网页配图-RDD操作.png)
Spark简单说来就只是三步：Create (RDD)，Transform (RDD)和Action(非RDD)
1. Create
常见的创建方式有三种（均是从一个SparkContext实例开始）：
 - textFile
 - sequenceFile
 - parallize，从一个Scala Collection开始
textFile/sequenceFile两个方法已经可以解决来自HDFS的所有类型的记录文件，parallize用于测试，读hbase等其它的，略麻烦，不是一条语句能搞定了。
2. Transform
 - 基本的：map/filter
 - 处理key-value：groupByKey, reduceByKey, combineByKey等等
这部分必须熟练掌握，PairRDDFunctions在`import org.apache.spark.SparkContext._`过后就可以自动的给类型是`RDD[(K, V)]`的rdd加上PairRDDFunctions里面的所有方法了。注意：Transform的输入是RDD，输出仍然是RDD。
3. Action
Action会完成RDD向基本数据类型的转换，结果不再是RDD，一般来说就是收集结果到driver结点或者直接写HDFS了。作为Hadoop的用户会一定要会使用`saveAsTextFile`和`saveAsSequeceFile`，收集结果用collect()或者reduce()到driver结点完成或其它操作。

## Spark工程
建一个maven工程，pom里面写上
```xml
<dependencies>
  <dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>2.4.0-mdh2.0.5</version>
    <type>jar</type>
    <scope>compile</scope>
    <exclusions>
      <exclusion>
        <groupId>asm</groupId>
        <artifactId>asm</artifactId>
      </exclusion>
      <exclusion>
        <groupId>org.jboss.netty</groupId>
        <artifactId>netty</artifactId>
      </exclusion>
      <exclusion>
        <artifactId>servlet-api</artifactId>
        <groupId>javax.servlet</groupId>
      </exclusion>
    </exclusions>
  </dependency>
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.10</artifactId>
    <version>1.1.0</version>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.8.1</version>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>org.scalatest</groupId>
    <artifactId>scalatest_2.10</artifactId>
    <version>2.2.1</version>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.4</version>
  </dependency>
</dependencies>

<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-shade-plugin</artifactId>
      <version>2.3</version>
      <configuration>
        <artifactSet>
          <includes>
            <include></include>
          </includes>
        </artifactSet>
      </configuration>

      <executions>
        <execution>
          <phase>package</phase>
          <goals>
            <goal>shade</goal>
          </goals>
        </execution>
      </executions>
    </plugin>

    <!-- maven的scala支持插件，适当的时候可以去用一用新的版本 -->
    <plugin>
      <groupId>net.alchim31.maven</groupId>
      <artifactId>scala-maven-plugin</artifactId>
      <version>3.1.3</version>
      <executions>
        <execution>
          <goals>
            <goal>compile</goal>
            <goal>testCompile</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

下面是一个代码实例，完成的工作是将格式为
`label    pos1:value1 pos2:value2 ... post:valuen#imei,appid`
的样本文件中去掉`0, 37, 38, 39, 51`五列的值。
```scala
package demo

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import com.xiaomi.data.ctr.feature.analysis.AppstoreRecordParser.getSampleRdd
import org.apache.hadoop.io.compress.GzipCodec

/**
 * 从base中去除 0, 37, 38, 39, 51五维特征
 */
object SampleReduction {
  /**
   * 定义一个用于解析出来的记录的载体类，为什么这样就可以了？语法
   */
  class AppStoreRecord(
    val imei: String,
    val appid: Int,
    val label: Int,
    val pv: Map[Int, Double])

  /**
   * 解析一条记录的函数
   */
  def parseRecord(line: String): AppStoreRecord = {
    val cols = line.split("\t")
    val label = cols(0).toInt
    val splits = cols(1).split("#")
    val Array(imei, appId) = splits(1).split(",")
    val pvPart = splits(0).split(" ")
    splits(1).split(",")(0)
    val values = pvPart.map(cell => { val t = cell.split(":"); (t(0).toInt, t(1).toDouble) })
    new AppStoreRecord(imei, appId.toInt, label, values.toMap)
  }

  /**
   * 需要有这个函数来成为入口，同java的main函数，格式固定
   */
  def main(args: Array[String]): Unit = {
    //固定，不可少
    val sparkConf = new SparkConf().setAppName("Sample Reduction")
    //外部执行spark程序时，master会指定，这里手工指定方便在IDE中跑起来
    if (!sparkConf.contains("spark.master")) sparkConf.setMaster("local[4]")
    println("master: " + sparkConf.get("spark.master"))
    //固定，必有这么一句
    val sc = new SparkContext(sparkConf)

    val zeroFeatures = Array(0, 37, 38, 39, 51)

    sc
      //create RDD
      .textFile(args(0))
      //transform 1
      .map(parseRecord)
      //transform 2
      //保存为label[tab]pos:value[space]...#imei,appid
      .map(t => {
        t.label + "\t" + (t.pv -- zeroFeatures).map { case (p, v) => p + ":" + v }.mkString(" ") + "#" + t.imei + "," + t.appid
      })
      //action
      .saveAsTextFile(args(1), classOf[GzipCodec])

    // spark自带的例子里面都有这么一句
    sc.stop
  }
}
```

IDE之类的操作训不在讲解范围之内了，打包好之后用以下语句执行即可（主jar包后而一定只有程序参数，或其它参数都向前站）：
```bash
spark-submit \
    --master yarn-client \
    --num-executors 3 \
    --executor-memory 2G \
    --queue user_profile_default \
    --class com.your.mainclass \
    target/feature-analysis-0.0.1-SNAPSHOT.jar \
    各种参数
```
可以在工程下而建一个脚本来省去一些公共参数的填写
```bash
#!/bin/bash

# 用于提交任务至yarn的脚本，参数：
# class 参数1...

class=$1
shift

spark-submit \
    --master yarn-client \
    --num-executors 3 \
    --executor-memory 2G \
    --queue user_profile_default \
    --class $class \
    target/feature-analysis-0.0.1-SNAPSHOT.jar \
    "$@"
```

## Spark相关资料
- Spark官网材料，简易的[Programming Guide](https://spark.apache.org/docs/latest/programming-guide.html)，以及以后你经常会访问的[API Docs](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.package)，将来会看很多很多遍，但是至少需要基本掌握[SparkContext](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkContext)、[RDD](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.RDD)和[PairRDDFunctions](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions)这三个类，你会发现处理日志基本就够用了~
- 公开课: *开课吧*上而有这么一门课，《[Spark实战演练](http://www.kaikeba.com/courses/60)》，而且还是免费的，虽然授课老师表情、动作、语气和语调都不丰富，但是毕竟是免费的嘛~我不会告诉你上而还有度娘的office系列课程的。
- 电子书: 《[Learning Spark](https://www.safaribooksonline.com/library/view/learning-spark/9781449359034/)》Safari上有前几章的在线观看，也可以去下个PDF，我找到的只有95页……
- 演示文档: [SlideShare](http://www.slideshare.net/)上面一大堆一大堆的，不过就别指望能给你实在的一步一步怎么做了。需要翻墙，还是希望国人在国外的网站上还是要自律，自己嘴上一时爽却妨碍了其它人访问有用的资源。
如果你跟我一样是个懒人，还是去*开课吧*上面上一课吧，不要觉得我是在打广告，我是真的懒……

## 小结
　　Spark结合Scala学习起来会有一定成本，但是对于尔等程序员一辈子要学习数十上百种语言人，一定是毫无压力的！一个人可以维护的代码量是有限的，用Spark/Scala确实是可以把自己从大段大段的Java重复代码中解脱出来。当然了，熟练才能解脱……你需要坚持到把Scala那些奇怪的东西消化掉，Google/AOL是好朋友，好好利用。
