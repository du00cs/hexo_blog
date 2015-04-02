---
layout: post
title: Spark问题备忘
date: "2014-12-12 17:46"
tags:
- Spark
- Scala
categories:
- Spark
---

　　想当年被mapreduce虐得死去活来，换上Spark其实是接着虐，记录在此仅作备忘。

### 1. Spark/Scala在Eclipse中的设置
　　IntelliJ IDEA也有相应插件，在此不提。基于Eclipse的方案中可以安装ScalaIDE的插件，但还是建议使用ScalaIDE，而且是[Milestone](http://scala-ide.org/download/milestone.html)版本。IDE的新版本不同于硬件驱动，往往是添加了更好的新功能，不需要坚持用一个老版本，况且开源的东西对老版本的东西感觉好像不维护的样子。  

　　scala工程在Eclipse中有一些基本上逃避不了的问题：  
1. 首先，**怎么创建一个scala的maven工程？**  
　　参考[Spark/Scala的入门材料](/2014/11/11/2014-11-11-spark-scala-introduction/)，建一个新工程，在pom中配置scala的maven编译插件即可。
2. **引入一个scala的maven工程后需要哪些设置？**  
情况可能各有不同，一条一条对照检查即可：
 - 未识别出工程为Scala工程(工程文件夹图标有M、J标记，没有S)——需要**添加工程的Scala特性**  
`工程->右键->Configure->Add Scala Nature`
 - 未能识别scala代码（没有变成java的package管理）——添加Scala代码路径为Source Folder  
比如src/main/scala文件夹下有scala源文件——`scala代码文件夹-> 右键 ->Build Path -> Use as Source Folder`
 - 可能的兼容性问题：修改JDK兼容版本从1.5到1.6——`JRE System Library -> 右键 -> Properties -> 选择Java SE-1.6`(比如thrift需要至少JDK1.6支持，如果没有飘红忽略亦可)
3. **Scala版本的依赖版本冲突**  
　　在Problems的View中可能会有红色的错误提示说xx.jar是用2.10编译的，而你使用的是2.11，这时对着问题`ctrl+1或者右键-> Quick Fix -> 下拉Scala Installation选择相应版本（2.10)`
4. **IDE在Build workspace时缓慢、报内存不够**  
　　调整最大内存分配。这个还是很有必要的，`菜单栏->Scala->Run Setup Diagnostics->选中Use recommended default settings`，如果Heap settings中的有1.5G以上就不用改了，如果有需要，修改eclipse.ini（去安装文件夹下找，把512m改成2048m）即可。


<!-- more >

### 2. org.apache.spark.SparkException: Error communicating with MapOutputTracker
　　问题：少量数据可以运行成功，而数据量放大到数十倍之后运行失败并报这个错。下面是官方对[spark.akka.frameSize](http://spark.apache.org/docs/latest/configuration.html)这个参数的解释

>Maximum message size to allow in "control plane" communication (for serialized tasks and task results), in MB. Increase this if your tasks need to send back large results to the driver (e.g. using collect() on a large dataset).

有了上面的解释就可以明白怎么去调整了：1. 对消息的大小限制放松，2.启用压缩

```scala
val sparkConf = new SparkConf()
  .setAppName("Host Access Freq Stats")
  .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
  .set("spark.akka.frameSize", "30"); //默认是10
```
如果这不能解决问题，可能就需要关注关注其它问题——比如文件块数是不是太多了(10,000+)？可以类似于reduceByKey的时候额外指定分区数：
```scala
rdd.reduceByKey(x => ???, 80)
```
文件块数多的时候调整调整分区数的大小也可以避免大量的10k/1M的小文件产生，小文件太多是会降低IO效率的。

### 3. 查看Accumulator/Counter
　　yarn-cluster模式启动的任务如果集群上配了有spark日志的web服务，是可以在任务执行时/结束后（没有专门的日志服务会临时起一个Spark UI，任务结束就没了）查看任务历史的。在Spark1.1之后“有名的”Accumulator可以在Spark Application UI上查看了——在UI上点击——Stages中子任务的Description，在Accumulators中就可以看到计数器了（有没有计数器还得看子任务是什么，如果是collect，显然是不会有的）。
```scala
//有名计数器
val logLinesAcc = sc.accumulator(0f, "（当日）日志输入")
//使用
logLineAcc += 1.5
```

### 最后
　　有一点写一点，足够长就再开一篇…这个样式真的要调了，好丑！
