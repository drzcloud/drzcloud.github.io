---
layout: post
title: Spark入门：03-Spark运行环境
category: spark
tags: [spark]
excerpt: Spark运行环境
lock: need
---

> ​	 Spark作为一个数据处理框架和计算引擎，被设计在所有常见的集群环境中运行, 在国内工作中主流的环境为Yarn，不过逐渐容器式环境也慢慢流行起来。接下来，我们就分别看看不同环境下Spark的运行

![image-20200604170028690](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604170028.png)

## 3.1 Local模式

### 3.1.1 解压缩文件

将spark-2.4.5-bin-without-hadoop-scala-2.12.tgz文件上传到Linux并解压缩，放置在指定位置，路径中不要包含中文或空格，后续如果涉及到解压缩操作，不再强调。

```shell
[jack@hadoop112 ~]$ tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
[jack@hadoop112 ~]$ cd /opt/module 
[jack@hadoop112 ~]$ mv spark-2.4.5-bin-without-hadoop-scala-2.12 spark-local
```

==spark2.4.5默认不支持Hadoop3，可以采用多种不同的方式关联Hadoop3。==

**方式一：**修改spark-local/conf/spark-env.sh文件，增加如下内容：

```
SPARK_DIST_CLASSPATH=$(/opt/module/hadoop3/bin/hadoop classpath)
```

**方式二：**也可以直接引入对应的Jar包，上传到spark-local/jars目录即可。

![image-20200604170710823](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604170710.png)

### 3.1.2 启动Local环境

1)  进入解压缩后的路径，执行如下指令

```shell
[jack@hadoop112 spark-local]$ pwd
/opt/module/spark-local
[jack@hadoop112 spark-local]$ bin/spark-shell --master local[*]
```

![image-20200604170844756](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604170844.png)

2) 启动成功后，可以输入网址进行Web UI监控页面访问

```
http://虚拟机地址:4040
```

![image-20200604170914380](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604170914.png)

### 3.1.3 命令行工具

​	在解压缩文件夹下的data目录中，添加word.txt文件。在命令行工具中执行如下代码指令（和IDEA中代码简化版一致）

```shell
sc.textFile("data/word.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect
```

![image-20200604171010572](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604171010.png)

### 3.1.4 退出本地模式

按键Ctrl+C或输入Scala指令

```
:quit
```

### 3.1.5 提交应用

```shell
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master local[2] \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
```

1)  --class表示要执行程序的主类

2)  --master local[2] 部署模式，默认为本地模式，数字表示分配的虚拟CPU核数量

3)  spark-examples_2.12-2.4.5.jar 运行的应用类所在的jar包

4)  数字10表示程序的入口参数，用于设定当前应用的任务数量

![image-20200604171557932](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604171558.png)

## 3.2 Standalone模式

> local本地模式毕竟只是用来进行练习演示的，真实工作中还是要将应用提交到对应的集群中去执行，这里我们来看看只使用Spark自身节点运行的集群模式，也就是我们所谓的独立部署（Standalone）模式。Spark的Standalone模式体现了经典的master-slave模式。

集群规划：

|       | Linux1             | Linux2 | Linux3 |
| ----- | ------------------ | ------ | ------ |
| Spark | Worker  **Master** | Worker | Worker |

### 3.1.1 解压缩文件

将spark-2.4.5-bin-without-hadoop-scala-2.12.tgz文件上传到Linux并解压缩在指定位置

```shell
[jack@hadoop112 ~]$ tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
[jack@hadoop112 ~]$ cd /opt/module 
[jack@hadoop112 ~]$ mv spark-2.4.5-bin-without-hadoop-scala-2.12 spark-standalone
```

==spark2.4.5默认不支持Hadoop3，可以采用多种不同的方式关联Hadoop3。==

**方式一：**修改spark-local/conf/spark-env.sh文件，增加如下内容：

```
SPARK_DIST_CLASSPATH=$(/opt/module/hadoop3/bin/hadoop classpath)
```

**方式二：**也可以直接引入对应的Jar包，上传到spark-local/jars目录即可。

![image-20200604170710823](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604170710.png)

### 3.2.2 修改配置文件

1）进入解压缩后路径的conf目录，修改slaves.template文件名为slaves

```shell
[jack@hadoop112 conf]$ pwd
/opt/module/spark-standalone/conf
[jack@hadoop112 conf]$ mv slaves.template slaves
```

2）修改slaves文件，添加work节点

```
hadoop112
hadoop113
hadoop114
```

3）修改spark-env.sh.template文件名为spark-env.sh

```shell
[jack@hadoop112 conf]$ mv spark-env.sh.template spark-env.sh
```

4）修改spark-env.sh文件，添加JAVA_HOME环境变量和集群对应的master节点

```shell
export JAVA_HOME=/opt/module/jdk1.8.0_144
SPARK_MASTER_HOST=hadoop112
SPARK_MASTER_PORT=7077
```

==注意：7077端口，相当于hadoop3内部通信的8020端口==

5）分发spark-standalone目录

```shell
[jack@hadoop112 module]$ pwd
/opt/module
[jack@hadoop112 module]$ xsync spark-standalone
```

### 3.2.3 启动集群

1）执行脚本命令：

```shell
[jack@hadoop112 spark-standalone]$ pwd
/opt/module/spark-standalone
[jack@hadoop112 spark-standalone]$ sbin/start-all.sh
```

2）查看三台服务器运行进程

分别登陆三台服务器使用jps命令查看，也可以直接使用自定义命令myjps查询集群服务状态。

```
================hadoop112================
3330 Jps
3238 Worker
3163 Master
================hadoop113================
2966 Jps
2908 Worker
================hadoop114================
2978 Worker
3036 Jps
```

3)）查看Master资源监控Web UI界面: [http://hadoop112:8080](http://hadoop112:8080)

![image-20200604173020876](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604173021.png)

### 3.2.4 提交应用

```shell
[jack@hadoop112 spark-standalone]$ bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop112:7077 \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
```



## 3.3 Yarn 模式

