---
layout: post
title: Spark入门：09-Spark广播变量
category: spark
tags: [spark]
excerpt: 广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个或多个Spark操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表，广播变量用起来都很顺手。在多个并行操作中使用同一个变量，但是 Spark会为每个任务分别发送。
lock: need
---

&nbsp;&nbsp;&nbsp;&nbsp;广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个或多个Spark操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表，广播变量用起来都很顺手。在多个并行操作中使用同一个变量，但是 Spark会为每个任务分别发送。

## 1 基础编程

使用自定义累加器实现WordCount。

```scala
// 广播变量：分布式共享只读变量

val rdd1 = sc.makeRDD(List(("a",1),("b",2), ("c",3)))
val list = List(("a",4),("b",5), ("c",6))
// TODO 声明广播变量
val bcList: Broadcast[List[(String, Int)]] = sc.broadcast(list)

val rdd2 = rdd1.map{
    case ( word, count1 ) => {
        var count2 = 0
        // TODO 使用广播变量
        for ( kv <- bcList.value ) {
            val w = kv._1
            val v = kv._2
            if ( w == word ) {
                count2 = v
            }
        }

        (word, (count1, count2))
    }
}
println(rdd2.collect().mkString(","))
```

![image-20200608232441317](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200608232441.png)