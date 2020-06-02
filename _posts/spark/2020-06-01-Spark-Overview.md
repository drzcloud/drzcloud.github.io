---
layout: post
title: Hive入门：01-Spark概述
category: spark
tags: [spark]
excerpt: Spark概述
lock: need
---

## 1.1 Spark是什么

![image-20200602231522694](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200602231522.png)

Spark是一种基于内存的快速、通用、可扩展的大数据分析计算引擎。

## 1.2 Spark and Hadoop

​	 在之前的学习中，Hadoop的MapReduce是大家广为熟知的计算框架，那为什么咱们还要学习新的计算框架Spark呢，这里就不得不提到Spark和Hadoop的关系。

首先从**时间节点**上来看：

- Hadoop
  - 2006年1月，Doug Cutting加入Yahoo，领导Hadoop的开发
  - 2008年1月，Hadoop成为Apache顶级项目
  - 2011年1.0正式发布
  - 2012年3月稳定版发布
  - 2013年10月发布2.X (Yarn)版本
- Spark
  - 2009年，Spark诞生于伯克利大学的AMPLab实验室
  - 2010年，伯克利大学正式开源了Spark项目
  - 2013年6月，Spark成为了Apache基金会下的项目
  - 2014年2月，Spark以飞快的速度成为了Apache的顶级项目
  - 2015年至今，Spark变得愈发火爆，大量的国内公司开始重点部署或者使用Spark

然后我们再从**功能**上来看:

- Hadoop
  - Hadoop是由java语言编写的，在分布式服务器集群上存储海量数据并运行分布式分析应用的开源框架
  - 作为Hadoop分布式文件系统，HDFS处于Hadoop生态圈的最下层，存储着所有的数据，支持着Hadoop的所有服务。它的理论基础源于Google的TheGoogleFileSystem这篇论文，它是GFS的开源实现。
  - MapReduce是一种编程模型，Hadoop根据Google的MapReduce论文将其实现，作为Hadoop的分布式计算模型，是Hadoop的核心。基于这个框架，分布式并行程序的编写变得异常简单。综合了HDFS的分布式存储和MapReduce的分布式计算，Hadoop在处理海量数据时，性能横向扩展变得非常容易。
  - HBase是对Google的Bigtable的开源实现，但又和Bigtable存在许多不同之处。HBase是一个基于HDFS的分布式数据库，擅长实时地随机读/写超大规模数据集。它也是Hadoop非常重要的组件。
- Spark
  - Spark是一种由Scala语言开发的快速、通用、可扩展的大数据分析引擎
  - Spark Core中提供了Spark最基础与最核心的功能
  - Spark SQL是Spark用来操作结构化数据的组件。通过Spark SQL，用户可以使用SQL或者Apache Hive版本的SQL方言（HQL）来查询数据。
  - Spark Streaming是Spark平台上针对实时数据进行流式计算的组件，提供了丰富的处理数据流的API。

​    由上面的信息可以获知，Spark出现的时间相对较晚，并且主要功能主要是用于数据计算，所以其实Spark一直被认为是Hadoop MR框架的升级版。

## 1.3 Spark or Hadoop

Hadoop的MR框架和Spark框架都是数据处理框架，那么我们在使用时如何选择呢？

- Hadoop MapReduce由于其设计初衷并不是为了满足循环迭代式数据流处理，因此在多并行运行的数据可复用场景（如：机器学习、图挖掘算法、交互式数据挖掘算法）中存在诸多计算效率等问题。所以Spark应运而生，Spark就是在传统的MapReduce 计算框架的基础上，利用其计算过程的优化，从而大大加快了数据分析、挖掘的运行和读写速度，并将计算单元缩小到更适合并行计算和重复使用的RDD。

- 机器学习中ALS、凸优化梯度下降等。这些都需要基于数据集或者数据集的衍生数据反复查询反复操作。MR这种模式不太合适，即使多MR串行处理，性能和时间也是一个问题。数据的共享依赖于磁盘。另外一种是交互式数据挖掘，MR显然不擅长。而Spark所基于的scala语言恰恰擅长函数的处理。

- Spark是一个分布式数据快速分析项目。它的核心技术是弹性分布式数据集（Resilient Distributed Datasets），提供了比MapReduce丰富的模型，可以快速在内存中对数据集进行多次迭代，来支持复杂的数据挖掘算法和图形计算算法。

- Spark和Hadoop的根本差异是多个任务之间的数据通信问题 : Spark多个任务之间数据通信是基于内存，而Hadoop是基于磁盘。

- Spark Task的启动时间快。Spark采用fork线程的方式，而Hadoop采用创建新的进程的方式。

- Spark只有在shuffle的时候将数据写入磁盘，而Hadoop中多个MR作业之间的数据交互都要依赖于磁盘交互

- Spark的缓存机制比HDFS的缓存机制高效。

​    经过上面的比较，我们可以看出在绝大多数的数据计算场景中，Spark确实会比MapReduce更有优势。但是Spark是基于内存的，所以在实际的生产环境中，由于内存的限制，可能会由于内存资源不够导致Job执行失败，此时，MapReduce其实是一个更好的选择，所以Spark并不能完全替代MR。

## 1.4 Spark 核心模块

![image-20200602232317059](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200602232317.png)

**Spark Core：**

​      Spark Core中提供了Spark最基础与最核心的功能，Spark其他的功能如：Spark SQL，Spark Streaming，GraphX, MLlib都是在Spark Core的基础上进行扩展的

**Spark SQL：**

​      Spark SQL是Spark用来操作结构化数据的组件。通过Spark SQL，用户可以使用SQL或者Apache Hive版本的SQL方言（HQL）来查询数据。

**Spark Streaming：**

​      Spark Streaming是Spark平台上针对实时数据进行流式计算的组件，提供了丰富的处理数据流的API。

**Spark MLlib：**

​      MLlib是Spark提供的一个机器学习算法库。MLlib不仅提供了模型评估、数据导入等额外的功能，还提供了一些更底层的机器学习原语。

**Spark GraphX：**

​      GraphX是Spark面向图计算提供的框架与算法库。