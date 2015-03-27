layout: post
title: "Spark学习——map & mapValues"
date: "2015-03-27 20:31"
tags:
- Spark
categories:
- Spark
- Scala
---
　　有时候觉得会不会从来没看过源码会是种遗憾，尝试着去看spark源码时又发现看不懂……

map和mapValues的区别其实很大，
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

{% blockquote Annotations https://www.artima.com/pins1ed/annotations.html Programming in Scala %}
Finally, Scala provides a @transient annotation for fields that should not be serialized at all. If you mark a field as @transient, then the framework should not save the field even when the surrounding object is serialized. When the object is loaded, the field will be restored to the default value for the type of the field annotated as @transient.
{% endblockquote %}


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

在spark中map和mapValues还是有很大的区别的
