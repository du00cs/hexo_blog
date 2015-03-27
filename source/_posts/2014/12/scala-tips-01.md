---
layout: post
title: Scala点点滴滴-JSON/正则/命令行解析
date: "2014-12-01 17:11"
---

# 1. JSON处理包
正如[JSON4S](https://github.com/json4s/json4s)官网所说，现在已经有6个Scala的JSON解析库了，为什么要使用这一个呢？——快速，简单！JSON4S可以将字符串解析成对象、容器，什么复杂的就免了，我只想学最简单的抽成Map的方法，其它的就交给我自己来处理好了。
## 1.1 依赖  
　　如果是使用的Spark，它的依赖中已经有json4s这个包了，无需再添加。如果没有可以在maven中添加：
```xml
<dependency>
  <groupId>org.json4s</groupId>
  <artifactId>json4s-native_${scala.version}</artifactId>
  <version>3.2.11</version>
</dependency>
<dependency>
  <groupId>org.json4s</groupId>
  <artifactId>json4s-jackson_${scala.version}</artifactId>
  <version>3.2.11</version>
</dependency>
```
## 1.2 样例
　　我最喜欢的例子（我也只用到了这个）
```scala
import org.json4s._
import org.json4s.jackson.JsonMethods._ //下面只用到了其中的parse方法

implicit val formats = DefaultFormats //不加这一句会提示formats找不到，并且还提示了将org.json4s.DefaultFormats提到前面
val s = """{"a":"","b":"mobile","c":"dior","d":"stable","e":"4.4.2"}"""
val deviceInfo = parse(s, false).extract[Map[String, String]]
println(deviceInfo)
// Map(e -> 4.4.2, a -> , b -> mobile, c -> dior, d -> stable)
```

　　官方文档中还有个抽取成对象的例子，各取所需吧～（反正我只需要上面的……）

```scala
import org.json4s._
import org.json4s.jackson.JsonMethods._
implicit val formats = DefaultFormats // Brings in default date formats etc.
case class Child(name: String, age: Int, birthdate: Option[java.util.Date])
case class Address(street: String, city: String)
case class Person(name: String, address: Address, children: List[Child])
val json = parse("""
{ "name": "joe",
"address": {
  "street": "Bulevard",
  "city": "Helsinki"
  },
  "children": [
  {
    "name": "Mary",
    "age": 5,
    "birthdate": "2004-09-04T18:06:22Z"
    },
    {
      "name": "Mazy",
      "age": 3
    }
    ]
  }
  """)
json.extract[Person]
// res0: Person =Person(joe,Address(Bulevard,Helsinki),List(Child(Mary,5,Some(Sat Sep 04 18:06:22 EEST 2004)), Child(Mazy,3,None)))
```
　　文档中还有很多丰富的内容，各取所需各取所需。

# 2. 正则表达式
　　scala中的正则表达式还是很有爱的——简单，才有爱……不需要引入什么包，直接上例子：
```scala
val nonHttpPattern = """^[^(http)].*""".r //"""可以避免转义
val ipAccessPattern = """^https{0,1}://\d+\.\d+\.\d+.*""".r
val downloadPattern = """.*\.(apk|exe|zip|rar)""".r //主要是为了过滤apk

def isUrlValid(url: String): Int = {
  url match {
    case nonHttpPattern(_*)  => 4
    case ipAccessPattern(_*) => 5
    case downloadPattern(_*) => 6
    case _                  => 0
  }
}

val date = """(\d\d\d\d)-(\d\d)-(\d\d)""".r
"2004-01-20" match {
  case date(year, month, day) => s"$year was a good year for PLs."
} // case可以匹配出分组，还是很强大的
```

# 3. 命令行解析
　　首先，如果你已经熟悉一个java版的并且没有时间（不想）学，那直接用就好了。其次，如果时间足够可以去挨个挨个比较比较scala各种各样的命令行解析包。这里只介绍scopt，scopt确实是很简洁以及简单的，直接上例子了：
```scala
//所有参数都需要有默认值，这样才能无参初始化一个实例
case class Config(logBase: String = ".",
  hist: String = ".",
  hostFilter: String = ".",
  maxRecordPerUser: Int = 200,
  minHostAccess: Int = 100,
  decay: Double = 0.95f,
  day: DateTime = null, //joda datetime
  partitions: Int = 80)

val parser = new scopt.OptionParser[Config]("run") {
  head("User Host Access Accumulator", "")
  //每个少写的空格其实都相当于一个"."
  opt[String]('i', "input") required () valueName ("<file>") action { (x, c) ⇒
    c.copy(logBase = x)
  } text ("input - hdfs路径，浏览器行为基础日志")
  //'h'是对应-h，"history"对应的是--history，还是很好理解的
  opt[String]('h', "history") required () valueName ("<file>") action { (x, c) ⇒
    c.copy(hist = x)
  } text ("input - hdfs路径，浏览器访问累积日志")
  opt[String]('f', "host-filter") required () valueName ("<file>") action { (x, c) ⇒
    c.copy(hostFilter = x)
  } text ("output - hdfs路径，host过滤文件")
  opt[String]('d', "day") required () valueName ("<date>") action { (x, c) ⇒
    c.copy(day = new DateTime(x)) //不只是能copy，简单的处理逻辑也是可以有的！
  } text ("date - yyyy-MM-dd日期")
  opt[Int]("max-record-per-user") optional () action {
    (x, c) ⇒ c.copy(maxRecordPerUser = x)
  } text ("每个用户记录访问频次的host最大数量")
  opt[Int]("min-host-access-limit") optional () action {
    (x, c) ⇒ c.copy(minHostAccess = x)
  } text ("全局host访问频次最低过滤条件")
  opt[Int]("decay") optional () action {
    (x, c) ⇒ c.copy(decay = x)
  } text ("访问频次衰减因子")
  opt[Int]("partitions") optional () action {
    (x, c) ⇒ c.copy(partitions = x)
  } text ("保存文件块的数量")
}
```
上面是解析部分，调用见下：
```scala
def main(args: Array[String]): Unit = {
  parser.parse(args, Config()) map { config ⇒
    //config已经获取到了
  } getOrElse {
    //额外的错误处理逻辑，默认会把命令行帮助打印出来
  }
}
```

[scopt](https://github.com/scopt/scopt)在github的官网上还有很复杂的例子，我第一次就是被它吓住了。scopt使用时的maven坐标
```xml
<dependency>
  <groupId>com.github.scopt</groupId>
  <artifactId>scopt_2.10</artifactId>
  <version>3.2.0</version>
</dependency>
```
最后看一下scopt给我们准备的精美帮助
```
User Host Access Accumulator
Usage: run [options]

-i <file> | --input <file>
    input - hdfs路径，浏览器行为基础日志
-h <file> | --history <file>
    input - hdfs路径，浏览器访问累积日志
-f <file> | --host-filter <file>
    output - hdfs路径，host过滤文件
-d <date> | --day <date>
    date - yyyy-MM-dd日期
--max-record-per-user <value>
    每个用户记录访问频次的host最大数量
--min-host-access-limit <value>
    全局host访问频次最低过滤条件
--decay <value>
    访问频次衰减因子
--partitions <value>
    保存文件块的数量
```
如果参数有错误会先打出来，再打这段帮助，错误信息是类似于这样的：
```
Error: Unknown option -o
Error: Unknown argument 'ojjljlkl'
Error: Missing option --host-filter
Error: Missing option --day
```
