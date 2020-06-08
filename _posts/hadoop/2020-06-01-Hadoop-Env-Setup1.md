---
layout: post
title: Spark入门：Hadoop之环境搭建（一）
category: hadoop
tags: [hadoop]
excerpt: Hadoop运行环境搭建教程（一）
lock: need
---

## 1 软件版本说明

> VMware-workstation-full-15.5.1-15018445.exe
>
> CentOS-7.5-x86_64-DVD-1804.iso
>
> jdk-8u212-linux-x64.tar.gz
>
> hadoop-3.1.3.tar.gz
> 

## 2、模板机安装

> ​	模板机是用于复制的基础环境，节省了新增节点需要重新安装服务器的繁琐操作。

### 2.1 模板机安装要求

1. 采用最小化安装方式
2. 修改网络YUM源
3. 安装基础指令
4. 设置固定ip地址
5. 修改主机名称
6. 配置主机名称映射
7. 关闭防火墙
8. 创建一个专用用户账号，配置用户具有root权限
9. 在/opt目录下创建文件夹，在/opt目录下创建module、software文件夹

虚拟机安装过程可参考该[博客](https://blog.csdn.net/lcode_ch/article/details/105889772)，下面教程是在虚拟机安装之后开始讲述。

### 2.2 修改网络YUM源

> 默认的系统YUM源，需要连接国外apache网站，网速比较慢，可以修改关联的网络YUM源为国内镜像的网站，比如网易163,aliyun等。

1)  安装wget, wget用来从指定的URL下载文件

```shell
[root@localhost ~]# yum install -y wget
```

2)  在/etc/yum.repos.d/目录下，备份默认的repos文件,

```shell
[root@localhost ~]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# pwd
/etc/yum.repos.d
[root@localhost yum.repos.d]# cp CentOS-Base.repo CentOS-Base.repo.backup
```

3)  下载aliyun的repos文件

```shell
[root@localhost yum.repos.d]# wget http://mirrors.aliyun.com/repo/Centos-7.repo
```

4)  使用下载好的repos文件替换默认的repos文件

```shell
[root@localhost yum.repos.d]# mv -f Centos-7.repo CentOS-Base.repo
```

5)  清理旧缓存数据，缓存新数据 

```shell
[root@localhost yum.repos.d]# yum clean all
[root@localhost yum.repos.d]# yum makecache
```

yum makecache就是把服务器的包信息下载到本地电脑缓存起来。

### 2.3 安装Linux基础指令

> 因为本次系统环境采用的最小化安装，所以会有很多常用命令没有默认集成，需要我们自己安装，所以第一步我们先安装一下常用的基础指令。

```shell
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33
-bash: vim: 未找到命令
[root@localhost ~]# sudo yum install -y epel-release keepalived libaio
[root@localhost ~]# sudo yum install -y psmisc nc net-tools rsync vim lrzsz ntp libzstd openssl-static tree iotop
```

### 2.3 修改静态IP

2.3.1 查看IP配置文件

```shell
sudo vim /etc/sysconfig/network-scripts/ifcfg-ens33
```

![image-20200502171914494](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502174356.png)

![image-20200502174454006](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502174455.png)

```shell
BOOTPROTO=static

IPADDR=192.168.11.50
GATEWAY=192.168.11.2
DNS1=192.168.11.2
```

重启网络服务之后看能不能ping通百度和本机电脑

```shell
[root@localhost yum.repos.d]# systemctl restart network
```

![image-20200502175054602](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502175055.png)

![image-20200502175249440](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502175251.png)

### 2.4 修改主机名称

```shell
[root@localhost ~]# cat /etc/hostname 
localhost.localdomain
[root@localhost ~]# vim /etc/hostname 
[root@localhost ~]# cat /etc/hostname 
hadoop50
[root@localhost ~]# reboot					# 修改之后执行reboot重启虚拟机
```

### 2.5 配置主机名称映射

> 稍后一共会创建四台虚拟机，名称如下：
>
> hadoop50：模板机
>
> 根据模板机完整克隆三台机器用作集群机器来用，hadoop51、hadoop52、hadoop53

```shell
sudo vim /etc/hosts
# 添加如下内容
192.168.11.50 hadoop50
192.168.11.51 hadoop51
192.168.11.52 hadoop52
192.168.11.53 hadoop53
```

### 2.6 修改window主机映射文件(host)文件

> Host Path：C:\Windows\System32\drivers\etc\host
>
> 设置映射之后，在window环境就可以通过主机名称访问虚拟机而不需要通过IP地址了。

```shell
192.168.11.50 hadoop50
192.168.11.51 hadoop51
192.168.11.52 hadoop52
192.168.11.53 hadoop52
```

测试：

![image-20200502175640963](https://lcode-cloudimg.oss-cn-shenzhen.aliyuncs.com/picGO/20200502175642.png)

### 2.7 关闭防火墙

```shell
sudo systemctl stop    firewalld	# 关闭防火墙
sudo systemctl disable firewalld	# 禁止开机启动
```

### 2.8 创建专用用户

> 创建一个专用的用户，防止使用root权限过大，误操作对系统造成不可弥补的伤害。

```shell
[root@hadoop50 ~]# sudo useradd jack
[root@hadoop50 ~]# sudo passwd  jack
更改用户 jack 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
[root@localhost ~]# 
```

### 2.9 配置用户具有root权限

```shell
# 修改/etc/sudoers文件，找到下面一行（91行），在root下面添加一行，如下所示：
sudo vi /etc/sudoers

## Allow root to run any commands anywhere
root    ALL=(ALL)           ALL
jack    ALL=(ALL)  NOPASSWD:ALL
```

### 2.10 在/opt目录下创建文件夹

> 在/opt目录下创建module、software文件夹。
>
> softwore：安装包目录
>
> module：软件安装目录

```shell
[root@hadoop50 ~]# cd /opt/
[root@hadoop50 opt]# ll
总用量 0
[root@hadoop50 opt]# mkdir module
[root@hadoop50 opt]# mkdir softwore
[root@hadoop50 opt]# ll
总用量 8
drwxr-xr-x. 2 root root 4096 5月   2 18:00 module
drwxr-xr-x. 2 root root 4096 5月   2 18:00 softwore
[root@hadoop50 opt]# chown jack:jack module/ softwore/		#修改目录所属用户和组
[root@hadoop50 opt]# ll
总用量 8
drwxr-xr-x. 2 jack jack 4096 5月   2 18:00 module
drwxr-xr-x. 2 jack jack 4096 5月   2 18:00 softwore
[root@hadoop50 opt]# 
```

## 3 安装JDK

1. 将JDK安装包上传到Linux /opt/software目录下**（使用jack用户登录，不要需要修改文件权限）**

2. 解压JDK到/opt/module目录下

   ```shell
   [jack@hadoop50 ~]$ cd /opt/softwore/
   [jack@hadoop50 softwore]$ ll
   总用量 190444
   -rw-rw-r--. 1 jack jack 195013152 5月   2 18:08 jdk-8u212-linux-x64.tar.gz
   [jack@hadoop50 softwore]$ tar -zxvf jdk-8u212-linux-x64.tar.gz -C /opt/module/
   ```

3. 配置JDK环境变量,两种方式

   3.1. 第一种（我使用第一种方式）

      新建/etc/profile.d/my_env.sh文件

      ```shell
   sudo vim /etc/profile.d/my_env.sh
      
   #JAVA_HOME
   export JAVA_HOME=/opt/module/jdk1.8.0_212
   export PATH=$PATH:$JAVA_HOME/bin
      ```

   3.2. 第二种

      直接将环境变量配置到 /etc/profile 文件中,在/etc/profile文件的末尾追加如下内容

      ```shell
   JAVA_HOME=/opt/module/jdk1.8.0_212
   PATH=$PATH:$JAVA_HOME/bin
   export PATH JAVA_HOME
      ```

4. 刷新环境变量

   ```shell
   source /etc/profile
   ```

5. 测试JDK是否安装成功

   ```shell
   java -version
   ```

## 4 安装Hadoop

> Hadoop下载地址：[https://archive.apache.org/dist/hadoop/common/hadoop-3.1.3/](https://archive.apache.org/dist/hadoop/common/hadoop-2.7.2/)

1. 将hadoop安装包上传到/opt/software目录下

2. 解压安装文件到/opt/module下面

   ```shell
   [jack@hadoop50 ~]$ cd /opt/softwore/
   [jack@hadoop50 softwore]$ ll
   总用量 520604
   -rw-rw-r--. 1 jack jack 338075860 5月   2 18:08 hadoop-3.1.3.tar.gz
   
   [jack@hadoop50 softwore]$ tar -zxvf hadoop-3.1.3.tar.gz -C /opt/module/
   ```

3. 查看是否解压成功

   ```shell
   ll /opt/module/hadoop-3.1.3
   ```

4. 将Hadoop添加到环境变量

   ```shell
   sudo vim /etc/profile.d/my_env.sh
   
   #JAVA_HOME
   JAVA_HOME=/opt/module/jdk1.8.0_212
   
   #HADOOP_HOME
   HADOOP_HOME=/opt/module/hadoop-3.1.3
   
   PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   export PATH JAVA_HOME HADOOP_HOME HIVE_HOME
   ```

5. 让修改后的文件生效

   ```shell
   [jack@hadoop50 softwore]$ source /etc/profile
   ```

6. 测试是否安装成功

   ```shell
   [jack@hadoop50 softwore]$ hadoop version
   Hadoop 3.1.3
   Source code repository https://gitbox.apache.org/repos/asf/hadoop.git -r ba631c436b806728f8ec2f54ab1e289526c90579
   Compiled by ztang on 2019-09-12T02:47Z
   Compiled with protoc 2.5.0
   From source with checksum ec785077c385118ac91aadde5ec9799
   This command was run using /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-common-3.1.3.jar
   [jack@hadoop50 softwore]$ 
   
   ```

7. 重启（如果Hadoop命令不能用再重启）

   ```shell
   reboot
   ```