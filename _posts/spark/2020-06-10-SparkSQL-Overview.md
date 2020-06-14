---
layout: post
title: Spark入门：01-SparkSQL概述
category: spark
tags: [spark]
excerpt: Spark SQL是Spark用于结构化数据(structured data)处理的Spark模块。
lock: need
---

## 1 SparkSQL是什么

![image-20200614162601353](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614162601.png)

![image-20200614162617567](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614162617.png)

官网链接：http://spark.apache.org/sql/

## 2 Hive and SparkSQL

​	SparkSQL的前身是Shark，给熟悉RDBMS但又不理解MapReduce的技术人员提供快速上手的工具。
​	Hive是早期唯一运行在Hadoop上的SQL-on-Hadoop工具。但是MapReduce计算过程中大量的中间磁盘落地过程消耗了大量的I/O，降低的运行效率，为了提高SQL-on-Hadoop的效率，大量的SQL-on-Hadoop工具开始产生，其中表现较为突出的是：

- Drill
- Impala
- Shark

其中Shark是伯克利实验室Spark生态环境的组件之一，是基于Hive所开发的工具，它修改了下图所示的右下角的内存管理、物理计划、执行三个模块，并使之能运行在Spark引擎上。

![image-20200614162835680](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614162835.png)

Shark的出现，使得SQL-on-Hadoop的性能比Hive有了10-100倍的提高。

![image-20200614162902009](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614162902.png)

​	 但是，随着Spark的发展，对于野心勃勃的Spark团队来说，Shark对于Hive的太多依赖（如采用Hive的语法解析器、查询优化器等等），制约了Spark的One Stack Rule Them All的既定方针，制约了Spark各个组件的相互集成，所以提出了SparkSQL项目。SparkSQL抛弃原有Shark的代码，汲取了Shark的一些优点，如内存列存储（In-Memory Columnar Storage）、Hive兼容性等，重新开发了SparkSQL代码；由于摆脱了对Hive的依赖性，SparkSQL无论在数据兼容、性能优化、组件扩展方面都得到了极大的方便，真可谓“退一步，海阔天空”。

- 数据兼容方面 SparkSQL不但兼容Hive，还可以从RDD、parquet文件、JSON文件中获取数据，未来版本甚至支持获取RDBMS数据以及cassandra等NOSQL数据；

- 性能优化方面 除了采取In-Memory Columnar Storage、byte-code generation等优化技术外、将会引进Cost Model对查询进行动态评估、获取最佳物理计划等等；

- 组件扩展方面 无论是SQL的语法解析器、分析器还是优化器都可以重新定义，进行扩展。

![image-20200614162948384](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614162948.png)

2014年6月1日Shark项目和SparkSQL项目的主持人Reynold Xin宣布：停止对Shark的开发，团队将所有资源放SparkSQL项目上，至此，Shark的发展画上了句话，但也因此发展出两个支线：SparkSQL和Hive on Spark。

![image-20200614163025496](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614163025.png)

​	其中SparkSQL作为Spark生态的一员继续发展，而不再受限于Hive，只是兼容Hive；而Hive on Spark是一个Hive的发展计划，该计划将Spark作为Hive的底层引擎之一，也就是说，Hive将不再受限于一个引擎，可以采用Map-Reduce、Tez、Spark等引擎。

​	对于开发人员来讲，SparkSQL可以简化RDD的开发，提高开发效率，且执行效率非常快，所以实际工作中，基本上采用的就是Spark SQL。Spark SQL为了简化RDD的开发，提高开发效率，提供了2个编程抽象，类似Spark Core中的RDD

- DataFrame
- DataSet

## 3 SparkSQL特点

### 3.1 易整合

无缝的整合了 SQL 查询和 Spark 编程。

![image-20200614163139158](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614163139.png)

### 3.2 统一的数据访问

使用相同的方式连接不同的数据源。

![image-20200614163211145](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614163211.png)

### 3.3 兼容Hive

![image-20200614163256527](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614163256.png)

### 3.4 标准数据连接

通过 JDBC 或者 ODBC 来连接。

![image-20200614163339192](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614163339.png)

## 4 DataFrame是什么

​	在Spark中，DataFrame是一种以RDD为基础的分布式数据集，类似于传统数据库中的二维表格。DataFrame与RDD的主要区别在于，前者带有schema元信息，即DataFrame所表示的二维表数据集的每一列都带有名称和类型。这使得Spark SQL得以洞察更多的结构信息，从而对藏于DataFrame背后的数据源以及作用于DataFrame之上的变换进行了针对性的优化，最终达到大幅提升运行时效率的目标。反观RDD，由于无从得知所存数据元素的具体内部结构，Spark Core只能在stage层面进行简单、通用的流水线优化。

​	同时，与Hive类似，DataFrame也支持嵌套数据类型（struct、array和map）。从 API 易用性的角度上看，DataFrame API提供的是一套高层的关系操作，比函数式的RDD API 要更加友好，门槛更低。

![image-20200614163509736](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614163509.png)

​	上图直观地体现了DataFrame和RDD的区别。

​	左侧的RDD[Person]虽然以Person为类型参数，但Spark框架本身不了解Person类的内部结构。而右侧的DataFrame却提供了详细的结构信息，使得 Spark SQL 可以清楚地知道该数据集中包含哪些列，每列的名称和类型各是什么。

​	DataFrame是为数据提供了Schema的视图。可以把它当做数据库中的一张表来对待

​	DataFrame也是懒执行的，但性能上比RDD要高，主要原因：优化的执行计划，即查询计划通过Spark catalyst optimiser进行优化。	比如下面一个例子:

![image-20200614163610846](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614163611.png)

​	为了说明查询优化，我们来看上图展示的人口数据分析的示例。图中构造了两个DataFrame，将它们join之后又做了一次filter操作。如果原封不动地执行这个执行计划，最终的执行效率是不高的。因为join是一个代价较大的操作，也可能会产生一个较大的数据集。如果我们能将filter下推到 join下方，先对DataFrame进行过滤，再join过滤后的较小的结果集，便可以有效缩短执行时间。而Spark SQL的查询优化器正是这样做的。简而言之，逻辑查询计划优化就是一个利用基于关系代数的等价变换，将高成本的操作替换为低成本操作的过程。 

![image-20200614163646669](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614163646.png)

## 5 DataSet是什么

​	DataSet是分布式数据集合。DataSet是Spark 1.6中添加的一个新抽象，是DataFrame的一个扩展。它提供了RDD的优势（强类型，使用强大的lambda函数的能力）以及Spark SQL优化执行引擎的优点。DataSet也可以使用功能性的转换（操作map，flatMap，filter等等）。

- DataSet是DataFrame API的一个扩展，是SparkSQL最新的数据抽象

- 用户友好的API风格，既具有类型安全检查也具有DataFrame的查询优化特性；

- 用样例类来对DataSet中定义数据的结构信息，样例类中每个属性的名称直接映射到DataSet中的字段名称；

- DataSet是强类型的。比如可以有DataSet[Car]，DataSet[Person]。

- DataFrame是DataSet的特列，DataFrame=DataSet[Row] ，所以可以通过as方法将DataFrame转换为DataSet。Row是一个类型，跟Car、Person这些的类型一样，所有的表结构信息都用Row来表示。获取数据时需要指定顺序