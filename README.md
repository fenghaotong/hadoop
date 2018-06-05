
# hadoop配置

- [hadoop基础知识](hadoop基础知识.ipynb)

- java JDK(下载jdk解压配置环境变量)
- nc(网络安全工具)
- gcc
- bashdb（下载bashdb, 配置、编译、安装）
- apache
- nginx

### 节点规划

|主机名称| IP地址 | 功能|
|:---:|:----|:-----|
|hadoop1|192.168.75.111|NameNode、DataNode、resourcemanager、nodemanager|
|hadoop2|192.168.75.111|DataNode、nodemanager|
|hadoop3|192.168.75.111|DataNode、nodemanager|

### 创建虚拟机配置网络

- 手动配置DHCP
- 修改域名，在/etc/sysconfig/network中添加HOSTNAME=hadoop1
- 把域名添加到/etc/hosts中
```
192.168.75.111 hadoop1 www.hadoop1.com
```

- 修改/etc/sysconfig/network-scripts/ifcfg-ens33文件
    - 添加DNS和网关一样
    - ONBOOT=yes

#### 安装nginx


- [pcre](http://www.pcre.org/)

```
tar zxvf pcre-8.36.tar.gz
cd pcre-8.36
./configure && make && make install
```

- [openssl](http://www.openssl.org/ ) 
```
 tar zxvf openssl-1.1.1.tar.gz
cd openssl-fips-1.1.1
./config && make && make install
```
- [zlib](http://www.zlib.net/)
```
tar zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure && make && make install
```
- [nginx](http://nginx.org/download/)

```
tar zxvf nginx-1.14.0.tar.gz
cd nginx-1.14.0
 ./configure && make && make install
 ```

### 单机版hadoop配置

- 下载[hadoop](http://hadoop.apache.org/releases.html)
- 解压压缩包到`/usr/local/`目录
- 配置环境变量
- 修改hadoop/etc/hadoop/hadoop-env.sh文件中的JAVA环境变量为JDK路径
- 测试
```
    which hadoop
    hadoop version
```    

```shell
$ mkdir /home/htfeng/data/input
$ cp hadoop/etc/hadoop/*.xml /home/htfeng/data/input
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.1.jar grep /home/htfeng/data/input /home/htfeng/data/output 'dfs[a-z.]+'
$ cat /home/htfeng/data/output/*
$ hdfs dfs -ls /
```

### 克隆虚拟机

- 修改网卡信息（忽略）

```
vi /etc/udev/rules.d/70-persistent-ipoib.rules 
```

- 修改主机名

```
vi /etc/sysconfig/network
```

- 修改ip信息

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33

修改ip、硬件地址和UUID
```

- 修改映射

```
vi /etc/hosts  

192.168.75.112 hadoop2 www.hadoop2.com
```

### 集群配置

- 配置JDK
- 配置hadoop
- 配置hadoop-env.sh
- 配置hadoop-core-site.xml
- 配置hadoop-hdfs-site.xml
- 配置hadoop-mapred-site.xml
- 配置hadoop-yarn-site.xml
- 配置slave
- 发送hadoop文件到所有机子上

**配置core-site.xml**
```xml
<configuration>
<!--hdfs filesystem namespace-->
    <property>
        <name>fs.defaultFS</name>
        <value>hadoop1:9000</value>
    </property>
<!--hdfs cache-->
    <property>
        <name>io.file.buffer.size</name>
        <value>4096</value>
    </property>
<!--tmp data-->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/htfeng/tmp/</value>
    </property>
    
    <property>
        <name>fs.hdfs.impl</name>
        <value>org.apache.hadoop.hdfs.DistributedFileSystem</value>
        <description>The FileSystem for hdfs: uris.</description>
    </property>
</configuration>
```

**配置hadoop-hdfs-site.xml**

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.block.size</name>
        <value>134217728</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/htfeng/hadoopdata/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/home/htfeng/hadoopdata/dfs/data</value>
    </property>
    <property>
        <name>dfs.checkpoint.dir</name>
        <value>/home/htfeng/hadoopdata/checkpoint/dfs/name</value>
    </property>
    <property>
        <name>dfs.http.address</name>
        <value>hadoop1:50070</value>
    </property>
    <property>
        <name>dfs.secondary.http.address</name>
        <value>hadoop1:50090</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>false</value>
    </property>
    <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>
</configuration>

```

**配置hadoop-mapred-site.xml**

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>hadoop1:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop1:19888</value>
    </property>
</configuration>
```

**配置hadoop-yarn-site.xml**

```xml
<configuration>

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop1</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>hadoop1:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>hadoop1:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>hadoop1:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>hadoop1:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>hadoop1:8088</value>
    </property>
</configuration>

```

**配置slave**

```
hadoop1
hadoop2
hadoop3

```

** 发送hadoop文件到所有机子上 **

```
scp -r ../hadoop-2.8.4/ hadoop2:/usr/local/
scp -r ../hadoop-2.8.4/ hadoop3:/usr/local/
```

**启动之前在namenode服务器上格式化，只需一次即可**

- 关闭防火墙

```
systemctl stop firewalld.service
```

```
hadoop namenode -format

全启动：start-all.sh
模块启动：
start-dfs.sh
start-yarn.sh
单个进程启动
hadoop-daemon.sh start/stop namenode
hadoop-daemons.sh start/stop datanode
yarn-daemon.sh start/stop namenode
yarn-daemons.sh start/stop datanode
mr-jobhistory-daemon.sh start/stop historyserver
```

> netstat -tnpl:查看启动端口

- 启动start-dfs.sh测试
```
http://192.168.75.111:50070
```
- 上传和下载文件（测试hdfs）
```
hdfs dfs -put ./README.txt /
```
- 启动start-yarn.sh跑一个mapreduce作业
```
yarn jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.4.jar wordcount /README.txt /out/00
```

### 免密登录

```
ssh-keygen -t rsa
ssh-copy-id hadoop1
ssh-copy-id hadoop2
ssh-copy-id hadoop3
```

## HDFS shell

### hdfs的shell

命令和shell命令基本一致

```shell
hdfs dfs -

Usage: hadoop fs [generic options]
	[-appendToFile <localsrc> ... <dst>]
	[-cat [-ignoreCrc] <src> ...]
	[-checksum <src> ...]
	[-chgrp [-R] GROUP PATH...]
	[-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
...
```

```
hdfs dfs -ls /   
hadoop fs - /  
hdfs dfs -ls -R / #递归查看  
hdfs dfs -mkdir -p /  #递归创建
hdfs dfs -touchz  /test/te.txt
hdfs dfs -put ./if.sh /if.sh  # 本地目录上传到hdfs文件目录
hdfs dfs -get /if.sh /home/htfeng  #从hdfs文件目录下载
hdfs dfs -du -s / #统计目录大小
```

### window中安装maven

- 下载[maven](http://mirrors.shu.edu.cn/apache/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.zip)
- 解压并复制到一个目录
- 配置环境变量
- 配置maven，修改settings.xml文件，指定本地仓库位置`H:\hadoop\mvnrepositry`， [maven](http://mvnrepository.com/)库
- 和java的编辑工具整合（eclipse,idea,myeclipse等）

[maven项目测试](maven测试.ipynb)

### hadoop核心协议RPC（远程过程调用协议）

例子

- 建一个rpc的包，然后创建下面三个java脚本,[具体代码](RPC.ipynb)
    - hello.java
    - RPCServer.java
    - RPCClient.java

### zookeeper

- [zookeeper基础知识](zookeeper.ipynb#zookeeper)
- [Download](http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz)

- 解压到/usr/local/
- 配置环境变量
- 修改配置文件/zookeeper/conf/zoo.cfg

```
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/home/htfeng/zkdata/
clientPort=2181
server.1=hadoop1:2888:3888
server.2=hadoop2:2888:3888
server.3=hadoop3:2888:3888
```

**在每个服务器上面创建myid,内容为server.X中的X**
```
mkdir /home/htfeng/zkdata
vi /home/htfeng/zkdata/myid
```

**[zookeeper测试](zookeeper.ipynb#zookeeper测试)**
