---
layout: post
title: Spark入门：Hadoop之环境搭建（二）
category: hadoop
tags: [hadoop]
excerpt: Hadoop运行环境搭建教程（二）
lock: need
---

## 1 克隆集群机器

### 1.1 克隆步骤

![image-20200502185307942](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502185309.png)

![image-20200502185330520](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502185332.png)

![image-20200502185338862](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502185348.png)

![image-20200502185709388](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502185710.png)

![image-20200502185438558](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502185442.png)

![image-20200502185819396](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502185821.png)

### 1.2 重复克隆步骤1，克隆出三台虚拟机来组建集群

![image-20200502190206128](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502190207.png)

### 1.3 分别登陆51、52、53服务器，修改ip地址和主机名称

#### 1.3.1 修改hadoop51服务器

```shell
[jack@hadoop51 ~]$ vim /etc/hostname 
hadoop51				

[jack@hadoop51 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-ens33 
IPADDR=192.168.11.51

[jack@hadoop51 ~]$ reboot
```

#### 1.3.2 修改hadoop52服务器

```shell
[jack@hadoop52 ~]$ vim /etc/hostname 
hadoop52			

[jack@hadoop52 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-ens33 
IPADDR=192.168.11.51

[jack@hadoop52 ~]$ reboot
```

#### 1.3.3 修改hadoop53服务器

```shell
[jack@hadoop53 ~]$ vim /etc/hostname 
hadoop53

[jack@hadoop53 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-ens33 
IPADDR=192.168.11.53

[jack@hadoop53 ~]$ reboot
```

## 2 集群配置(完全分布式)

### 2.1 集群部署规划

> 注意：NameNode和SecondaryNameNode不要安装在同一台服务器
> ​           ResourceManager也很消耗内存，不要和NameNode、SecondaryNameNode配置在同一台机器上。

|      | hadoop102        | hadoop103                  | hadoop104                 |
| ---- | ---------------- | -------------------------- | ------------------------- |
| HDFS | NameNodeDataNode | DataNode                   | SecondaryNameNodeDataNode |
| YARN | NodeManager      | ResourceManagerNodeManager | NodeManager               |

### 2.2 配置集群

#### 1）配置hadoop-env.sh

```shell
[jack@hadoop50 module]$ cd /opt/module/hadoop-3.1.3/etc/hadoop/
[jack@hadoop50 hadoop]$ echo $JAVA_HOME			#查询JAVA_HOME地址，下面配置到hadoop-env.sh中
/opt/module/jdk1.8.0_212
[jack@hadoop50 hadoop]$ vim hadoop-env.sh 

# export JAVA_HOME=								#找到这一行，然后在下面添加JAVA_HOME目录地址
export JAVA_HOME=/opt/module/jdk1.8.0_212
```

#### 2）核心配置文件core-site.xml

```shell
[jack@hadoop50 hadoop]$ pwd
/opt/module/hadoop-3.1.3/etc/hadoop
[jack@hadoop50 hadoop]$ vim /core-site.xml 
```

configuration节点中 添加如下内容：

```xml
<!-- 指定NameNode的地址 -->
<property>
	<name>fs.defaultFS</name>
    <value>hdfs://hadoop51:9820</value>
</property>
<!-- 指定hadoop数据的存储目录  
      官方配置文件中的配置项是hadoop.tmp.dir ,用来指定hadoop数据的存储目录,此次配置用的hadoop.data.dir是自己定义的变量， 因为在hdfs-site.xml中会使用此配置的值来具体指定namenode 和 datanode存储数据的目录
-->
<property>
	<name>hadoop.data.dir</name>
    <value>/opt/module/hadoop-3.1.3/data</value>
</property>

<!-- 下面是兼容性配置，先跳过 -->
<!-- 配置该jack(superUser)允许通过代理访问的主机节点 -->
<property>
	<name>hadoop.proxyuser.jack.hosts</name>
    <value>*</value>
</property>
<!-- 配置该jack(superuser)允许代理的用户所属组 -->
<property>
	<name>hadoop.proxyuser.jack.groups</name>
    <value>*</value>
</property>
<!-- 配置该jack(superuser)允许代理的用户-->
<property>
	<name>hadoop.proxyuser.jack.users</name>
	<value>*</value>
</property>
```

#### 3）HDFS配置文件 hdfs-site.xml

```shell
[jack@hadoop50 hadoop]$ vim hdfs-site.xml
```

添加如下内容：

```xml
<configuration>
<!-- 指定NameNode数据的存储目录 -->
<property>
	<name>dfs.namenode.name.dir</name>
    <value>file://${hadoop.data.dir}/name</value>
</property>

<!-- 指定Datanode数据的存储目录 -->
<property>
    <name>dfs.datanode.data.dir</name>
    <value>file://${hadoop.data.dir}/data</value>
</property>
    
<!-- 指定SecondaryNameNode数据的存储目录 -->
<property>
    <name>dfs.namenode.checkpoint.dir</name>
    <value>file://${hadoop.data.dir}/namesecondary</value>
</property>
   
<!-- 兼容配置，先跳过 -->
<property>
    <name>dfs.client.datanode-restart.timeout</name>
    <value>30s</value>
</property>

<!-- nn web端访问地址-->
<property>
  	<name>dfs.namenode.http-address</name>
  	<value>hadoop51:9870</value>
</property>
<!-- 2nn web端访问地址-->
<property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop53:9868</value>
</property>
</configuration>
```

#### 4）YARN配置文件 yarn-site.xml

```shell
[jack@hadoop50 hadoop]$ vim yarn-site.xml
```

添加如下内容：

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
<property>
	<name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<!-- 指定ResourceManager的地址-->
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop52</value>
</property>
<!-- 环境变量的继承 -->
<property>
    <name>yarn.nodemanager.env-whitelist</name>        
    <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
</property>
</configuration>
[jack@hadoop51 hadoop]$ 

```

#### 5）MapReduce配置文件 mapred-site.xml

```shell
[jack@hadoop50 hadoop]$ vim mapred-site.xml
```

添加如下内容：

```xml
<!—指定MapReduce程序运行在Yarn上 -->
<property>
	<name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```

### 2.3 在集群上分发配置好的hadoop

> xsync：自定义同步脚本，在本文后面有代码。

```shell
xsync /opt/module/hadoop-3.1.3

==================== hadoop51 ====================
The authenticity of host 'hadoop51 (192.168.11.51)' can't be established.
ECDSA key fingerprint is SHA256:wO9s2euKcQf497zb+uGAPOCCDro2I6ShkE1p2IwGhkA.
ECDSA key fingerprint is MD5:be:fd:0a:ea:24:c1:51:06:23:a8:aa:9a:af:69:c7:41.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'hadoop53,192.168.11.51' (ECDSA) to the list of known hosts.
jack@hadoop51's password: 
jack@hadoop51's password: 
==================== hadoop52 ====================
The authenticity of host 'hadoop51 (192.168.11.52)' can't be established.
ECDSA key fingerprint is SHA256:wO9s2euKcQf497zb+uGAPOCCDro2I6ShkE1p2IwGhkA.
ECDSA key fingerprint is MD5:be:fd:0a:ea:24:c1:51:06:23:a8:aa:9a:af:69:c7:41.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'hadoop53,192.168.11.52' (ECDSA) to the list of known hosts.
jack@hadoop52's password: 
jack@hadoop52's password: 
==================== hadoop53 ====================
The authenticity of host 'hadoop53 (192.168.11.53)' can't be established.
ECDSA key fingerprint is SHA256:wO9s2euKcQf497zb+uGAPOCCDro2I6ShkE1p2IwGhkA.
ECDSA key fingerprint is MD5:be:fd:0a:ea:24:c1:51:06:23:a8:aa:9a:af:69:c7:41.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'hadoop53,192.168.11.53' (ECDSA) to the list of known hosts.
jack@hadoop53's password: 
jack@hadoop53's password: 
```

分发后登陆hadoop52、hadoop53服务器查看刚刚修改的配置文件是否同步成功。

![image-20200502193544488](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502193545.png)

## 3 集群单点启动

1）如果集群是第一次启动，需要**格式化NameNode**，在hadoop51执行格式化

```shell
hdfs namenode -format
```

2）在hadoop51上启动NameNode、DataNode、NodeManager

```shell
hdfs --daemon start namenode
hdfs --daemon start datanode
yarn --daemon start nodemanager
```

3）在hadoop52上启动yarn、DataNode、NodeManager

```shell
hdfs --daemon start datanode
yarn --daemon start resourcemanager
yarn --daemon start nodemanager
```

4）在hadoop53上启动2nn、DataNode、NodeManager

```shell
hdfs --daemon start secondarynamenode
hdfs --daemon start datanode
yarn --daemon start nodemanager
```

完成后分别在三台服务器执行jps命令，看到如下结果（进程号可能不同）：

```shell
[jack@hadoop51 bin]$ jps
1456 DataNode
1347 NameNode
1815 Jps
1534 NodeManager

[jack@hadoop52 hadoop]$ jps
1248 DataNode
1426 NodeManager
1322 ResourceManager
1821 Jps

[jack@hadoop53 hadoop-3.1.3]$ jps
1331 DataNode
1411 NodeManager
1588 Jps
1268 SecondaryNameNode
```

4）在web端查看服务是否正确运行

http://hadoop51:9870/dfshealth.html#tab-datanode

![image-20200502195903519](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502195905.png)

http://hadoop52:8088/cluster/nodes

![image-20200502201745058](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502201746.png)



## 4 自定义脚本

> ​	切换到用户家目录，然后创建bin目录，进入到bin目录中创建自定义脚本，因为用户bin目录在环境变量中已经声明，在该目录创建的脚本可以直接调用。

```shell
[jack@hadoop51 hadoop]$ cd ~
[jack@hadoop51 ~]$ ll -a
总用量 32
drwx------. 2 jack jack 4096 5月   2 19:19 .
drwxr-xr-x. 3 root root 4096 5月   2 17:57 ..
-rw-------. 1 jack jack  574 5月   2 18:33 .bash_history
-rw-r--r--. 1 jack jack   18 4月  11 2018 .bash_logout
-rw-r--r--. 1 jack jack  193 4月  11 2018 .bash_profile
-rw-r--r--. 1 jack jack  231 4月  11 2018 .bashrc
-rw-------. 1 jack jack 7013 5月   2 19:19 .viminfo
[jack@hadoop51 ~]$ mkdir bin
[jack@hadoop51 ~]$ cd bin/
```

### 4.1 xsync脚本

> 同步文件到集群其他集群。
>
> 参数：文件或文件目录

```shell
[jack@hadoop51 bin]$ vim xsync

#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
  echo Not Enough Arguement!
  exit;
fi
#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104			# 主机名改成自己的
do
  echo ====================  $host  ====================
  #3. 遍历所有目录，挨个发送
  for file in $@
  do
    #4 判断文件是否存在
    if [ -e $file ]
    then
      #5. 获取父目录
      pdir=$(cd -P $(dirname $file); pwd)
      #6. 获取当前文件的名称
      fname=$(basename $file)
      ssh $host "mkdir -p $pdir"
      rsync -av $pdir/$fname $host:$pdir
    else
      echo $file does not exists!
    fi
  done
done
```

```shell
chmod 777 xsync										# 给脚本添加执行权限
```

