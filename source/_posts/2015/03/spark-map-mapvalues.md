layout: post
title: "Spark学习——map & mapValues"
date: "2015-03-27 20:31"
tags:
- Spark
- Scala
categories:
- Spark
---
　　有时候觉得会不会从来没看过源码会是种遗憾，尝试着去看spark源码时又发现看不懂……

　　map和mapValues的区别其实很大，最重要的区别是mapValues只对Tuple2的第二个元素进行操作，保留第一个元素key不变（废话）。先上结论：
  - mapValues在不改变数据分区的情况下对数据进行一些转换操作，从而避免在进一步的join/reduce之类的操作中产生不必要的shuffle开销，如果想要制造narrow dependency，这一点还是需要知道；
  - map操作过后partitioner会丢失，仿上——join/reduce之类的操作时会产生shuffle

Spark源码中map/mapValues的相关体现:
```scala
//RDD中的partitioner
  /** Optionally overridden by subclasses to specify how they are partitioned. */
  @transient val partitioner: Option[Partitioner] = None

//MappedRDD
private[spark]
class MappedRDD[U: ClassTag, T: ClassTag](prev: RDD[T], f: T => U)
  extends RDD[U](prev) {

  override def getPartitions: Array[Partition] = firstParent[T].partitions

  override def compute(split: Partition, context: TaskContext) =
    firstParent[T].iterator(split, context).map(f)
}
```
<!-- more >
　　map的相关代码在RDD中，值得注意的是其中的partitioner变量是带有`@transient`标记的，标记的具体解释可以参照
{% blockquote Annotations https://www.artima.com/pins1ed/annotations.html Programming in Scala %}
Finally, Scala provides a @transient annotation for fields that should not be serialized at all. If you mark a field as @transient, then the framework should not save the field even when the surrounding object is serialized. When the object is loaded, the field will be restored to the default value for the type of the field annotated as @transient.
{% endblockquote %}
　　这也就意味着由map生成的RDD都是不带有partition信息的。同样的，再看看`MappedValuesRDD`
```scala
private[spark]
class MappedValuesRDD[K, V, U](prev: RDD[_ <: Product2[K, V]], f: V => U)
  extends RDD[(K, U)](prev) {

  override def getPartitions = firstParent[Product2[K, U]].partitions

  override val partitioner = firstParent[Product2[K, U]].partitioner

  override def compute(split: Partition, context: TaskContext): Iterator[(K, U)] = {
    firstParent[Product2[K, V]].iterator(split, context).map { pair => (pair._1, f(pair._2)) }
  }
}
```
　　MappedValuesRDD中，partitioner是直接取了第一个祖先的分区的，所以RDD的partition信息是得到了保留的。
