---
layout: post
title: Spark入门：02-SparkSQL核心编程
category: spark
tags: [spark]
excerpt: 我们Spark SQL所提供的 DataFrame和DataSet模型进行编程，以及了解它们之间的关系和转换。
lock: need
---

## 1 新的起点

​	Spark Core中，如果想要执行应用程序，需要首先构建上下文环境对象SparkContext，Spark SQL其实可以理解为对Spark Core的一种封装，不仅仅在模型上进行了封装，上下文环境对象也进行了封装。

​	在老的版本中，SparkSQL提供两种SQL查询起始点：一个叫SQLContext，用于Spark自己提供的SQL查询；一个叫HiveContext，用于连接Hive的查询。

​	SparkSession是Spark最新的SQL查询起始点，实质上是SQLContext和HiveContext的组合，所以在SQLContex和HiveContext上可用的API在SparkSession上同样是可以使用的。SparkSession内部封装了SparkContext，所以计算实际上是由sparkContext完成的。当我们使用 spark-shell 的时候, spark 会自动的创建一个叫做spark的SparkSession, 就像我们以前可以自动获取到一个sc来表示SparkContext

![image-20200614164400342](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614164400.png)

## 2 DataFrame

​	Spark SQL的DataFrame API 允许我们使用 DataFrame 而不用必须去注册临时表或者生成 SQL 表达式。DataFrame API 既有 transformation操作也有action操作。

### 2.1 创建DataFrame

​	在Spark SQL中SparkSession是创建DataFrame和执行SQL的入口，创建DataFrame有三种方式：通过Spark的数据源进行创建；从一个存在的RDD进行转换；还可以从Hive Table进行查询返回。

1. 从Spark数据源进行创建

   1. 查看Spark支持创建文件的数据源格式

      ```scala
      scala> spark.read.
      csv   format   jdbc   json   load   option   options   orc   parquet   schema   table   text   textFile
      ```

   2. 在spark的项目input目录中创建user.json文件

      ```json
      {"username":"zhangsan","age":20}
      ```

   3. 读取json文件创建DataFrame

      ```scala
      val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL01")
      val spark = SparkSession.builder().config(sparkConf).getOrCreate()
      import spark.implicits._
      val jsonDF: DataFrame = spark.read.json("input/user.json")
      ```

      ==注意：如果从内存中获取数据，spark可以知道数据类型具体是什么。如果是数字，默认作为Int处理；但是从文件中读取的数字，不能确定是什么类型，所以用bigint接收，可以和Long类型转换，但是和Int不能进行转换==

   4. 展示结果

      ```
      +---+--------+
      |age|username|
      +---+--------+
      | 20|zhangsan|
      +---+--------+
      ```

2. 从RDD进行转换（后面进行完善）

3. 从Hive Table进行查询返回（后面进行完善）

### 2.2 SQL语法

> SQL语法风格是指我们查询数据的时候使用SQL语句来查询，这种风格的查询必须要有临时视图或者全局视图来辅助。

1. 读取JSON文件创建DataFrame

   ```scala
   val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL01")
   val spark = SparkSession.builder().config(sparkConf).getOrCreate()
   import spark.implicits._
   val jsonDF: DataFrame = spark.read.json("input/user.json")
   ```

2. 对DataFrame创建一个临时表

   ```scala
   // 将df转换为临时视图
   jsonDF.createOrReplaceTempView("user")
   ```

   

3. 通过SQL语句实现查询全表

   ```scala
   spark.sql("select * from user").show()
   ```

4. 结果展示

   ```
   +---+--------+
   |age|username|
   +---+--------+
   | 20|zhangsan|
   +---+--------+
   ```

   ==注意：普通临时表是Session范围内的，如果想应用范围内有效，可以使用全局临时表。使用全局临时表时需要全路径访问，如：global_temp.people==

5. 对于DataFrame创建一个全局表

   ```scala
   jsonDF.createGlobalTempView("user1")
   ```

6. 通过SQL语句实现查询全表

   ```scala
   spark.sql("SELECT * FROM global_temp.user1").show()
   ```

### 2.3 DSL语法

​	DataFrame提供一个特定领域语言(domain-specific language, DSL)去管理结构化的数据。可以在 Scala, Java, Python 和 R 中使用 DSL，使用 DSL 语法风格不必去创建临时视图了

1. 创建一个DataFrame

   ```scala
   val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL01")
   val spark = SparkSession.builder().config(sparkConf).getOrCreate()
   import spark.implicits._
   val jsonDF: DataFrame = spark.read.json("input/user.json")
   ```

2. 查看DataFrame的Schema信息

   ```scala
   jsonDF.printSchema
   ```

3. 只查看"username"列数据

   ```scala
   jsonDF.select("username").show()
   
   +---+--------+
   |age|username|
   +---+--------+
   | 20|zhangsan|
   +---+--------+
   ```

4. 查看"username"列数据以及"age+1"数据

   注意：涉及到运算的时候, 每列都必须使用$, 或者采用引号表达式：单引号+字段名

   ```scala
   jsonDF.select($"username",$"age" + 1).show()
   jsonDF.select('username, 'age + 1).show()
   jsonDF.select('username, 'age + 1 as "newage").show()
   ```

5. 查看"age"大于"30"的数据

   ```scala
   jsonDF.filter($"age">30).show()
   ```

6. 按照"age"分组，查看数据条数

   ```scala
   jsonDF.groupBy("age").count.show
   ```

### 2.4 RDD转换为DataFrame

​	在IDEA中开发程序时，如果需要RDD与DF或者DS之间互相操作，那么需要引入 ==import spark.implicits._==

​	这里的spark不是Scala中的包名，而是创建的sparkSession对象的变量名称，所以必须先创建SparkSession对象再导入。这里的spark对象不能使用var声明，因为Scala只支持val修饰的对象的引入。

​	spark-shell中无需导入，自动完成此操作。

```scala
 val rdd = spark.sparkContext.makeRDD(List(
     (1, "zhangsan", 30),
     (2, "lisi", 20),
     (3, "wangwu", 40)
 ))
// RDD转换为DataFrame
val df = rdd.toDF("id", "name", "age")
df.createOrReplaceTempView("user")
```

实际开发中，一般通过样例类将RDD转换为**DataFrame**

```scala
case class User(name:String, age:Int)
sc.makeRDD(List(("zhangsan",30), ("lisi",40))).map(t=>User(t._1, t._2)).toDF.show
```

```scala
// TODO 使用自定义函数在SQL中完成数据的转换
spark.udf.register("addName", (x: String) => "Name:" + x)
spark.udf.register("changeAge", (x: Int) => 18)

spark.sql("select addName(username),changeAge(age) from user").show()
```

### 2.5 DataFrame转换为RDD

DataFrame其实就是对RDD的封装，所以可以直接获取内部的RDD。

```scala
// df: org.apache.spark.sql.DataFrame = [name: string, age: int]
val df = sc.makeRDD(List(("zhangsan",30), ("lisi",40))).map(t=>User(t._1, t._2)).toDF
// rdd: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = MapPartitionsRDD[46] at rdd at <console>:25
val rdd = df.rdd
// array: Array[org.apache.spark.sql.Row] = Array([zhangsan,30], [lisi,40])
val array = rdd.collect
// [zhangsan,30]
println(array(0))
// zhangsan
println(array(0)(0))
// zhangsan
println(array(0).getAs[String]("name"))
```

## 3 DataSet

​	DataSet是具有强类型的数据集合，需要提供对应的类型信息。

### 3.1 创建DataSet

1. 使用样例类序列创建DataSet

   ```scala
   case class Person(name: String, age: Long)
   
   val caseClassDS = Seq(Person("zhangsan",2)).toDS()
   caseClassDS.show
   ```

2. 使用基本类型的序列创建DataSet

   ```scala
   val ds = Seq(1,2,3,4,5).toDS()
   ds.show
   ```

==注意：在实际使用的时候，很少用到把序列转换成DataSet，更多的是通过RDD来得到DataSet==

### 3.2 RDD转换为DataSet

​	SparkSQL能够自动将包含有case类的RDD转换成DataSet，case类定义了table的结构，case类属性通过反射变成了表的列名。Case类可以包含诸如Seq或者Array等复杂的结构。

```scala
// 定义样例类
case class User(name:String, age:Int)

val rdd: RDD[(Int, String, Int)] = spark.sparkContext.makeRDD(List(
    (1, "zhangsan", 30),
    (2, "lisi", 20),
    (3, "wangwu", 40)
))
val userRDD: RDD[User] = rdd.map {
    case (id, name, age) => {
        User(id, name, age)
    }
}
val userDS: Dataset[User] = userRDD.toDS()
val newDS: Dataset[User] = userDS.map(user => {
    User(user.id, "name" + user.name, user.age)
})
newDS.show()
```

### 3.3 DataSet转换为RDD

DataSet其实也是对RDD的封装，所以可以直接获取内部的RDD。

```scala
case class User(name:String, age:Int)

sc.makeRDD(List(("zhangsan",30), ("lisi",49))).map(t => User(t._1, t._2)).toDS()
val rdd = res11.rdd
rdd.collect
```

## 4 DataFrame和DataSet转换

DataFrame其实是DataSet的特例，所以它们之间是可以互相转换的。

- DataFrame转换为DataSet

  ```scala
  case class User(name:String, age:Int)
  
  val df = sc.makeRDD(List(("zhangsan",30), ("lisi",49))).toDF("name","age")
  val ds = df.as[User]
  ```

- DataSet转换为DataFrame

  ```scala
  val ds = df.as[User]
  val df = ds.toDF
  ```

## 5 RDD、DataFrame、DataSet三者的关系

在SparkSQL中Spark为我们提供了两个新的抽象，分别是DataFrame和DataSet。他们和RDD有什么区别呢？

首先从版本的产生上来看：

- Spark1.0 => RDD 
- Spark1.3 => DataFrame
- Spark1.6 => Dataset

  如果同样的数据都给到这三个数据结构，他们分别计算之后，都会给出相同的结果。不同是的他们的执行效率和执行方式。在后期的Spark版本中，DataSet有可能会逐步取代RDD和DataFrame成为唯一的API接口。

### 5.1 三者的共性

- RDD、DataFrame、DataSet全都是spark平台下的分布式弹性数据集，为处理超大型数据提供便利;
- 三者都有惰性机制，在进行创建、转换，如map方法时，不会立即执行，只有在遇到Action如foreach时，三者才会开始遍历运算;
- 三者有许多共同的函数，如filter，排序等;
- 在对DataFrame和Dataset进行操作许多操作都需要这个包:import spark.implicits._（在创建好SparkSession对象后尽量直接导入）
- 三者都会根据 Spark 的内存情况自动缓存运算，这样即使数据量很大，也不用担心会内存溢出

- 三者都有partition的概念
- DataFrame和DataSet均可使用模式匹配获取各个字段的值和类型

### 5.2 三者的区别

**1) RDD**

- RDD一般和spark mlib同时使用

- RDD不支持sparksql操作

**2) DataFrame**

- 与RDD和Dataset不同，DataFrame每一行的类型固定为Row，每一列的值没法直接访问，只有通过解析才能获取各个字段的值

- DataFrame与DataSet一般不与 spark mlib 同时使用

- DataFrame与DataSet均支持 SparkSQL 的操作，比如select，groupby之类，还能注册临时表/视窗，进行 sql 语句操作

- DataFrame与DataSet支持一些特别方便的保存方式，比如保存成csv，可以带上表头，这样每一列的字段名一目了然(后面专门讲解)

**3) DataSet**

- Dataset和DataFrame拥有完全相同的成员函数，区别只是每一行的数据类型不同。 DataFrame其实就是DataSet的一个特例  type DataFrame = Dataset[Row]

- DataFrame也可以叫Dataset[Row],每一行的类型是Row，不解析，每一行究竟有哪些字段，各个字段又是什么类型都无从得知，只能用上面提到的getAS方法或者共性中的第七条提到的模式匹配拿出特定字段。而Dataset中，每一行是什么类型是不一定的，在自定义了case class之后可以很自由的获得每一行的信息

### 5.3 三者的互相转换

见上文。

 ## 6 IDEA开发SparkSQL

实际开发中，都是使用IDEA进行开发的。

### 6.1 添加依赖

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.12</artifactId>
    <version>2.4.5</version>
</dependency>
```

### 6.2 代码实现

```scala
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, sql}
import org.apache.spark.sql.{DataFrame, Dataset, Row, SparkSession}

object SparkSQL01_Test {
  def main(args: Array[String]): Unit = {
    // TODO 创建环境对象

    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL01")
    val spark = SparkSession.builder().config(sparkConf).getOrCreate()

    //spark不是包名，是上下文环境对象名
    import spark.implicits._

    // TODO 逻辑操作
    val jsonDF: DataFrame = spark.read.json("input/user.json")

    // TODO SQL
    // 将df转换为临时视图
    jsonDF.createOrReplaceTempView("user")
    spark.sql("select * from user").show()

    // TODO DSL
    // 如果查询列名采用单引号，那么需要隐式转换。
    jsonDF.select("name","age").show()
    jsonDF.select('name,'age).show()

    val rdd: RDD[(Int, String, Int)] = spark.sparkContext.makeRDD(List(
      (1, "zhangsan", 30),
      (2, "lisi", 20),
      (3, "wangwu", 40)
    ))

    // TODO RDD <=> DataFrame
    val df: DataFrame = rdd.toDF("id","name","age")
    val dfToRDD: RDD[Row] = df.rdd

    // TODO RDD <=> DataSet
    val userRDD: RDD[User] = rdd.map {
      case (id, name, age) => {
        User(id, name, age)
      }
    }
    val userDS: Dataset[User] = userRDD.toDS()
    val dsToRDD: RDD[User] = userDS.rdd

    // TODO DataFrame <=> DataSet
    val dfToDS: Dataset[User] = df.as[User]
    val dsToDF: DataFrame = dfToDS.toDF()


    rdd.foreach(println)
    df.show()
    userDS.show()

    // TODO 释放对象
    spark.stop()
  }
  case class User(id: Int, name: String, age: Int)
}
```

## 7 用户自定义函数

用户可以通过spark.udf功能添加自定义函数，实现自定义功能。

### 7.1 UDF

**1）创建DataFrame**

```scala
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL01")
val spark = SparkSession.builder().config(sparkConf).getOrCreate()
import spark.implicits._
val jsonDF: DataFrame = spark.read.json("input/user.json")
```

**2）注册UDF**

```scala
spark.udf.register("addName", (x: String) => "Name:" + x)
spark.udf.register("changeAge", (x: Int) => 18)
```

**3）创建临时表**

```scala
jsonDF.createOrReplaceTempView("user")
```

**4）应用UDF**

```scala
 spark.sql("select addName(username),changeAge(age) from user").show()
```

### 7.2 UDAF

​	强类型的Dataset和弱类型的DataFrame都提供了相关的聚合函数， 如 **count()，countDistinct()，avg()，max()，min()**。除此之外，用户可以设定自己的自定义聚合函数。通过继承**UserDefinedAggregateFunction**来实现用户自定义聚合函数。

**需求：计算平均工资**

一个需求可以采用很多种不同的方法实现需求

**1）实现方式 - UDAF - RDD**

```scala
val conf: SparkConf = new SparkConf().setAppName("app").setMaster("local[*]")
val sc: SparkContext = new SparkContext(conf)
val res: (Int, Int) = sc.makeRDD(List(("zhangsan", 20), ("lisi", 30), ("wangw", 40))).map {
  case (name, age) => {
    (age, 1)
  }
}.reduce {
  (t1, t2) => {
    (t1._1 + t2._1, t1._2 + t2._2)
  }
}
println(res._1/res._2)
// 关闭连接
sc.stop()
```

**2）实现方式 - UDAF - 累加器**

```scala
class MyAC extends AccumulatorV2[Int,Int]{
  var sum:Int = 0
  var count:Int = 0
  override def isZero: Boolean = {
    return sum ==0 && count == 0
  }

  override def copy(): AccumulatorV2[Int, Int] = {
    val newMyAc = new MyAC
    newMyAc.sum = this.sum
    newMyAc.count = this.count
    newMyAc
  }

  override def reset(): Unit = {
    sum =0
    count = 0
  }

  override def add(v: Int): Unit = {
    sum += v
    count += 1
  }

  override def merge(other: AccumulatorV2[Int, Int]): Unit = {
    other match {
      case o:MyAC=>{
        sum += o.sum
        count += o.count
      }
      case _=>
    }
  }
  override def value: Int = sum/count
}
```

**3）实现方式 - UDAF - 弱类型**

```scala
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types.{DataType, IntegerType, LongType, StructField, StructType}
import org.apache.spark.sql.{DataFrame, Row, SparkSession}

object SparkSQL03_UDAF {
  def main(args: Array[String]): Unit = {
    // TODO 创建环境对象

    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL01")
    val spark = SparkSession.builder().config(sparkConf).getOrCreate()

    //spark不是包名，是上下文环境对象名
    import spark.implicits._
    // TODO 逻辑操作
//    val jsonDF: DataFrame = spark.read.json("input/user.json")
//    jsonDF.createOrReplaceTempView("user")

    val rdd: RDD[(Int, String, Int)] = spark.sparkContext.makeRDD(List(
      (1, "zhangsan", 30),
      (2, "lisi", 20),
      (3, "wangwu", 40)
    ))

    // TODO RDD <=> DataFrame
    val df: DataFrame = rdd.toDF("id","name","age")
    df.createOrReplaceTempView("user")

    spark.udf.register("addName", (x: String) => "Name:" + x)
    spark.udf.register("changeAge", (x: Int) => 18)


    // TODO 定义用户的自定义聚合函数
    // TODO 1. 创建UDAF函数
    val udaf = new MyAvgAgeUDAF
    // TODO 2. 注册到SparkSQL中
    spark.udf.register("avgAge", udaf)
    // TODO 3. 在SQL中使用聚合函数
    spark.sql("select avgAge(age) from user").show()

    // TODO 释放对象
    spark.stop()

  }
  // 自定义聚合函数
  // 1. 继承UserDefinedAggregateFunction
  // 2. 重写方法

  // totalage,count
  class MyAvgAgeUDAF extends UserDefinedAggregateFunction {

    // TODO 输入数据的结构信息：年龄信息
    override def inputSchema: StructType = {
      StructType(Array(StructField("age", IntegerType)))
    }

    // TODO 缓冲区的数据结构信息：年龄的总和，人的数量
    override def bufferSchema: StructType = {
      StructType(Array(
        StructField("totalage", LongType),
        StructField("count", LongType),
      ))
    }

    // TODO 聚合函数返回的结果类型
    override def dataType: DataType = LongType

    // TODO 函数的稳定性，有点像幂等性，传入相同的值，返回的结果一样
    override def deterministic: Boolean = true

    // TODO 函数缓冲区初始化
    override def initialize(buffer: MutableAggregationBuffer): Unit = {
      buffer(0) = 0L
      buffer(1) = 0L
    }

    // TODO 更新缓冲区数据
    override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
      buffer(0) = buffer.getLong(0) + input.getInt(0)
      buffer(1) = buffer.getLong(1) + 1
    }

    // TODO 合并缓冲区
    override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
      buffer1(0) = buffer1.getLong(0) + buffer2.getLong(0)
      buffer1(1) = buffer1.getLong(1) + buffer2.getLong(1)
    }

    // TODO 函数计算
    override def evaluate(buffer: Row): Any = {
      buffer.getLong(0) / buffer.getLong(1)
    }
  }
}
```

**4）实现方式 - UDAF - 强类型**

```scala
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.expressions.Aggregator
import org.apache.spark.sql.{DataFrame, Dataset, Encoder, Encoders, Row, SparkSession, TypedColumn}

object SparkSQL04_UDAF_Class {
  def main(args: Array[String]): Unit = {
    // TODO 创建环境对象
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL")
    // builder 构建，创建
    val spark = SparkSession.builder().config(sparkConf).getOrCreate()
    // 导入隐式转换，这里的spark其实是环境对象的名称
    // 要求这个对象必须使用val声明
    import spark.implicits._
    // TODO 逻辑操作

    val rdd = spark.sparkContext.makeRDD(List(
      (1, "zhangsan", 30L),
      (2, "lisi", 20L),
      (3, "wangwu", 40L)
    ))

    val df = rdd.toDF("id", "name", "age")
    val ds = df.as[User]

    // TODO 创建UDAF函数
    val udaf = new MyAvgAgeUDAFClass
    // TODO 在SQL中使用聚合函数

    // 因为聚合函数是强类型，那么sql中没有类型的概念，所以无法使用
    // 可以采用DSL语法方法进行访问
    // 将聚合函数转换为查询的列让DataSet访问。
    ds.select(udaf.toColumn).show

    spark.stop()
  }

  case class User(id: Int, name: String, age: Long)
  case class AvgBuffer(var totalage: Long, var count: Long)

  // 自定义聚合函数
  // 1. 继承Aggregator，定义泛型
  //    -IN：输入数据的类型User
  //    BUF：缓冲区的数据类型AvgBuffer
  //    OUT：输出数据的类型Long
  // 2. 重写方法
  class MyAvgAgeUDAFClass extends Aggregator[User,AvgBuffer,Long] {
    // TODO 缓冲区的初始值
    override def zero: AvgBuffer = {
      AvgBuffer(0L, 0L)
    }

    // TODO 聚合数据
    override def reduce(buffer: AvgBuffer, user: User): AvgBuffer = {
      buffer.totalage = buffer.totalage + user.age
      buffer.count = buffer.count + 1
      buffer
    }

    // TODO 合并
    override def merge(buffer1: AvgBuffer, buffer2: AvgBuffer): AvgBuffer = {
      buffer1.totalage = buffer1.totalage + buffer2.totalage
      buffer1.count = buffer1.count + buffer2.count
      buffer1
    }

    // TODO 计算函数的结果
    override def finish(reduction: AvgBuffer): Long = {
      reduction.totalage / reduction.count
    }

    // TODO DataSet默认额编解码器，用于序列化，固定写法
    // 自定义类型就是produce   自带类型根据类型选择
    override def bufferEncoder: Encoder[AvgBuffer] = Encoders.product

    override def outputEncoder: Encoder[Long] = Encoders.scalaLong
  }
}
```

## 8 数据的加载和保存

### 8.1 通用的加载和保存方式

> ​	SparkSQL提供了通用的保存数据和数据加载的方式。这里的通用指的是使用相同的API，根据不同的参数读取和保存不同格式的数据，SparkSQL默认读取和保存的文件格式为==parquet==。

**1）加载数据**

spark.read.load 是加载数据的通用方法

```scala
scala> spark.read.
csv   format   jdbc   json   load   option   options   orc   parquet   schema   table   text   textFile
```

如果读取不同格式的数据，可以对不同的数据格式进行设定

```scala
spark.read.format("…")[.option("…")].load("…")
val frame1: DataFrame = spark.read.format("json").load("input/user.json")
```

- format("…")：指定加载的数据类型，包括"csv"、"jdbc"、"json"、"orc"、"parquet"和"textFile"。

- load("…")：在"csv"、"jdbc"、"json"、"orc"、"parquet"和"textFile"格式下需要传入加载数据的路径。

- option("…")：在"jdbc"格式下需要传入JDBC相应参数，url、user、password和dbtable

我们前面都是使用read API 先把文件加载到 DataFrame然后再查询，其实，我们也可以直接在文件上进行查询:  ==文件格式.`文件路径`==

```scala
spark.sql("select * from json.`/opt/module/data/user.json`").show
```

```scala
import org.apache.spark.SparkConf
import org.apache.spark.sql.expressions.Aggregator
import org.apache.spark.sql.{DataFrame, Encoder, Encoders, SparkSession}

object SparkSQL05_LoadSave {
  def main(args: Array[String]): Unit = {

    // TODO 创建环境对象
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL")
    // builder 构建，创建
    val spark = SparkSession.builder().config(sparkConf).getOrCreate()
    import spark.implicits._

    // TODO SparkSQL通用的读取和保存

    // TODO 通用的读取
    // RuntimeException: xxx/input/user.json is not a Parquet file.
    // SparkSQL通用读取的默认数据格式为Parquet列式存储格式。
    val frame: DataFrame = spark.read.load("input/users.parquet")
    frame.show()

    // 如果想要改变读取文件的格式。需要使用特殊的操作。
    // TODO 如果读取的文件格式为JSON格式，Spark对JSON文件的格式有要求
    // JSON => JavaScript Object Notation
    // JSON文件的格式要求整个文件满足JSON的语法规则
    // Spark读取文件默认是以行为单位来读取的。
    // Spark读取JSON文件时，要求文件中的每一行JSON的格式要求
    // 如果文件格式不正确，那么不会发生错误，但是解析结果不正确
    // TODO 通用的读取方式
    // 写框架一般使用这种：spark.read.format("json").load("input/user.json")
    // 专用版：spark.read.json("input/user.json")
    val frame1: DataFrame = spark.read.format("json").load("input/user.json")
    frame1.show()
    spark.stop()
  }
}
```

**2）保存数据**

df.write.save 是保存数据的通用方法，SparkSQL默认通用保存的文件格式为parquet

```scala
val df: DataFrame = spark.read.format("json").load("input/user.json")
scala> df.write.
csv  jdbc   json  orc   parquet textFile… … // 支持的格式
```

如果保存不同格式的数据，可以对不同的数据格式进行设定

```scala
df.write.format("…")[.option("…")].save("…")
```

- format("…")：指定保存的数据类型，包括"csv"、"jdbc"、"json"、"orc"、"parquet"和"textFile"。

- save ("…")：在"csv"、"orc"、"parquet"和"textFile"格式下需要传入保存数据的路径。

- option("…")：在"jdbc"格式下需要传入JDBC相应参数，url、user、password和dbtable

保存操作可以使用 SaveMode, 用来指明如何处理数据，使用mode()方法来设置。

有一点很重要: 这些 SaveMode 都是没有加锁的, 也不是原子操作。

SaveMode是一个枚举类，其中的常量包括：

| Scala/Java                      | Any Language     | Meaning                    |
| ------------------------------- | ---------------- | -------------------------- |
| SaveMode.ErrorIfExists(default) | "error"(default) | 如果文件已经存在则抛出异常 |
| SaveMode.Append                 | "append"         | 如果文件已经存在则追加     |
| SaveMode.Overwrite              | "overwrite"      | 如果文件已经存在则覆盖     |
| SaveMode.Ignore                 | "ignore"         | 如果文件已经存在则忽略     |

```scala
df.write.mode("append").json("/opt/module/data/output")
df.write.mode("append").format("json").save("output")
```

### 8.2 Parquet

> ​	Spark SQL的默认数据源为Parquet格式。Parquet是一种能够有效存储嵌套数据的列式存储格式。
>
> ​	数据源为Parquet文件时，Spark SQL可以方便的执行所有的操作，不需要使用format。修改配置项spark.sql.sources.default，可修改默认数据源格式。

**1）加载数据**

```scala
val df = spark.read.load("input/users.parquet")
df.show
```

**2） 保存数据**

```scala
var df = spark.read.json("input/users.json")
// 保存为parquet格式
df.write.mode("append").save("output")
```

### 8.3 JSON

> ​	Spark SQL 能够自动推测JSON数据集的结构，并将它加载为一个Dataset[Row]. 可以通过SparkSession.read.json()去加载JSON 文件。

==注意：Spark读取的JSON文件不是传统的JSON文件，每一行都应该是一个JSON串。==

格式如下：

```json
{"name":"Michael"}
{"name":"Andy"， "age":30}
{"name":"Justin"， "age":19}
```

1）导入隐式转换

```scala
import spark.implicits._
```

2）加载JSON文件

```scala
val path = "/opt/module/spark-local/people.json"
val peopleDF = spark.read.json(path)
```

3）创建临时表

```scala
peopleDF.createOrReplaceTempView("people")
```

4）数据查询

```scala
val teenagerNamesDF = spark.sql("SELECT name FROM people WHERE age BETWEEN 13 AND 19")
teenagerNamesDF.show()
+------+
|  name|
+------+
|Justin|
+------+
```

### 8.4 CSV

> Spark SQL可以配置CSV文件的列表信息，读取CSV文件,CSV文件的第一行设置为数据列

```scala
object SparkSQL08_Load_CSV {
  def main(args: Array[String]): Unit = {

    // TODO 创建环境对象
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL")
    // builder 构建，创建
    val spark = SparkSession.builder().config(sparkConf).getOrCreate()
    import spark.implicits._

    val frame: DataFrame = spark.read.format("csv")
      .option("sep", ";")
      .option("inferSchema", "true")
      .option("header", "true")
      .load("input/user.csv")
    frame.show()
    spark.stop()
  }
}
```

### 8.5 MySQL

> ​	Spark SQL可以通过JDBC从关系型数据库中读取数据的方式创建DataFrame，通过对DataFrame一系列的计算后，还可以将数据再写回关系型数据库中。如果使用spark-shell操作，可在启动shell时指定相关的数据库驱动路径或者将相关的数据库驱动放到spark的类路径下。

```scala
bin/spark-shell --jars mysql-connector-java-5.1.27-bin.jar
```

我们这里只演示在Idea中通过JDBC对Mysql进行操作

**1）导入依赖**

```scala
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.27</version>
</dependency>
```

**2）读取数据**

```scala
val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL")

//创建SparkSession对象
val spark: SparkSession = SparkSession.builder().config(conf).getOrCreate()

import spark.implicits._

//方式1：通用的load方法读取
spark.read.format("jdbc")
    .option("url", "jdbc:mysql://localhost:3306/test")
    .option("driver", "com.mysql.jdbc.Driver")
    .option("user", "root")
    .option("password", "123456")
    .option("dbtable", "user")
    .load().show


//方式2:通用的load方法读取 参数另一种形式
spark.read.format("jdbc")
  .options(Map("url"->"jdbc:mysql://localhost:3306/test?user=root&password=123123",
    "dbtable"->"user","driver"->"com.mysql.jdbc.Driver")).load().show

//方式3:使用jdbc方法读取
val props: Properties = new Properties()
props.setProperty("user", "root")
props.setProperty("password", "123123")
val df: DataFrame = spark.read.jdbc("jdbc:mysql://localhost:3306/test", "user", props)
df.show

//释放资源
spark.stop()
```

**3）写入数据**

```scala
import org.apache.spark.SparkConf
import org.apache.spark.sql.{DataFrame, SaveMode, SparkSession}


object SparkSQL10_Save_MySQL {

    def main(args: Array[String]): Unit = {

        // TODO 创建环境对象
        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL")
        // builder 构建，创建
        val spark = SparkSession.builder().config(sparkConf).getOrCreate()
        import spark.implicits._

        al rdd: RDD[User2] = spark.sparkContext.makeRDD(List(User2("lisi", 20), User2("zs", 30)))
		val frameDS: Dataset[User2] = rdd.toDS
        // 方式1：通用的方式  format指定写出类型
        frameDS.write.format("jdbc")
                .option("url", "jdbc:mysql://localhost:3306/test")
                .option("driver", "com.mysql.jdbc.Driver")
                .option("user", "root")
                .option("password", "123123")
                .option("dbtable", "user1")
                .mode(SaveMode.Append)
                .save()
		//方式2：通过jdbc方法
        val props: Properties = new Properties()
        props.setProperty("user", "root")
        props.setProperty("password", "123123")
        ds.write.mode(SaveMode.Append)
        		.jdbc("jdbc:mysql://localhost:3306/test", "user", props)
        
        spark.stop
    }
}
```

### 8.6 Hive

**代码操作Hive**

1）导入依赖

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-hive_2.12</artifactId>
    <version>2.4.5</version>
</dependency>

<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>3.1.2</version>
</dependency>
```

2）将hive-site.xml文件拷贝到项目的resources目录中，代码实现

==将集群的/opt/module/hive/conf/hive-site.xml 拷贝到项目的resources目录，正常情况什么都不需要改。==

```scala
import org.apache.spark.SparkConf
import org.apache.spark.sql.{DataFrame, SaveMode, SparkSession}

object SparkSQL11_Load_Hive {
    def main(args: Array[String]): Unit = {
        // TODO 创建环境对象
        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL")
        // builder 构建，创建

        // TODO 默认情况下SparkSQL支持本地Hive操作的，执行前需要启用Hive的支持
        // 调用enableHiveSupport方法。
        val spark = SparkSession.builder()
                .enableHiveSupport()
                .config(sparkConf).getOrCreate()
        // 导入隐式转换，这里的spark其实是环境对象的名称

        // 可以使用基本的sql访问hive中的内容
        //spark.sql("create table aa(id int)")
        //spark.sql("show tables").show()
        spark.sql("load data local inpath 'input/id.txt' into table aa")
        spark.sql("select * from aa").show

        spark.stop
    }
}
```

注意：在开发工具中创建数据库默认是在本地仓库，通过参数修改数据库仓库的地址: config("spark.sql.warehouse.dir", "hdfs://hadoop112:8020/user/hive/warehouse")

如果在执行操作时，出现如下错误：

![image-20200614210311118](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200614210311.png)

可以代码最前面增加如下代码解决：

```scala
System.setProperty("HADOOP_USER_NAME", "atguigu")
```

