---
layout: post
title: Spark入门：03-Spark运行环境
category: spark
tags: [spark]
excerpt: Spark作为一个数据处理框架和计算引擎，被设计在所有常见的集群环境中运行, 在国内工作中主流的环境为Yarn，不过逐渐容器式环境也慢慢流行起来。接下来，我们就分别看看不同环境下Spark的运行...
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
[jack@hadoop112 module]$ tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
[jack@hadoop112 module]$ cd /opt/module 
[jack@hadoop112 software]$ mv spark-2.4.5-bin-without-hadoop-scala-2.12 spark-standalone
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
[jack@hadoop112 spark-standalone]$ sbin/start-all.sh	# 启动命令
[jack@hadoop112 spark-standalone]$ sbin/stop-all.sh		# 停止命令
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
[jack@hadoop112 spark-standalone]$ pwd
/opt/module/spark-standalone
[jack@hadoop112 spark-standalone]$ bin/spark-submit \   # 提交应用
--class org.apache.spark.examples.SparkPi \				# 表示要执行程序的主类
--master spark://hadoop112:7077 						# 独立运行模式，7070为spark内部通信的端口
spark-examples_2.12-2.4.5.jar   						# 程序主类所在的jar
10  													# 设定当前应用的任务数量
```

执行任务时，会产生多个Java进程

![image-20200604205948798](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604205948.png)

执行任务时，默认采用服务器集群节点的总核数，每个节点内存1024M

![image-20200604210020274](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604210020.png)

### 3.2.5 提交参数说明

| 参数                     | 解释                                                         | 可选值举例                                    |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| --class                  | Spark程序中包含主函数的类                                    | 本地模式：local[*]、spark://linux1:7077、Yarn |
| --master                 | Spark程序运行的模式(环境)                                    | 符合集群内存配置即可，具体情况具体分析。      |
| --executor-memory 1G     | 指定每个executor可用内存为1G                                 | 符合集群内存配置即可，具体情况具体分析。      |
| --total-executor-cores 2 | 指定所有executor使用的cpu核数为2个                           | 符合集群内存配置即可，具体情况具体分析。      |
| --executor-cores         | 指定每个executor使用的cpu核数                                | 符合集群内存配置即可，具体情况具体分析。      |
| application-jar          | 打包好的应用jar，包含依赖。这个URL在集群<br />中全局可见。 比如hdfs:// 共享存储系统，<br />如果是file:// path，那么所有的节点的path都包含同样的jar | 符合集群内存配置即可，具体情况具体分析。      |
| application-arguments    | 传给main()方法的参数                                         | 符合集群内存配置即可，具体情况具体分析。      |

### 3.2.6 配置历史服务

由于spark-shell停止掉后，集群监控linux1:4040页面就看不到历史任务的运行情况，所以开发时都配置历史服务器记录任务运行情况。

1） 修改spark-defaults.conf.template文件名为spark-defaults.conf

```shell
[jack@hadoop112 conf]$ mv spark-defaults.conf.template spark-defaults.conf
```

2）修改spark-default.conf文件，配置日志存储路径

```
spark.eventLog.enabled          true
spark.eventLog.dir              hdfs://hadoop112:8020/directory
```

==注意：需要启动hadoop集群，HDFS上的directory目录需要提前存在。==

```
sbin/start-dfs.sh
hadoop fs -mkdir /directory
```

3）修改spark-env.sh文件, 添加日志配置

```
export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080 
-Dspark.history.fs.logDirectory=hdfs://hadoop112:8020/directory 
-Dspark.history.retainedApplications=30"
```

- 参数1含义：WEB UI访问的端口号为18080

- 参数2含义：指定历史服务器日志存储路径

- 参数3含义：指定保存Application历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数。

4）分发配置文件

```
xsync conf
```

5）重新启动集群和历史服务

```shell
[jack@hadoop112 spark-standalone]$ sbin/start-all.sh
[jack@hadoop112 spark-standalone]$ sbin/start-history-server.sh
```

6） 重新执行任务

```shell
[jack@hadoop112 spark-standalone]$ bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop112:7077 \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
```

![image-20200604203853214](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604203853.png)

7）查看历史服务：[http://hadoop112:18080](http://hadoop112:18080)

![image-20200604210136013](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604210136.png)

### 3.2.7 配置高可用（HA）

> ​	所谓的高可用是因为当前集群中的Master节点只有一个，所以会存在单点故障问题。所以为了解决单点故障问题，需要在集群中配置多个Master节点，一旦处于活动状态的Master发生故障时，由备用Master提供服务，保证作业可以继续执行。这里的高可用一般采用Zookeeper设置

集群规划：

|       | hadoop112                         | hadoop113                         | hadoop114             |
| ----- | --------------------------------- | --------------------------------- | --------------------- |
| Spark | Master<br />Zookeeper<br />Worker | Master<br />Zookeeper<br />Worker | Zookeeper<br />Worker |

1) 停止集群

```shell
[jack@hadoop112 spark-standalone]$ sbin/stop-all.sh 
```

2) 启动Zookeeper

```
myzk start
```

3) 修改spark-env.sh文件添加如下配置

```shell
#注释如下内容：
#SPARK_MASTER_HOST=linux1
#SPARK_MASTER_PORT=7077
SPARK_MASTER_WEBUI_PORT=8989

#添加如下内容:
export SPARK_DAEMON_JAVA_OPTS="
-Dspark.deploy.recoveryMode=ZOOKEEPER 
-Dspark.deploy.zookeeper.url=linux1,linux2,linux3 
-Dspark.deploy.zookeeper.dir=/spark"
```

4) 分发配置文件

```
xsync conf/ 
```

5) 启动集群

```shell
[jack@hadoop112 spark-standalone]$ sbin/start-all.sh 
```

6) 启动lhadoop106的单独Master节点，此时hadoop106节点Master状态处于备用状态

![image-20200604211754070](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604211754.png)

7) 停止hadoop105的Master资源监控进程

```shell
[jack@hadoop112 spark-standalone]$ kill -9 Master进程号
```

8) 查看hadoop112的Master 资源监控Web UI，稍等一段时间后，hadoop113节点的Master状态提升为活动状态

![image-20200604211907537](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604211907.png)

## 3.3 Yarn 模式

> ​	独立部署（Standalone）模式由Spark自身提供计算资源，无需其他框架提供资源。这种方式降低了和其他第三方资源框架的耦合性，独立性非常强。但是你也要记住，Spark主要是计算框架，而不是资源调度框架，所以本身提供的资源调度并不是它的强项，所以还是和其他专业的资源调度框架集成会更靠谱一些。所以接下来我们来学习在强大的Yarn环境下Spark是如何工作的（其实是因为在国内工作中，Yarn使用的非常多）。

### 3.3.1 解压缩文件

将spark-2.4.5-bin-without-hadoop-scala-2.12.tgz文件上传到Linux并解压缩在指定位置

```shell
[jack@hadoop112 module]$ tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
[jack@hadoop112 module]$ cd /opt/module 
[jack@hadoop112 software]$ mv spark-2.4.5-bin-without-hadoop-scala-2.12 spark-yarn
```

==spark2.4.5默认不支持Hadoop3，可以采用多种不同的方式关联Hadoop3。==

**方式一：**修改spark-local/conf/spark-env.sh文件，增加如下内容：

```
SPARK_DIST_CLASSPATH=$(/opt/module/hadoop3/bin/hadoop classpath)
```

**方式二：**也可以直接引入对应的Jar包，上传到spark-local/jars目录即可。

![image-20200604170710823](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604170710.png)

### 3.3.2 修改配置文件

1) 修改hadoop配置文件/opt/module/hadoop/etc/hadoop/yarn-site.xml, 并分发

```xml
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
     <name>yarn.nodemanager.pmem-check-enabled</name>
     <value>false</value>
</property>

<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
     <name>yarn.nodemanager.vmem-check-enabled</name>
     <value>false</value>
</property>
```

2) 修改conf/spark-env.sh，添加JAVA_HOME和YARN_CONF_DIR配置

```shell
mv spark-env.sh.template spark-env.sh

export JAVA_HOME=/opt/module/jdk1.8.0_212
YARN_CONF_DIR=/opt/module/hadoop-3.1.3/etc/hadoop
```

### 3.3.3 启动HDFS以及YARN集群

```shell
# 使用自定义启动命令启动HDFS及YARN
[jps@hadoop112 conf]$ mycluster start
```

### 3.3.4 提交应用

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
```

1111111111

查看http://hadoop113:8088页面，点击History，查看历史页面

![image-20200604213535767](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604213535.png)

### 3.3.5 配置历史服务器

1) 修改spark-defaults.conf.template文件名为spark-defaults.conf

```shell
[jack@hadoop112 conf]# mv spark-defaults.conf.template spark-defaults.conf
```

2) 修改spark-default.conf文件，配置日志存储路径

```shell
spark.eventLog.enabled          true
spark.eventLog.dir              hdfs://hadoop112:8020/directory # 也可能是9820，根据hadoop配置设置
```

注意：需要启动hadoop集群，HDFS上的目录需要提前存在。

```shell
[jack@hadoop112 hadoop]# sbin/start-dfs.sh
[jack@hadoop112 hadoop]# hadoop fs -mkdir /directory
```

3) 修改spark-env.sh文件, 添加日志配置

```shell
export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080 
-Dspark.history.fs.logDirectory=hdfs://hadoop112:8020/directory 
-Dspark.history.retainedApplications=30"
```

- 参数1含义：WEB UI访问的端口号为18080
- 参数2含义：指定历史服务器日志存储路径
- 参数3含义：指定保存Application历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数。

4) 修改spark-defaults.conf

```shell
spark.yarn.historyServer.address=linux1:18080
spark.history.ui.port=18080
```

5) 启动历史服务

```shell
sbin/start-history-server.sh
```

6) 重新提交应用

```shell
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
```

7) Web页面查看日志：http://hadoop113:8088

![image-20200604213535767](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604213535.png)

## 3.4  K8S & Mesos模式

>    Mesos是Apache下的开源分布式资源管理框架，它被称为是分布式系统的内核,在Twitter得到广泛使用,管理着Twitter超过30,0000台服务器上的应用部署，但是在国内，依然使用着传统的Hadoop大数据框架，所以国内使用Mesos框架的并不多，但是原理其实都差不多，这里我们就不做过多讲解了。

![image-20200604214339465](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604214339.png)

​    容器化部署是目前业界很流行的一项技术，基于Docker镜像运行能够让用户更加方便地对应用进行管理和运维。容器管理工具中最为流行的就是Kubernetes（k8s），而Spark也在最近的版本中支持了k8s部署模式。这里我们也不做过多的讲解。给个链接大家自己感受一下：https://spark.apache.org/docs/latest/running-on-kubernetes.html

![image-20200604214411295](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604214411.png)

## 3.5  Windows模式

> ​	在我们自己学习时，每次都需要启动虚拟机，启动集群，这是一个比较繁琐的过程，并且会占大量的系统资源，导致系统执行变慢，不仅仅影响学习效果，也影响学习进度，Spark非常暖心地提供了可以在windows系统下启动本地集群的方式，这样，在不使用虚拟机的情况下，也能学习Spark的基本使用

### 3.5.1 解压缩文件
将文件spark-2.4.5-bin-without-hadoop-scala-2.12.tgz解压缩到无中文无空格的路径中，将hadoop3依赖jar包拷贝到jars目录中。

![image-20200604214644427](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604214644.png)

### 3.5.2 启动本地环境

执行解压缩文件路径下bin目录中的spark-shell.cmd文件，启动Spark本地环境

```
sc.textFile("input/word.txt").flatMap(_.split(",")).map((_,1)).reduceByKey(_+_).collect
```

![image-20200604214728159](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604214728.png)

### 3.5.3 命令行提交应用

![image-20200604214831999](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200604214832.png)

## 3.6  部署模式对比

| 模式       | Spark安装机器数 | 需启动的进程   | 所属者 | 应用场景 |
| ---------- | --------------- | -------------- | ------ | -------- |
| Local      | 1               | 无             | Spark  | 测试     |
| Standalone | 3               | Master及Worker | Spark  | 单独部署 |
| Yarn       | 1               | Yarn及HDFS     | Hadoop | 混合部署 |

## 3.7  端口号

- Spark查看当前Spark-shell运行任务情况端口号：4040（计算）
- Spark Master内部通信服务端口号：7077
- Standalone模式下，Spark Master Web端口号：8080（资源）
- Spark历史服务器端口号：18080
- Hadoop YARN任务运行情况查看端口号：8088