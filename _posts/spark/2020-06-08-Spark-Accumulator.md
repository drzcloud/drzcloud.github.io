---
layout: post
title: Spark入门：08-Spark累加器
category: spark
tags: [spark]
excerpt: 累加器用来把Executor端变量信息聚合到Driver端。在Driver程序中定义的变量，在Executor端的每个Task都会得到这个变量的一份新的副本，每个task更新这些副本的值后，传回Driver端进行merge。
lock: need
---

> ​	累加器用来把Executor端变量信息聚合到Driver端。在Driver程序中定义的变量，在Executor端的每个Task都会得到这个变量的一份新的副本，每个task更新这些副本的值后，传回Driver端进行merge。
>

## 1 系统累加器

**累加器类型：**

- sc.longAccumulator()				// long类型
- sc.doubleAccumulator()           // double类型
- sc.collectionAccumulator()      // 集合类型

```scala
val rdd = sc.makeRDD(List(1,2,3,4,5))
// 声明累加器
var sum = sc.longAccumulator("sum");
rdd.foreach(
  num => {
    // 使用累加器
    sum.add(num)
  }
)
// 获取累加器的值
println("sum = " + sum.value)
```

## 2 自定义累加器

使用自定义累加器实现WordCount。

```scala
import org.apache.spark.rdd.RDD
import org.apache.spark.util.{AccumulatorV2, LongAccumulator}
import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable

/**
 * Created with IntelliJ IDEA.
 *
 * @Author: drz
 * @Date: 2020/06/08 11:57
 * @Description:
 */
object Spark63_Acc3 {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("FIle - RDD")
    val sc = new SparkContext(sparkConf)
    val rdd: RDD[String] = sc.makeRDD(List("hello word", "hello ", "spark", "scale"))
    // TODO 累加器：WordCount
    // TODO 1. 创建累加器
    val acc = new MyWordCountAccumulator
    // TODO 2. 注册累加器
    sc.register(acc)

    // TODO 3. 使用累加器
    rdd.flatMap(_.split(" ")).foreach {
      word => {
        acc.add(word)
      }
    }
    // TODO 4. 获取累加器
    println(acc.value)

  }

  // TODO 自定义累加器
  //  1. 继承AccumulatorV2，定义泛型[IN,OUT]
  //              IN ：累加器输入的值的类型
  //              OUT：累加器返回结果的类型
  //  2. 重写方法
  class MyWordCountAccumulator extends AccumulatorV2[String, mutable.Map[String, Int]] {

    // 存储WordCount的集合
    var wordCountMap = mutable.Map[String, Int]()

    // TODO 累加器是否初始化
    override def isZero: Boolean = {
      wordCountMap.isEmpty
    }

    // TODO 复制累加器
    override def copy(): AccumulatorV2[String, mutable.Map[String, Int]] = {
      new MyWordCountAccumulator
    }

    // TODO 重置累加器
    override def reset(): Unit = {
      wordCountMap.clear()
    }

    // TODO 向累加器中增加值
    override def add(word: String): Unit = {
      // word - count
      wordCountMap.update(word, wordCountMap.getOrElse(word, 0) + 1)
    }

    // TODO 合并当前累加器和其他累加器
    //  合并累加器
    override def merge(other: AccumulatorV2[String, mutable.Map[String, Int]]): Unit = {
      val map1 = wordCountMap
      val map2 = other.value
      wordCountMap = map1.foldLeft(map2)(
        (map, kv) => {
          map(kv._1) = map.getOrElse(kv._1, 0) + kv._2
          map
        }
      )
    }

    // TODO 返回累加器的值(Out)
    override def value: mutable.Map[String, Int] = {
      wordCountMap
    }
  }
}
```

