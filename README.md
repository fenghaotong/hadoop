
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

**[配置core-site.xml](hadoop基础知识.ipynb#配置core-site.xml)**

**[配置hadoop-hdfs-site.xml](hadoop基础知识.ipynb#配置hadoop-hdfs-site.xml)**

**[配置hadoop-mapred-site.xml](hadoop基础知识.ipynb#配置hadoop-mapred-site.xml)**

**[配置hadoop-yarn-site.xml](hadoop基础知识.ipynb#配置hadoop-yarn-site.xml)**

**[配置slave](hadoop基础知识.ipynb#配置slave)**

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

### HDFS的HA

- [HDFS的HA基础讲解](HA.ipynb)

**规划**

|主机名称| IP地址 | 功能|
|:---:|:----|:-----|
|hadoop1|192.168.75.111|NameNode、DataNode、jouranlnode、quroumPeerMain、zkf|
|hadoop2|192.168.75.111|DataNode、jouranlnode、quroumPeerMain、zkf|
|hadoop3|192.168.75.111|DataNode、jouranlnode、quroumPeerMain|

> 在配置HA之前，要对之前的配置做一个备份

```
mv /usr/local/hadoop-2.8.4/ /usr/local/hadoop-2.8.4_bak
```

#### [HA的配置](http://hadoop.apache.org/docs/r2.8.4/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html)

- 解压下载的hadoop，到/usr/local/目录下

在/usr/local/hadoop-2.8.4/etc/hadoop/下
- 配置hadoop-env.sh
- 配置hadoop-core-site.xml
- 配置hadoop-hdfs-site.xml
- 配置slave
- 发送hadoop文件到所有机子上
- hadoop2也作为namenode免密登陆

**[配置hadoop-env.sh](HA.ipynb#配置hadoop-env.sh)**

**[配置hadoop-core-site.xml](HA.ipynb#配置hadoop-core-site.xml)**

**[配置hadoop-hdfs-site.xml](HA.ipynb#配置hadoop-hdfs-site.xml)**

**[配置slaves](HA.ipynb#配置slaves)**

**发送hadoop文件到所有机子上**

```
scp -r ../hadoop-2.8.4/ hadoop2:/usr/local/
scp -r ../hadoop-2.8.4/ hadoop3:/usr/local/
```

**hadoop2也作为namenode免密登陆**

- 在hadoop2主机上

```
ssh-keygen -t rsa
ssh-copy-id hadoop2
ssh-copy-id hadoop1
ssh-copy-id hadoop3
```

**按步骤依次执行下面程序**

- 启动zk
- 启动journalnode服务（单个启动、多个进程启动）
```
hadoop-daemon.sh start journalnode
hadoop-daemons.sh start journalnode
```

- 挑选两个namenode中的一台进行格式化，然后并且启动
```
hdfs namenode -format
hadoop-daemon.sh start namenode
```
- 在另一台namenode的机器上拉去元数据（也可以复制）
```
hdfs namenode -bootstrapStandby
```
- 格式化zk
```
hdfs zkfc -formatZK
```
- 启动

```
start-dfs.sh
```
> 会有五个进程，查看是否全部启动   
> 查看webUI是否正(http://192.168.75.111:50070,http://192.168.75.112:50070)  
> 在hdfs中的读写文件  
> 然后关闭一个namenode失败，查看是否自动切换   

```
kill -9 [namenode pid]
hadoop-daemon.sh start namenode
```

### Yarn的HA

**规划**

|主机名称| IP地址 | 功能|
|:---:|:----|:-----|
|hadoop1|192.168.75.111|resourcemanager、nodemanager、quroumPeerMain|
|hadoop2|192.168.75.111|nodemanager、quroumPeerMain|
|hadoop3|192.168.75.111|nodemanager、quroumPeerMain|

#### HA的配置

- [HA基础](HA.ipybnb)
- 配置yarn-site.xml
- 配置mapred-site.xml
- 远程发送
- 直接启动
- 测试

**[配置yarn-site.xml](HA.ipynb#配置yarn-site.xml)**

**[配置mapred-site.xml](HA.ipynb#配置mapred-site.xml)**

**远程发送**

```
scp -r ./etc/hadoop/mapred-site.xml ./etc/hadoop/yarn-site.xml hadoop2:/usr/local/hadoop-2.8.4/etc/hadoop/

scp -r ./etc/hadoop/mapred-site.xml ./etc/hadoop/yarn-site.xml hadoop3:/usr/local/hadoop-2.8.4/etc/hadoop/
```

**直接启动**

```
#hadoop1
start-yarn.sh
#hadoop2
yarn-daemon.sh start resourcemanager
```

**测试**

```
yarn jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.4.jar wordcount /words /out/00
```


```python

```
