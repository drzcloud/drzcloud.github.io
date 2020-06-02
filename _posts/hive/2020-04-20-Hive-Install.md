---
layout: post
title: Hive入门：01-Hive安装部署
category: hive
tags: [hive]
excerpt: Hive安装部署
lock: need
---


**1）Hive官网地址**

http://hive.apache.org/

**2）文档查看地址**

https://cwiki.apache.org/confluence/display/Hive/GettingStarted

**3）下载地址**

http://archive.apache.org/dist/hive/

**4）github地址**

https://github.com/apache/hive

### 2 MySql安装

**1）检查当前系统是否安装过Mysq**

```shell
[jack@hadoop102 ~]$ rpm -qa|grep mariadb
mariadb-libs-5.5.56-2.el7.x86_64 //如果存在通过如下命令卸载
[jack @hadoop102 ~]$ sudo rpm -e --nodeps  mariadb-libs   //用此命令卸载mariadb
```

**2）将MySQL安装包拷贝到/opt/software目录下**

```shell
[jack @hadoop102 software]# ll
总用量 528384
-rw-r--r--. 1 root root 609556480 3月  21 15:41 mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
```

**3）解压MySQL安装包**

```shell
[jack @hadoop102 software]# mkdir mysql
[jack @hadoop102 software]# tar -xf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar -C mysql/
```

![image-20200424230021789](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200424230023.png)

**4）在安装目录下执行rpm安装**

```shell
sudo rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
```

**注意:按照顺序依次执行!!!**

如果Linux是最小化安装的，在安装mysql-community-server-5.7.28-1.el7.x86_64.rpm时可能会出现如下错误：

```shell
[jack@hadoop102 mysql]$ sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
警告：mysql-community-server-5.7.28-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
错误：依赖检测失败：
        libaio.so.1()(64bit) 被 mysql-community-server-5.7.28-1.el7.x86_64 需要
        libaio.so.1(LIBAIO_0.1)(64bit) 被 mysql-community-server-5.7.28-1.el7.x86_64 需要
        libaio.so.1(LIBAIO_0.4)(64bit) 被 mysql-community-server-5.7.28-1.el7.x86_64 需要
```

通过yum安装缺少的依赖,然后重新安装mysql-community-server-5.7.28-1.el7.x86_64 即可

```shell
[jack@hadoop102 software] yum install -y libaio
```

**5）删除/etc/my.cnf文件中datadir指向的目录下的所有内容,如果有内容的情况下：**

​	查看datadir的值：

```
[mysqld]
datadir=/var/lib/mysql
```

​	删除/var/lib/mysql目录下的所有内容：

```shell
[jack @hadoop102 mysql]# cd /var/lib/mysql
[jack @hadoop102 mysql]# sudo rm -rf /*    //注意执行命令的位置
```

**6）初始化数据库**

```shell
[jack @hadoop102 opt]$ sudo mysqld --initialize --user=mysql
```

**7）查看临时生成的root用户的密码** 

```shell
[jack @hadoop102 opt]$ cat /var/log/mysqld.log 
```

![image-20200424230800531](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200424230801.png)

**8）启动MySQL服务**

```
[jack @hadoop102 opt]$ sudo systemctl start mysqld
```

**9）登录MySQL数据库**

```
[jack @hadoop102 opt]$ mysql -uroot -p
Enter password:   输入临时生成的密码   
```
登录成功.

**10）必须先修改root用户的密码,否则执行其他的操作会报错**

```
mysql> set password = password("新密码")
```

**11）修改mysql库下的user表中的root用户允许任意ip连接**

```
mysql> update mysql.user set host='%' where user='root';
mysql> flush privileges;
```

### 3 Hive安装部署

**1）把apache-hive-3.1.2-bin.tar.gz上传到linux的/opt/software目录下**

**2）解压apache-hive-3.1.2-bin.tar.gz到/opt/module/目录下面**

```shell
[jack@hadoop102 software]$ tar -zxvf /opt/software/apache-hive-3.1.2-bin.tar.gz -C /opt/module/
```

**3）修改apache-hive-3.1.2-bin.tar.gz的名称为hive**

```shell
[jack@hadoop102 software]$ mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive
```

**4）修改/etc/profile.d/my_env.sh，添加环境变量**

```shell
[jack@hadoop102 software]$ sudo vim /etc/profile.d/my_env.sh
```

**5）添加Hive环境变量**

```shell
#JAVA_HOME
JAVA_HOME=/opt/module/jdk1.8.0_212

#HADOOP_HOME
HADOOP_HOME=/opt/module/hadoop-3.1.3

#HIVE_HOME
HIVE_HOME=/opt/module/hive

PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin
export PATH JAVA_HOME HADOOP_HOME HIVE_HOME
```

**6）解决日志Jar包冲突**

```shell
[jack@hadoop102 software]$ mv $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.jar $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.bak
```

### 4 Hive元数据配置到MySql

#### 4.1 拷贝驱动

将MySQL的JDBC驱动拷贝到Hive的lib目录下

```shell
[jack@hadoop102 software]$ cp /opt/software/mysql-connector-java-5.1.48.jar $HIVE_HOME/lib
```

#### 4.2 配置Metastore到MySql

在$HIVE_HOME/conf目录下新建hive-site.xml文件

```shell
[jack@hadoop102 software]$ vim $HIVE_HOME/conf/hive-site.xml
```

添加如下内容：

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <!-- jdbc连接的URL -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop112:3306/metastore?useSSL=false</value>
    </property>

    <!-- jdbc连接的Driver-->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    <!-- jdbc连接的username-->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <!-- jdbc连接的password -->
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
    <!-- Hive默认在HDFS的工作目录 -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
    
    <!-- Hive元数据存储版本的验证 -->
    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>
    <!-- 指定存储元数据要连接的地址 -->
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://hadoop112:9083</value>
    </property>
    <!-- 指定hiveserver2连接的端口号 -->
    <property>
    <name>hive.server2.thrift.port</name>
    <value>10000</value>
    </property>
    <!-- 指定hiveserver2连接的host -->
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>hadoop112</value>
    </property>
    <!-- 元数据存储授权  -->
    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
    <!-- 设置计算引擎为tez-->
    <property>
        <name>hive.execution.engine</name>
        <value>tez</value>
    </property>
    <!-- 设置tez容器大小为1024M -->
    <property>
        <name>hive.tez.container.size</name>
        <value>1024</value>
    </property>
    <!-- Hive方式访问时-打印当前库和表头 start-->
    <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
        <description>Whether to print the names of the columns in query output.</description>
    </property>
    <property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
        <description>Whether to include the current database in the Hive prompt.</description>
    </property>
    <!-- Hive方式访问时-打印当前库和表头 end-->
</configuration>
```

### 5 安装Tez引擎

Tez是一个Hive的运行引擎，性能优于MR。为什么优于MR呢？看下图。

![image-20200424232138338](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200424232139.png)

​	用Hive直接编写MR程序，假设有四个有依赖关系的MR作业，上图中，绿色是Reduce Task，云状表示写屏蔽，需要将中间结果持久化写到HDFS。

​	Tez可以将多个有依赖的作业转换为一个作业，这样只需写一次HDFS，且中间节点较少，从而大大提升作业的计算性能。

**1）将tez安装包拷贝到集群，并解压tar包**

```shell
[jack@hadoop102 software]$ mkdir /opt/module/tez
[jack@hadoop102 software]$ tar -zxvf /opt/software/tez-0.10.1-SNAPSHOT-minimal.tar.gz -C /opt/module/tez
```

**2）上传tez依赖到HDFS**

```shell
[jack@hadoop102 software]$ hadoop fs -mkdir /tez
[jack@hadoop102 software]$ hadoop fs -put /opt/software/tez-0.10.1-SNAPSHOT.tar.gz /tez
```

**3）新建tez-site.xml**

```shell
[jack@hadoop102 software]$ vim $HADOOP_HOME/etc/hadoop/tez-site.xml
```

添加如下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
    	<name>tez.lib.uris</name>
    	<value>${fs.defaultFS}/tez/tez-0.10.1-SNAPSHOT.tar.gz</value>
	</property>
	<property>
     	<name>tez.use.cluster.hadoop-libs</name>
     	<value>true</value>
	</property>
	<property>
     	<name>tez.am.resource.memory.mb</name>
     	<value>1024</value>
	</property>
    <property>
         <name>tez.am.resource.cpu.vcores</name>
         <value>1</value>
    </property>
    <property>
     	<name>tez.container.max.java.heap.fraction</name>
     	<value>0.4</value>
	</property>
	<property>
     	<name>tez.task.resource.memory.mb</name>
     	<value>1024</value>
	</property>
	<property>
     	<name>tez.task.resource.cpu.vcores</name>
     	<value>1</value>
	</property>
</configuration>
```

**4）修改Hadoop环境变量**

```
[jack@hadoop102 software]$ vim $HADOOP_HOME/etc/hadoop/shellprofile.d/tez.sh
```

添加Tez的Jar包相关信息

```shell
hadoop_add_profile tez
function _tez_hadoop_classpath
{
  hadoop_add_classpath "$HADOOP_HOME/etc/hadoop" after
  hadoop_add_classpath "/opt/module/tez/*" after
  hadoop_add_classpath "/opt/module/tez/lib/*" after
}
```

**5）修改Hive的计算引擎（上述配置中已经配置过了，这里可以省略）**

```shell
[jack@hadoop102 software]$ vim $HIVE_HOME/conf/hive-site.xml
```

添加：

```xml
<property>
    <name>hive.execution.engine</name>
    <value>tez</value>
</property>
<property>
    <name>hive.tez.container.size</name>
    <value>1024</value>
</property>
```

**6）解决日志Jar包冲突**

```shell
[jack@hadoop102 software]$ rm /opt/module/tez/lib/slf4j-log4j12-1.7.10.jar
```

### 6 启动Hive

#### 6.1 初始化元数据库

**1）登陆MySQL（密码写自己设置的）**

```shell
[jack@hadoop102 software]$ mysql -uroot -p123456
```

**2）新建Hive元数据库**

数据库名metastore和$HIVE_HOME/conf/hive-site.xml 中的保持一致即可。

```sql
mysql> create database metastore;
mysql> quit;
```

**3）初始化Hive元数据库**

```shell
[jack@hadoop102 software]$ schematool -initSchema -dbType mysql -verbose
```

#### 6.2 启动metastore和hiveserver2

**1）Hive 2.x以上版本，要先启动这两个服务，否则会报错：**

```
FAILED: HiveException java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
```

（1）启动metastore

```
[jack@hadoop202 hive]$ hive --service metastore 
2020-04-24 16:58:08: Starting Hive Metastore Server  
注意: 启动后窗口不能再操作，需打开一个新的shell窗口做别的操作
```

（2） 启动 hiveserver2

```
[jack@hadoop202 hive]$ hive --service hiveserver2
which: no hbase in (/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/module/jdk1.8.0_212/bin:/opt/module/hadoop-3.1.3/bin:/opt/module/hadoop-3.1.3/sbin:/opt/module/hive/bin:/home/jack/.local/bin:/home/jack/bin)
2020-04-24 17:00:19: Starting HiveServer2  
注意: 启动后窗口不能再操作，需打开一个新的shell窗口做别的操作
```

**2）编写hive服务启动脚本 myhive**

（1） 前台启动的方式导致需要打开多个shell窗口，可以使用如下方式后台方式启动

​		   nohup: 放在命令开头，表示不挂起,也就是关闭终端进程也继续保持运行状态

​		   2>&1 : 表示将错误重定向到标准输出上

​		   &: 放在命令结尾,表示后台运行

​		    一般会组合使用: nohup  [xxx命令操作]> file  2>&1 &  ， 表示将xxx命令运行的

​			结果输出到file中，并保持命令启动的进程在后台运行。

​			如上命令不要求掌握。 

```shell
[jack@hadoop202 hive]$ nohup hive --service metastore 2>&1 &
[jack@hadoop202 hive]$ nohup hive --service hiveserver2 2>&1 &
```

（2） 为了方便使用，可以直接编写脚本来管理服务的启动和关闭

```shell
[jack@hadoop102 hive]$ cd /home/jack/bin/
[jack@hadoop102 bin]$ vim myhive
```

内容如下：此脚本的编写不要求掌握。直接拿来使用即可。

```shell
#!/bin/bash
HIVE_LOG_DIR=$HIVE_HOME/logs
if [ ! -d $HIVE_LOG_DIR ]
then
	mkdir -p $HIVE_LOG_DIR
fi
#检查进程是否运行正常，参数1为进程名，参数2为进程端口
function check_process()
{
    pid=$(ps -ef 2>/dev/null | grep -v grep | grep -i $1 | awk '{print $2}')
    ppid=$(netstat -nltp 2>/dev/null | grep $2 | awk '{print $7}' | cut -d '/' -f 1)
    echo $pid
    [[ "$pid" =~ "$ppid" ]] && [ "$ppid" ] && return 0 || return 1
}

function hive_start()
{
    metapid=$(check_process HiveMetastore 9083)
    cmd="nohup hive --service metastore >$HIVE_LOG_DIR/metastore.log 2>&1 &"
    cmd=$cmd" sleep 4; hdfs dfsadmin -safemode wait >/dev/null 2>&1"
    [ -z "$metapid" ] && eval $cmd || echo "Metastroe服务已启动"
    server2pid=$(check_process HiveServer2 10000)
    cmd="nohup hive --service hiveserver2 >$HIVE_LOG_DIR/hiveServer2.log 2>&1 &"
    [ -z "$server2pid" ] && eval $cmd || echo "HiveServer2服务已启动"
}

function hive_stop()
{
    metapid=$(check_process HiveMetastore 9083)
    [ "$metapid" ] && kill $metapid || echo "Metastore服务未启动"
    server2pid=$(check_process HiveServer2 10000)
    [ "$server2pid" ] && kill $server2pid || echo "HiveServer2服务未启动"
}

case $1 in
"start")
    hive_start
    ;;
"stop")
    hive_stop
    ;;
"restart")
    hive_stop
    sleep 2
    hive_start
    ;;
"status")
    check_process HiveMetastore 9083 >/dev/null && echo "Metastore服务运行正常" || echo "Metastore服务运行异常"
    check_process HiveServer2 10000 >/dev/null && echo "HiveServer2服务运行正常" || echo "HiveServer2服务运行异常"
    ;;
*)
    echo Invalid Args!
    echo 'Usage: '$(basename $0)' start|stop|restart|status'
    ;;
esac
```

**3）添加执行权限**

```shell
[jack@hadoop102 bin]$ chmod 777 myhive
```

**4）启动/停止/查看Hive后台服务**

```shell
[jack@hadoop102 hive]$ myhive start	#后台启动hive
[jack@hadoop102 hive]$ myhive stop	#停止Hive服务
[jack@hadoop102 hive]$ myhive status	#查看Hive服务
```

#### 6.3 HiveJDBC访问

**1）启动beeline客户端**

```shell
[jack@hadoop102 hive]$ bin/beeline -u jdbc:hive2://hadoop102:10000 -n jack
```

**2）看到如下界面**

```
Connecting to jdbc:hive2://hadoop102:10000
Connected to: Apache Hive (version 3.1.2)
Driver: Hive JDBC (version 3.1.2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 3.1.2 by Apache Hive
0: jdbc:hive2://hadoop102:10000>
```

#### 6.4 Hive访问

**1）启动hive客户端**

```shell
[jack@hadoop102 hive]$ bin/hive
```

**2）看到如下界面**

```
which: no hbase in (/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/module/jdk1.8.0_212/bin:/opt/module/hadoop-3.1.3/bin:/opt/module/hadoop-3.1.3/sbin:/opt/module/hive/bin:/home/jack/.local/bin:/home/jack/bin)
Hive Session ID = 36f90830-2d91-469d-8823-9ee62b6d0c26

Logging initialized using configuration in jar:file:/opt/module/hive/lib/hive-common-3.1.2.jar!/hive-log4j2.properties Async: true
Hive Session ID = 14f96e4e-7009-4926-bb62-035be9178b02
hive>
```

**3）打印 当前库 和 表头**

在hive-site.xml中加入如下两个配置: **（hive-site.xml配置中已配置，这里可以省略）**

```xml
<property>
    <name>hive.cli.print.header</name>
    <value>true</value>
    <description>Whether to print the names of the columns in query output.</description>
</property>
<property>
    <name>hive.cli.print.current.db</name>
    <value>true</value>
    <description>Whether to include the current database in the Hive prompt.</description>
</property>
```

### 7 Hive常用交互命令

```xml
[jack@hadoop102 hive]$ bin/hive -help
usage: hive
 -d,--define <key=value>          Variable subsitution to apply to hive
                                  commands. e.g. -d A=B or --define A=B
    --database <databasename>     Specify the database to use
 -e <quoted-query-string>         SQL from command line
 -f <filename>                    SQL from files
 -H,--help                        Print help information
    --hiveconf <property=value>   Use value for given property
    --hivevar <key=value>         Variable subsitution to apply to hive
                                  commands. e.g. --hivevar A=B
 -i <filename>                    Initialization SQL file
 -S,--silent                      Silent mode in interactive shell
 -v,--verbose                     Verbose mode (echo executed SQL to the console)
```

**1）“-e”不进入hive的交互窗口执行sql语句**

```shell
[jack@hadoop102 hive]$ bin/hive -e "select id from student;"
```

**2）“-f”执行脚本中sql语句**
（1）在/opt/module/hive/下创建datas目录并在datas目录下创建hivef.sql文件

```shell
[jack@hadoop102 datas]$ touch hivef.sql
```

（2）文件中写入正确的sql语句

```sql
select *from student;
```

**（3）执行文件中的sql语句**

```shell
[jack@hadoop102 hive]$ bin/hive -f /opt/module/hive/datas/hivef.sql
```

**（4）执行文件中的sql语句并将结果写入文件中**

```shell
[jack@hadoop102 hive]$ bin/hive -f /opt/module/hive/datas/hivef.sql  > /opt/module/datas/hive_result.txt
```

### 8 Hive其他命令操作

**1）退出hive窗口：**

```
hive(default)>exit;
hive(default)>quit;
```

在新版的hive中没区别了，在以前的版本是有的：
exit:先隐性提交数据，再退出；
quit:不提交数据，退出；
**2）在hive cli命令窗口中如何查看hdfs文件系统**

```
hive(default)>dfs -ls /;
```

**3）查看在hive中输入的所有历史命令**
（1）进入到当前用户的根目录/root或/home/jack
（2）查看. hivehistory文件

```
[jack@hadoop102 ~]$ cat .hivehistory
```

### 9 Hive常见属性配置

**1.9.1 Hive运行日志信息配置**
1）Hive的log默认存放在/tmp/jack/hive.log目录下（当前用户名下）
2）修改hive的log存放日志到/opt/module/hive/logs
（1）修改/opt/module/hive/conf/hive-log4j.properties.template文件名称为hive-log4j.properties

```shell
[jack@hadoop102 conf]$ pwd
/opt/module/hive/conf
[jack@hadoop102 conf]$ mv hive-exec-log4j2.properties.template hive-log4j2.properties
```

（2）在hive-log4j.properties文件中修改log存放位置

```xml
property.hive.log.dir = /opt/module/hive/logs
```

### 9.2 参数配置方式

**1）查看当前所有的配置信息**

```
hive>set;
```

**2）参数的配置三种方式**

（1）配置文件方式

​		  默认配置文件：**hive-default.xml** 

​		  用户自定义配置文件：**hive-site.xml**

​      注意：**用户自定义配置会覆盖默认配置。**另外，Hive也会读入Hadoop的配置，**因为Hive是作为Hadoop的客户端启动的**，Hive的配置会覆盖Hadoop的配置。配置文件的设定对本机启动的所有Hive进程都有效。

（2）命令行参数方式

启动Hive时，可以在命令行添加-hiveconf param=value来设定参数。

 例如：

```shell
[jack@hadoop103 hive]$ bin/hive -hiveconf mapred.reduce.tasks=10;
```

注意：仅对本次hive启动有效

查看参数设置：	

```shell
hive (default)> set mapred.reduce.tasks;
```

（3）参数声明方式

可以在HQL中使用SET关键字设定参数

例如：

```shell
hive (default)> set mapred.reduce.tasks=100;
```

**注意：仅对本次hive启动有效。**

查看参数设置：

```shell
hive (default)> set mapred.reduce.tasks;
```

​	上述三种设定方式的优先级依次递增。即配置文件<命令行参数<参数声明。注意某些系统级的参数，例如log4j相关的设定，必须用前两种方式设定，因为那些参数的读取在会话建立以前已经完成了。

**3）临时修改日志级别**

操作hive的时候，每次都会打印很多info日志，如果在配置文件修改防止会漏error错误，这里临时设置日志关闭

```
SET hive.server2.logging.operation.level = NONE;
```