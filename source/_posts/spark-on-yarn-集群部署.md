---
title: spark on yarn 集群部署
date: 2018-05-03 21:33:54
categories:
	- 技术
tags:
	- spark
	- 大数据
copyright: true
---

最近用到spark，把本地搭建spark on yarn 集群的步骤记录一下

<!--more-->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=28029141&auto=1&height=66"></iframe>

|IP|主机名|
|---|---|
|192.168.22.137|spark-master|
|192.168.22.150|spark-slave1|

### 更改主机名
确定每个节点的主机名与它在集群中所处的位置相同
如果不同，需要修改`vi /etc/hostname`
重启生效


### 可能需要些安装某些工具包

- 更换sources源
```bash
vi /etc/apt/sources.list
```
```bash
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
```

- apt install net-tools 

- apt-get install iputils-ping


### 修改各主机的hosts文件
`vi /etc/hosts`
添加以下内容
```
192.168.22.137 spark-master
192.168.22.150 spark-slave1
```

### SSH免密登录
我看了网上别人的说只需要安装server，但是我没有成功，我安装了server和client才行
```bash
apt-get install openssh-client
apt-get install openssh-server

# 启动ssh服务
etc/init.d/ssh start
```
*关于ssh服务可以参照这个链接
http://linux.it.net.cn/e/server/ssh/2015/0501/14838.html*

紧接着就是配置各主机的免密登录

- 所有的主机都需要生成私钥和公钥(直接回车)

`ssh-keygen -t rsa`

- 将所有主机的`~/.ssh/id_rsa.pub`都要放在master节点的`~/.ssh/`目录下(最好更改用以区分)

我使用的`lrzsz`工具(有点笨)。之后再主机执行

*你也可以使用`scp ~/.ssh/id_rsa.pub root@<hostname|ip>:~/.ssh/id_rsa.pub.slave1`*

- 将所有公钥加到用于认证的公钥文件authorized_keys中
```bash
cat ~/.ssh/id_rsa.pub* >> ~/.ssh/authorized_keys
```
此时`~/.ssh/`再将`~/.ssh/authorized_keys`拷贝其他节点，到此个主机就完成了免密登录

验证一下:
```bash
ssh spark-master
ssh spark-slave1
```
如果出现如下，就说明你成功了

[![1.png](https://i.loli.net/2018/04/28/5ae42fa58b086.png)](https://i.loli.net/2018/04/28/5ae42fa58b086.png)



### 所需环境配置

#### 准备软件

用的版本不是最新的，看个人需要，但要保证各软件的版本要互相支持

```bash
.
├── hadoop-2.6.5.tar.gz
├── jdk-8u171-linux-x64.tar.gz
├── scala-2.10.4.tgz
└── spark-1.6.3-bin-hadoop2.6.tgz
```
可直接去各大官网下载，如果你想省事，直接从网盘里下载也行

https://pan.baidu.com/s/1vSu-6OTMvkROCBsiJwbMWQ

#### 统一配置环境

我将所有的软件放在`/spark/software/`目录下、解压与修改文件名，然后统一配置环境变量

`vi /etv/profile`

假如如下内容

```
export JAVA_HOME=/spark/software/java
export JRE_HOME=$JAVA_HOME/jre
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=$CLASSPATH:.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib

export SCALA_HOME=/spark/software/scala
export PATH=$PATH:$SCALA_HOME/bin

export HADOOP_HOME=/spark/software/hadoop
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export YARN_HOME=/spark/software/hadoop
export YARN_CONF_DIR=${YARN_HOME}/etc/hadoop
```

然后执行`source /etc/profile` 另其生效
然后测试，出现如下说明成功了
[![2.png](https://i.loli.net/2018/04/28/5ae4341889464.png)](https://i.loli.net/2018/04/28/5ae4341889464.png)

当然你也可以吧hadoop和spark假如path中，这样就可以随时使用`hdfs`和`spark-submit`命令了

**其他主机做同样的操作**

提示：各主机最好都统一路径，这样修改一个文件，然后将文件直接远程拷贝到其他主机上就行了

### HADOOP配置
在`/spark/software/hadoop/etc/hadoop`目录下需要配置以下几个文件：
```
hadoop-env.sh，
yarn-env.sh，
slaves，
core-site.xml，
hdfs-site.xml，
maprd-site.xml，
yarn-site.xml
```

#### hadoop-env.sh
`export JAVA_HOME=/spark/software/java`

#### yarn-env.sh
`export JAVA_HOME=/spark/software/java`

#### slaves
```
spark-slave1
```
(这里我只添加了一个slave，你也可以把master加上去)

#### core-site.xml
添加如下：
```xml
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://spark-master:9000/</value>
</property>
<property>
    <name>hadoop.tmp.dir</name>
    <value>file:/spark/software/hadoop/tmp</value>
</property>
```
#### hdfs-site.xml
添加如下：
```xml
<property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>spark-master:9001</value>
</property>
<property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/spark/software/hadoop/dfs/name</value>
</property>
<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/spark/software/hadoop/dfs/data</value>
</property>
<property>
    <name>dfs.replication</name>
    <value>3</value>
</property>
```
#### maprd-site.xml
添加如下
```xml
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
<!-- 下面的视情况而配置你可以先只配置上面的即可 -->
<property>
    <name>mapreduce.map.memory.mb</name>
    <value>1536</value>
</property>
<property>
    <name>mapreduce.map.java.opts</name>
    <value>-Xmx1024M</value>
</property>
<property>
    <name>mapreduce.reduce.memory.mb</name>
    <value>1024</value>
</property>
<property>
    <name>mapreduce.reduce.java.opts</name>
    <value>-Xmx1024M</value>
</property>
```
#### yarn-site.xml
添加如下
```xml
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
<property>
    <name>yarn.resourcemanager.address</name>
    <value>spark-master:8032</value>
</property>
<property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>spark-master:8030</value>
</property>
<property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>spark-master:8035</value>
</property>
<property>
    <name>yarn.resourcemanager.admin.address</name>
    <value>spark-master:8033</value>
</property>
<property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>spark-master:8088</value>
</property>
<!-- 下面是情况而定 具体可以参考这里 http://blog.javachen.com/2015/06/05/yarn-memory-and-cpu-configuration.html-->
<property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>2000</value>
</property>
<property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>2000</value>
</property>
```

以上配置完毕之后，要同步到其他主机上,因为配置了免密，可以这样操作
```bash
scp /spark/software/hadoop/etc/hadoop/ root@spark-slave1:/spark/software/hadoop/etc/hadoop/
```

### HADOOP启动
进入`/spark/software/hadoop`目录下
- 格式化namenode
`bin/hfds namenode -format` 

当出现“successful”的字样，就说明成功了

- 启动dfs
`sbin/start-dfs.sh`

- 启动yarn
`sbin/start-yarn.sh`


接下来验证，spark-master 执行jps，有以下几个进程
```
27570 SecondaryNameNode
27720 ResourceManager
27356 NameNode
32476 Jps
```

每个slave上应该有以下几个进程
```
18324 DataNode
18489 NodeManager
21055 Jps
```

可以在任意一台主机上的浏览器输入
```
http://spark-master:8088/cluster/nodes     yarn管理界面
http://spark-master:50070                  hdfs页面
```


### spark 环境
在`/spark/software/spark/conf`目录下修改`spark-env.sh`(需先拷贝spark-env.sh.template)文件
```
export SCALA_HOME=spark/software/scala
export JAVA_HOME=/spark/software/java
export HADOOP_HOME=/spark/software/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
SPARK_MASTER_IP=spark-master
SPARK_LOCAL_DIRS=/spark/software/spark
SPARK_DRIVER_MEMORY=1G
```

同样在slaves文件中添加子节点
```
spark-slave1
```

同样将这两个文件发送到其他主机对应位置


然后在`/spark/software/spark`目录下执行
```bash

```

### 集群配置参考


- http://wuchong.me/blog/2015/04/04/spark-on-yarn-cluster-deploy/
https://www.jianshu.com/p/aa6f3a366727


### 可能用到的链接

- http://www.cnblogs.com/zlslch/p/6683814.html