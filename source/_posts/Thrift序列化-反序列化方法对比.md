title: Thrift序列化/反序列化方法对比
date: 2014-11-20 20:00:52
tags: 
- Java
categories:
- Java 
---

还记得最初到公司的时候thrift序列化还是用的JSON模式，现在想想效率还是太低了。先上结论部分

结论
==
Thrift提供了（至少）三种序列化方法，Json、Binary和Compact，三者之间性能差距还是比较大的。Json方式的选取往往不是基于效率的选择，下面是两种二进制模式Binary和Compact之间的比较：

1. 序列化：compact模式压缩节省19.3%的空间，耗时节省20.4%
2. 反序列化：compact模式耗时增加3.3%

||方法 |长度|耗时|
|---|---|---|
|序列化|binary|26039|16.433
||compact|21020|13.085
||json|25096|56.137
|反序列化|binary| |8.963
||compact| |9.257
||json||61.511

测试用例
===
1. thrift定义
```thrift
struct STestObject{
     1: i64 userId;  
     2: i64 timestamp;  
     3: list<string> apps;  
     4: list<i32> pos;  
}
```

填充数据的方法
```scala
val apps = for (i <- 1 to 1000) yield "com.xiaomi.channel"
val pos = for (i <- 1 to 1000) yield new Integer(100)
val o = new STestObject(123123, d.getMillis(), apps.toList.asJava, os.toList.asJava)
```

最后序列化，反序列化都是做100,000次

具体的代码是Scala的，但是还是（完全）可以说明问题的。
```scala
import com.xiaomi.data.o2o.model.STestObject
import scala.collection.JavaConverters._
import scala.collection.JavaConversions._
import org.apache.thrift.TSerializer
import org.apache.thrift.protocol.TBinaryProtocol
import org.apache.thrift.protocol.TCompactProtocol
import org.apache.thrift.TDeserializer
import java.util.Date
import org.apache.thrift.protocol.TJSONProtocol

/**
 * @author du00
 *
 */
object ThriftSerializationTest {
  def main(args: Array[String]): Unit = {
    val d = new Date

    val apps = for (i <- 1 to 1000) yield "com.xiaomi.channel"
    val pos = for (i <- 1 to 1000) yield new Integer(100)

    val o = new STestObject(123123, d.getTime(), apps.toList.asJava, pos.toList.asJava)

    //binary
    var se = new TSerializer(new TBinaryProtocol.Factory())
    var start = new Date
    for (i <- 1 to 100000) se.serialize(o)
    var stop = new Date
    val binary = se.serialize(o)
    println("binary", binary.length, (stop.getTime() - start.getTime()) / 1000.0)

    //compact
    se = new TSerializer(new TCompactProtocol.Factory())
    start = new Date
    for (i <- 1 to 100000) se.serialize(o)
    stop = new Date
    val compact = se.serialize(o)
    println("compact", compact.length, (stop.getTime() - start.getTime()) / 1000.0)

    //json
    se = new TSerializer(new TJSONProtocol.Factory())
    start = new Date
    for (i <- 1 to 100000) se.serialize(o)
    stop = new Date
    val json = se.serialize(o)
    println("json", json.length, (stop.getTime() - start.getTime()) / 1000.0)

    //binary
    var de = new TDeserializer(new TBinaryProtocol.Factory())
    start = new Date
    for (i <- 1 to 100000) de.deserialize(o, binary)
    stop = new Date
    println("binary", (stop.getTime() - start.getTime()) / 1000.0, o)

    //compact
    de = new TDeserializer(new TCompactProtocol.Factory())
    start = new Date
    for (i <- 1 to 100000) de.deserialize(o, compact)
    stop = new Date
    println("compact", (stop.getTime() - start.getTime()) / 1000.0, o)

    //json
    de = new TDeserializer(new TJSONProtocol.Factory())
    start = new Date
    for (i <- 1 to 100000) de.deserialize(o, json)
    stop = new Date
    println("json", (stop.getTime() - start.getTime()) / 1000.0, o)
  }
}
```