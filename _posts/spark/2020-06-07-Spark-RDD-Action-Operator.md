---
layout: post
title: Spark入门：07-Spark-RDD行动算子
category: spark
tags: [spark]
excerpt: 所谓的行动算子，其实不会在产生新的RDD,而是触发作业的执行。
lock: need
---

## RDD行动算子

```scala
// 所谓的行动算子，其实不会在产生新的RDD,而是触发作业的执行
// 行动算子执行后，会获取到作业的执行结果
// 转换算子不会触发作业的执行，只是功能的扩展和包装
```

### 1）reduce

> 聚集RDD中的所有元素，先聚合分区内数据，再聚合分区间数据

- 函数签名

```scala
def reduce(f: (T, T) => T): T
```

- 代码演示

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 聚合数据
val data: Int = rdd.reduce(_+_)
// sout:10
println(data)
```

### 2）collect

> 在驱动程序中，以数组Array的形式返回数据集的所有元素

- 函数签名

```
def collect(): Array[T]
```

- 代码演示

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))

// 收集数据到Driver
rdd.collect().foreach(println)
```

### 3）count

> 返回RDD中元素的个数

- 函数签名

```scala
def count(): Long
```

- 代码演示

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))

// 返回RDD中元素的个数
val countResult: Long = rdd.count()
```

### 4）first

> 返回RDD中的第一个元素

- 函数签名

```scala
def first(): T
```

- 代码演示

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))

// 返回RDD中元素的个数
val firstResult: Int = rdd.first()
println(firstResult)
```

### 5）take

> 返回一个由RDD的前n个元素组成的数组

- 函数签名

```scala
def take(num: Int): Array[T]
```

- 代码演示

```
vval rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))

// 返回RDD中元素的个数
val takeResult: Array[Int] = rdd.take(2)
println(takeResult.mkString(","))
```

### 6）takeOrdered

> 返回该RDD排序后的前n个元素组成的数组

- 函数签名

```scala
def takeOrdered(num: Int)(implicit ord: Ordering[T]): Array[T]
```

- 代码演示

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1,3,2,4))

// 返回RDD中元素的个数
val result: Array[Int] = rdd.takeOrdered(2)
```

### 7）aggregate

> 分区的数据通过初始值和分区内的数据进行聚合，然后再和初始值进行分区间的数据聚合

- 函数签名

```scala
def aggregate[U: ClassTag](zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U): U
```

- 代码演示

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4), 8)

// 将该RDD所有元素相加得到结果
//val result: Int = rdd.aggregate(0)(_ + _, _ + _)
val result: Int = rdd.aggregate(10)(_ + _, _ + _)
```

### 8）fold

> 折叠操作，aggregate的简化版操作

- 函数签名

```scala
def fold(zeroValue: T)(op: (T, T) => T): T
```

- 代码演示

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4))
val foldResult: Int = rdd.fold(0)(_+_)
```

### 9）countByKey

> 统计每种key的个数。

- 函数签名

```scala
def countByKey(): Map[K, Long]
```

- 代码演示

```scala
val rdd: RDD[(Int, String)] = sc.makeRDD(List((1, "a"), (1, "a"), (1, "a"), (2, "b"), (3, "c"), (3, "c")))

// 统计每种key的个数
val result: collection.Map[Int, Long] = rdd.countByKey()
```

### 10）save相关算子

> 将数据保存到不同格式的文件中。

- 函数签名

```scala
def saveAsTextFile(path: String): Unit
def saveAsObjectFile(path: String): Unit
def saveAsSequenceFile(
  path: String,
  codec: Option[Class[_ <: CompressionCodec]] = None): Unit
```

- 代码演示

```scala
// 保存成Text文件
rdd.saveAsTextFile("output")

// 序列化成对象保存到文件
rdd.saveAsObjectFile("output1")

// 保存成Sequencefile文件
rdd.map((_,1)).saveAsSequenceFile("output2")
```

### 11）foreach

> 分布式遍历RDD中的每一个元素，调用指定函数。

- 函数签名

```scala
def foreach(f: T => Unit): Unit = withScope {
    val cleanF = sc.clean(f)
    sc.runJob(this, (iter: Iterator[T]) => iter.foreach(cleanF))
}
```

- 代码演示

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))

// 收集后打印
rdd.map(num=>num).collect().foreach(println)

println("****************")

// 分布式打印
rdd.foreach(println)
```
