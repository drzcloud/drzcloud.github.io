---
layout: post
title: Spark入门：03-SparkSQL项目实战
category: spark
tags: [spark]
excerpt: 通过实战案例练习下SparkSQL的使用。
lock: need
---

## 1 数据准备

我们这次 Spark-sql 操作中所有的数据均来自 Hive，首先在 Hive 中创建表,，并导入数据。

一共有3张表： 1张用户行为表，1张城市表，1 张产品表

```sql
CREATE TABLE `user_visit_action`(
  `date` string,
  `user_id` bigint,
  `session_id` string,
  `page_id` bigint,
  `action_time` string,
  `search_keyword` string,
  `click_category_id` bigint,
  `click_product_id` bigint,
  `order_category_ids` string,
  `order_product_ids` string,
  `pay_category_ids` string,
  `pay_product_ids` string,
  `city_id` bigint)
row format delimited fields terminated by '\t';
load data local inpath 'input/user_visit_action.txt' into table user_visit_action;

CREATE TABLE `product_info`(
  `product_id` bigint,
  `product_name` string,
  `extend_info` string)
row format delimited fields terminated by '\t';
load data local inpath 'input/product_info.txt' into table product_info;

CREATE TABLE `city_info`(
  `city_id` bigint,
  `city_name` string,
  `area` string)
row format delimited fields terminated by '\t';
load data local inpath 'input/city_info.txt' into table city_info;
```

## 2 需求：各区域热门商品 Top3

### 2.1 需求简介

> ​	这里的热门商品是从点击量的维度来看的，计算各个区域前三大热门商品，并备注上每个商品在主要城市中的分布比例，超过两个城市用其他显示。

例如：

| 地区 | 商品名称 | 点击次数 | 城市备注                        |
| ---- | -------- | -------- | ------------------------------- |
| 华北 | 商品A    | 100000   | 北京21.2%，天津13.2%，其他65.6% |
| 华北 | 商品P    | 80200    | 北京63.0%，太原10%，其他27.0%   |
| 华北 | 商品M    | 40000    | 北京63.0%，太原10%，其他27.0%   |
| 东北 | 商品J    | 92000    | 大连28%，辽宁17.0%，其他 55.0%  |

### 2.2 需求分析

- 查询出来所有的点击记录，并与 city_info 表连接，得到每个城市所在的地区，与 Product_info 表连接得到产品名称

- 按照地区和商品 id 分组，统计出每个商品在每个地区的总点击次数

- 每个地区内按照点击次数降序排列

- 只取前三名

- 城市备注需要自定义 UDAF 函数

### 2.3 功能实现

- 连接三张表的数据，获取完整的数据（只有点击）

- 将数据根据地区，商品名称分组

- 统计商品点击次数总和,取Top3

- 实现自定义聚合函数显示备注

**代码实现：**

```scala
package com.iisrun.spark.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

/**
 * 数据生成
 */
object SparkSQL13_Req_Mock {
  def main(args: Array[String]): Unit = {
    System.setProperty("HADOOP_USER_NAME","atguigu")
    // TODO 创建环境对象
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL")

    // TODO 访问外置的Hive
    val spark = SparkSession.builder()
      .enableHiveSupport()
      .config(sparkConf)
      .getOrCreate()
    import spark.implicits._

    spark.sql("use atguigu200213")

    spark.sql("""CREATE TABLE `user_visit_action`(
                |  `date` string,
                |  `user_id` bigint,
                |  `session_id` string,
                |  `page_id` bigint,
                |  `action_time` string,
                |  `search_keyword` string,
                |  `click_category_id` bigint,
                |  `click_product_id` bigint,
                |  `order_category_ids` string,
                |  `order_product_ids` string,
                |  `pay_category_ids` string,
                |  `pay_product_ids` string,
                |  `city_id` bigint)
                |row format delimited fields terminated by '\t'
                |""".stripMargin)
    spark.sql(
      """
        |load data local inpath 'input/user_visit_action.txt' into table user_visit_action
        |""".stripMargin)

    spark.sql(
      """
        |CREATE TABLE `product_info`(
        |  `product_id` bigint,
        |  `product_name` string,
        |  `extend_info` string)
        |row format delimited fields terminated by '\t'
        |""".stripMargin)
    spark.sql(
      """
        |load data local inpath 'input/product_info.txt' into table product_info
        |""".stripMargin)

    spark.sql(
      """
        |CREATE TABLE `city_info`(
        |  `city_id` bigint,
        |  `city_name` string,
        |  `area` string)
        |row format delimited fields terminated by '\t'
        |""".stripMargin)

    spark.sql(
      """
        |load data local inpath 'input/city_info.txt' into table city_info
        |""".stripMargin)

    spark.sql(
      """
        |select * from city_info
            """.stripMargin).show(10)
    spark.stop()
  }
}
```

```scala
package com.iisrun.spark.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession
/**
 * 完成：
 *    1. 连接三张表的数据，获取完整的数据（只有点击）
 *    2. 将数据根据地区，商品名称分组
 *	  3. 统计商品点击次数总和,取Top3
 */
object SparkSQL14_Req {
  def main(args: Array[String]): Unit = {
    System.setProperty("HADOOP_USER_NAME","atguigu")
    // TODO 创建环境对象
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL")
    // builder 构建，创建

    // TODO 访问外置的Hive
    val spark = SparkSession.builder()
      .enableHiveSupport()
      .config(sparkConf)
      .getOrCreate()

    spark.sql("use atguigu200213")
    spark.sql(
      """
        |select
        |   *
        |from (
        |    select
        |        *,
        |        rank() over( partition by area order by clickCount desc ) as rank
        |    from (
        |        select
        |            area,
        |            product_name,
        |            count(*) as clickCount
        |        from (
        |            select
        |               a.*,
        |               c.area,
        |               p.product_name
        |            from user_visit_action a
        |            join city_info c on c.city_id = a.city_id
        |            join product_info p on p.product_id = a.click_product_id
        |            where a.click_product_id > -1
        |        ) t1 group by area, product_name
        |    ) t2
        |) t3
        |where rank <= 3
            """.stripMargin).show

    spark.stop()
  }
}
```

```scala
package com.iisrun.spark.sql

import org.apache.spark.SparkConf
import org.apache.spark.sql.{Encoder, Row, SparkSession}
import org.apache.spark.sql.expressions.{Aggregator, MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types.{DataType, LongType, MapType, StringType, StructField, StructType}

/**
 * 完成：
 *    1. 连接三张表的数据，获取完整的数据（只有点击）
 *    2. 将数据根据地区，商品名称分组
 *	  3. 统计商品点击次数总和,取Top3
 *    4. 实现自定义聚合函数显示备注
 *  使用临时表将一个SQL拆分成多个临时表，增加代码的阅读性。
 */
object SparkSQL15_Req1 {
  def main(args: Array[String]): Unit = {
    System.setProperty("HADOOP_USER_NAME","atguigu")
    // TODO 创建环境对象
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL")

    // builder 构建，创建
    // TODO 访问外置的Hive
    val spark = SparkSession.builder()
      .enableHiveSupport()
      .config(sparkConf)
      .getOrCreate()

    import spark.implicits._

    spark.sql("use atguigu200213")

    // TODO 从hive表中获取满足条件的数据
    spark.sql(
      """
        |select
        |   a.*,
        |   c.area,
        |   c.city_name,
        |   p.product_name
        |from user_visit_action a
        |join city_info c on c.city_id = a.city_id
        |join product_info p on p.product_id = a.click_product_id
        |where a.click_product_id > -1
        |""".stripMargin).createOrReplaceTempView("t1")

    // TODO 将数据根据区域和商品进行分组，统计商品点击的数量
    // 北京，上海，北京，深圳
    // **********************************
    // in：cityname:String
    // buffer：要有两个结构，(total,map)
    // out：remark:String
    // (商品点击总和，每个城市点击总和)
    // (商品点击总和，Map(城市，点击Sum))
    // 城市点击sum / 商品点击总和 %
    // TODO 创建自定义聚合函数
    val udaf = new CityRemarkUDAF

    // TODO 注册聚合函数
    spark.udf.register("cityRemark",udaf);
    spark.sql(
      """
        |select
        |	area,
        |	product_name,
        |	count(*) as clickCount,
        | cityRemark(city_name)
        |from
        |	 t1
        |group by area, product_name
        |""".stripMargin).createOrReplaceTempView("t2")

    // TODO 将统计结果根据数量进行排序（降序）
    spark.sql(
      """
        |select
        |	*,
        |	rank() over( partition by area order by clickCount desc ) as rank
        |from
        |	t2
        |""".stripMargin)createOrReplaceTempView("t3")

    // TODO 取前三名
    spark.sql(
      """
        |select
        |   *
        |from t3
        |where rank <= 3
            """.stripMargin).show
    spark.stop()
  }


  class CityRemarkUDAF extends UserDefinedAggregateFunction {
    // TODO 输入的数据其实就是城市名称
    override def inputSchema: StructType = {
      StructType(Array(StructField("cityName",StringType)))
    }

    // TODO 缓冲区中的数据应该为：totalcnt，Map(cityName，cnt)
    override def bufferSchema: StructType = {
      StructType(Array(
        StructField("totalcnt", LongType),
        StructField("citymap", MapType(StringType, LongType))
      ))
    }

    // TODO 返回城市备注的字符串
    override def dataType: DataType = {
      StringType
    }

    override def deterministic: Boolean = true

    // TODO 缓冲区的初始化
    override def initialize(buffer: MutableAggregationBuffer): Unit = {
      buffer(0) = 0L
      buffer(1) = Map[String, Long]()
//      buffer.update(0,0L) 也可以用这种方法
    }

    // TODO 更新缓冲区
    override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
      val cityName: String = input.getString(0)
      // 点击总和需要增加
      buffer(0) = buffer.getLong(0) + 1
      // 城市点击增加
      val citymap: Map[String, Long] = buffer.getAs[Map[String, Long]](1)

      val newClickCount = citymap.getOrElse(cityName, 0L) + 1

      buffer(1) = citymap.updated(cityName, newClickCount)
    }

    // TODO 合并缓冲区
    override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
      // 合并点击数量总和
      buffer1(0) = buffer1.getLong(0) + buffer2.getLong(0)

      //  合并城市点击map
      val map1: Map[String, Long] = buffer1.getAs[Map[String, Long]](1)
      val map2: Map[String, Long] = buffer2.getAs[Map[String, Long]](1)

      buffer1(1) = map1.foldLeft(map2) {
        case (map, (k, v)) => {
          map.updated(k, map.getOrElse(k, 0L) + v)
        }
      }
    }

    // TODO 对缓冲区进行计算并返回
    override def evaluate(buffer: Row): Any = {
      val totalcount: Long = buffer.getLong(0)
      val citymap: collection.Map[String, Long] = buffer.getMap[String,Long](1)

      val cityToCountList: List[(String, Long)] = citymap.toList.sortWith(
        (left, right) => {
          left._2 > right._2
        }
      ).take(2)

      val hasRest = citymap.size>2
      var rest = 0L

      val s = new StringBuilder
      cityToCountList.foreach{
        case (city,cnt)=>{
          val r = (cnt * 100 / totalcount)
          s.append(city + " " +  r+ "%,")
          rest = rest+r
        }
      }
      s.toString()+" 其他：" + (100-rest) + "%"
//      if (hasRest) {
//        s.toString()+" 其他：" + (100-rest) + ""
//      } else   {
//        s.toString()
//      }
    }
  }
}
```

