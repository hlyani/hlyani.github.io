# Hadoop 集群搭建

##### 1、解压相关软件包

```
tar -zxvf jdk1.8.0_111.tar.gz
tar -zxvf hadoop-2.7.3.tar.gz
tar -zxvf zookeeper-3.4.9.tar.gz
tar -zxvf hbase-1.2.3.tar.gz
```

##### 2、增加环境变量

```
vim /etc/profile
export JAVA_HOME=/opt/jdk1.8.0_111
export HADOOP_HOME=/opt/hadoop-2.7.3
export ZOOKEEPER_HOME=/opt/zookeeper-3.4.9
export HBASE_HOME=/opt/hbase-1.2.3
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin:$HBASE_HOME/bin
```

##### 3、编辑hosts

```
vim /etc/hosts
192.169.1.1 hadoop1
```

##### 4、配置hadoop

```
cd hadoop-2.7.3/etc/hadoop
```

##### 5、编辑hadoop-env.sh

```
vim hadoop-env.sh
export JAVA_HOME=/opt/jdk1.8.0_111
```

##### 6、将 slave 的主机名写入到该文件

```
vim slaves
hadoop2
hadoop3
```

##### 7、编辑core-site.xml

```
vim core-site.xml

<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop1:9000</value>
  </property>
  <property>		
    <name>hadoop.tmp.dir</name>
    <value>file:/usr/local/hadoop/tmp</value>
    <description>Abase for other temporary directories.</description>
  </property>
</configuration>
```

##### 8、编辑hdfs-site.xml

```
vim hdfs-site.xml
<configuration>
  <property>
    <name>dfs.namenode.http-address</name>
      <value>hadoop1:50070</value>
  </property>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
      <value>hadoop1:50090</value>
  </property>
  <property>
    <name>dfs.replication</name>
      <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
      <value>file:/usr/local/hadoop/tmp/dfs/name</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
      <value>file:/usr/local/hadoop/tmp/dfs/data</value>
  </property>
</configuration>
```

##### 9、编辑mapred-site.xml

```
vim mapred-site.xml

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

##### 10、编辑yarn-site.xml

```
vim yarn-site.xml

<configuration>
  <property>
    <name>yarn.resoursemanager.hostname</name>
    <value>hadoop1</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
```

##### 11、拷贝hadoop所有配置到所有其他节点

##### 12、启动hadoop

> 首次启动需要先在 master 节点上执行 namenode 的格式化操作

```
hdfs namenode -format
```

> 成功的话，会看到 “successfully formatted” 和 “Exitting with status 0” 的提示，若为 “Exitting with status 1” 则是出错。

> 完成 Hadoop 格式化后，在 namenode 节点上启动 Hadoop 各个服务

```
start-dfs.sh
--------------
58993 NameNode
59601 Jps
59459 SecondaryNameNode
59304 DataNode
```

```
start-yarn.sh
--------------
58993 NameNode
59649 ResourceManager
59459 SecondaryNameNode
60070 Jps
59767 NodeManager
59304 DataNode
```

```
mr-jobhistory-daemon.sh start historyserver
--------------------
58993 NameNode
59649 ResourceManager
60147 Jps
59459 SecondaryNameNode
59767 NodeManager
59304 DataNode
60108 JobHistoryServer
```

> 通过命令 jps 可以查看各个节点所启动的进程。正确的话，在 Master 节点上可以看到
> NameNode,ResourceManager,SecondaryNameNode,JobHistoryServer 进程。
> 在 Slave 节点可以看到 DataNode 和 NodeManager 进程。

> 另外还需要在 Master 节点上通过命令 hdfs dfsadmin -report
> 查看 DataNode 是否正常启动，如果 Live datanode 不为 0，则说明集群启动成功，

* NameNode结点                http://hadoop1:50070
* SecondaryNameNode    http://hadoop1:50090

##### 13、系统启动正常后，跑个程序试试

```
$mkdir input
$cd input
$echo "hello world">test1.txt
$echo "hello hadoop">test2.txt
$cd ..
$bin/hadoop dfs -put input in
$bin/hadoop jar build/hadoop-0.20.2-examples.jar wordcount in out
$bin/hadoop dfs -cat out/*
```

##### 14、Hadoop配置文件说明

> Hadoop 的运行方式是由配置文件决定的（运行 Hadoop 时会读取配置文件），因此如果需要从伪分布式模式切换回非分布式模式，需要删除 core-site.xml 中的配置项。

> 此外，伪分布式虽然只需要配置 fs.defaultFS 和 dfs.replication 就可以运行（官方教程如此），不过若没有配置 hadoop.tmp.dir 参数，则默认使用的临时目录为 /tmp/hadoo-hadoop，而这个目录在重启时有可能被系统清理掉，导致必须重新执行 format 才行。所以我们进行了设置，同时也指定 dfs.namenode.name.dir 和 dfs.datanode.data.dir，否则在接下来的步骤中可能会出错。

##### 15、配置zookeeper

```
cd zookeeper-3.4.9/conf
mv zoo_sample.cfg zoo.cfg
dataDir=/opt/zookeeper-3.4.9/data

server.0=hadoop1:2888:3888
server.1=hadoop2:2888:3888
server.2=hadoop3:2888:3888
```

> 接下来分别在 hadoop1、hadoop2、hadoop3 节点上的/opt/zookeeper-3.4.9/data 目录下创建一
> 个名为 myid 的文件，并在hadoop1节点上的 myid 文件里输入 0，在hadoop2节点上的myid输入
> 1，在hadoop3节点上的 myid 文件里输入 2

```
mkdir /opt/zookeeper-3.4.9/data
echo 0 >> /opt/zookeeper-3.4.9/data/myid
```

> 拷贝zookeeper配置到所有其他节点

> 每个节点启动zkserver

```
zkServer.sh start
---------------------
5089 QuorumPeerMain
5107 Jps
4468 SecondaryNameNode
4932 JobHistoryServer
4634 ResourceManager
4254 NameNode
```

> 验证 ZooKeeper 服务，三台节点必须是 1 个 leader 2 个 follower 的状态才算配置正确

```
zkServer.sh status 
```

##### 16、配置hbase

```
cd hbase-1.2.3/conf
```

```
vim hbase-env.sh
export JAVA_HOME=/opt/jdk1.8.0_111
export HBASE_MANAGES_ZK=false
```

```
vim hbase-site.xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hadoop1:9000/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>hadoop1,hadoop2,hadoop3</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/zookeeper-3.4.9/data/</value>
  </property>
</configuration>
```

```
vim regionservers
hadoop1
hadoop2
hadoop3
```

> 启动hbase

> 启动 HBase 时需要确保 hdfs 已经启动，使用命令 hdfs dfsadmin -report 查看以下 HDFS
> 集群是否正常，如果正常，在 master 节点上执行以下命令启动 HBase 集群:

```
start-hbase.sh 
-----------------------------
5089 QuorumPeerMain
4468 SecondaryNameNode
4932 JobHistoryServer
5476 HRegionServer
5319 HMaster
5545 Jps
4634 ResourceManager
4254 NameNode
```

```
http://hadoop1:16010
```

