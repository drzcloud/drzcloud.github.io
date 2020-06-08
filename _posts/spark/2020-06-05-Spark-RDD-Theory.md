---
layout: post
title: Spark入门：05-Spark-RDD概念
category: spark
tags: [spark]
excerpt: Spark计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于处理不同的应用场景。
lock: need
---

Spark计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于处理不同的应用场景。三大数据结构分别是：

- RDD : 弹性分布式数据集

- 累加器：分布式共享只写变量

- 广播变量：分布式共享只读变量

接下来我们一起看看这三大数据结构是如何在数据处理中使用的。

## 1 RDD

```sql
-- 什么是RDD?
弹性分布式数据集，一种数据处理的模型&数据结构，是一个抽象类。如汽车模型，航母模型、手机模型等。

-- RDD的特点：
	1. 可分区：提高消费能力，更适合并发计算，类似kafka的消费者消费数据
	2. 弹性：变化，可变。
		a、存储弹性：可以在磁盘和内存之间自动切换；
		b、容错弹性：数据丢失可以自动回复；
		c、计算弹性：计算出错后重试机制；
		d、分区弹性： 根据计算结果动态改变分区的数量。
     3. 不可变：类似不可变集合
          RDD只存储计算的逻辑，不存储数据，计算的逻辑是不可变的，一旦改变，则会创建新的RDD；
     4. RDD ：一个抽象类，需要子类具体实现,说明有很多种数据处理方式
```

## 2 IO

```sql
-- IO流分为：
          字节流          字符流
输入流   inputStream       read
输出流   outPutStream      write

节点流  File+ 字符流/字节流
处理流  buffer(  File+ 字符流/字节流 )

-- 解读如下三张图流程
图1：使用字节流读取一个文件的内容并打印到控制台，使用一个文件节点流，只能读取一部分数据，然后打印，然后再读取一部分数据，再进行打印，慢；
图2：增加一个缓冲流，将获取的数据暂时先存放在内存的一个缓冲区内，等到一定的数据量以后，再统一处理；
图3：发现字节流获取的数据，打印到控制台，我们是不认识的，中间使用一个字节流转字符流，将读取的数据转化为字符流，然后再将字符缓存到内存，待达到一定的数据量以后，再往控制台上打印。

综上，发现，如上的过程属于装饰者模式，前者的结果传递到后者，一层一层的包裹起来。
```

![image-20200604221413351](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604221413.png)
![image-20200604221320626](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604221320.png)

![image-20200604221352619](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604221352.png)

## 3 RDD执行原理

```sql
-- RDD的执行原理：
1. 类似IO处理；
2. 体现装饰者模式，通过new的方式体现装饰者模式；
3. 延迟加载，RDD只是封装了逻辑，只要当执行算子（如collect()）执行时，才会开始执行。

-- 与IO的区别：
RDD不保存数据，只保留逻辑，但是IO会保存数据

-- 如何理解RDD
不可变集合 --> 增加新的数据 --> 创建新的集合
RDD --> 扩展新的功能 --> 创建新的RDD
```

![image-20200604221511071](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604221511.png)

```sql
-- 解读RDD：
1.一个执行器Executor可以有多个core核，默认情况下一个分区产生一个task，一个 task可以被一个core执行，由于一个执行器可以有多个核，所以一个执行器可以执行多个task；
```

![image-20200604221612259](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604221612.png)

## 4 RDD的核心属性

### 4.1 分区列表

RDD数据结构中存在分区列表，用于执行任务时并行计算，是实现分布式计算的重要属性。

```sql
-- 什么是分区列表
就是RDD中的多个分区
```

![image-20200604225854664](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604225854.png)

### 4.2 分区计算函数

```sql
-- 什么是分区计算函数
Spark在计算时，是使用分区函数对每一个分区进行计算
```

![image-20200604225648602](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604225648.png)

### 4.3 RDD之间的依赖关系

RDD是计算模型的封装，当需求中需要将多个计算模型进行组合时，就需要将多个RDD建立依赖关系。

![image-20200604225800723](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604225800.png)

### 4.4 分区器（可选）

Spark在计算时，是使用分区函数对每一个分区进行计算

```sql
-- 什么是分区器
数据进入分区的规则，对数据进行分区。
1. 只能是KV键值对的数据可以进行分区
2. 默认没有分区器
```

![image-20200604230017251](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604230017.png)

### 4.5 首选位置（可选）

```
计算数据时，可以根据计算节点的状态选择不同的节点位置进行计算
```

![image-20200604230048390](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604230048.png)

![20200603003308](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604230102.png)

