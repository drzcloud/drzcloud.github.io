---
layout: post
title: Spark入门：01-Spark快速上手
category: spark
tags: [spark]
excerpt: Spark快速上手
lock: need
---

> ​	 在大数据早期的课程中我们已经学习了MapReduce框架的原理及基本使用，并了解了其底层数据处理的实现方式。接下来，咱们就走进Spark的世界，了解一下它是如何带领我们完成数据处理的。

## 2.1 创建Maven项目

### 2.1.1 增加Scala插件

> ​    Spark由Scala语言开发的，所以本课件接下来的开发所使用的语言也为Scala，咱们当前使用的Spark版本为2.4.5，默认采用的Scala版本为2.12，所以后续开发时。我们依然采用这个版本。开发前请保证IDEA开发工具中含有Scala开发插件

![image-20200604165518751](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604165518.png)

### 2.1.1 增加依赖关系

> 修改Maven项目中的POM文件，增加Spark框架的依赖关系。本课件基于Spark2.4.5版本，使用时请注意对应版本。

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_2.12</artifactId>
        <version>2.4.5</version>
    </dependency>
</dependencies>
<build>
    <plugins>
        <!-- 该插件用于将Scala代码编译成class文件 -->
        <plugin>
            <groupId>net.alchim31.maven</groupId>
            <artifactId>scala-maven-plugin</artifactId>
            <version>3.2.2</version>
            <executions>
                <execution>
                    <!-- 声明绑定到maven的compile阶段 -->
                    <goals>
                        <goal>testCompile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.0.0</version>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 2.1.3 WordCount

> 为了能直观地感受Spark框架的效果，接下来我们实现一个大数据学科中最常见的案例WordCount。

```scala
// 创建Spark运行配置对象
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

// 创建Spark上下文环境对象（连接对象）
val sc : SparkContext = new SparkContext(sparkConf)

// 读取文件数据
val fileRDD: RDD[String] = sc.textFile("input/word.txt")

// 将文件中的数据进行分词
val wordRDD: RDD[String] = fileRDD.flatMap( _.split(" ") )

// 转换数据结构 word => (word, 1)
val word2OneRDD: RDD[(String, Int)] = wordRDD.map((_,1))

// 将转换结构后的数据按照相同的单词进行分组聚合
val word2CountRDD: RDD[(String, Int)] = word2OneRDD.reduceByKey(_+_)

// 将数据聚合结果采集到内存中
val word2Count: Array[(String, Int)] = word2CountRDD.collect()

// 打印结果
word2Count.foreach(println)

//关闭Spark连接
sc.stop()
```

执行过程中，会产生大量的执行日志，如果为了能够看好的查看程序的执行结果，可以在项目的resources目录中创建log4j.properties文件，并添加日志配置信息：

```properties
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n

# Set the default spark-shell log level to ERROR. When running the spark-shell, the
# log level for this class is used to overwrite the root logger's log level, so that
# the user can have different defaults for the shell and regular Spark apps.
log4j.logger.org.apache.spark.repl.Main=ERROR

# Settings to quiet third party logs that are too verbose
log4j.logger.org.spark_project.jetty=ERROR
log4j.logger.org.spark_project.jetty.util.component.AbstractLifeCycle=ERROR
log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=ERROR
log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=ERROR
log4j.logger.org.apache.parquet=ERROR
log4j.logger.parquet=ERROR

# SPARK-9183: Settings to avoid annoying messages when looking up nonexistent UDFs in SparkSQL with Hive support
log4j.logger.org.apache.hadoop.hive.metastore.RetryingHMSHandler=FATAL
log4j.logger.org.apache.hadoop.hive.ql.exec.FunctionRegistry=ERROR
```

### 2.1.4 异常处理

如果本机操作系统是Windows，在程序中使用了Hadoop相关的东西，比如写入文件到HDFS，则会遇到如下异常：

![image-20200604165830665](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604165830.png)

出现这个问题的原因，并不是程序的错误，而是windows系统用到了hadoop相关的服务，解决办法是通过配置关联到windows的系统依赖就可以了

![image-20200604165851327](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604165851.png)

在IDEA中配置Run Configuration，添加HADOOP_HOME变量

![image-20200604165908246](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604165908.png)

![image-20200604165914266](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604165914.png)