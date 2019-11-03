# [大数据相关环境安装]()

## 一、基础准备

### 1、关闭防火墙等（所有虚拟机都执行）

##### 1）、关闭 firewalld

```
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```

##### 2)、关闭 selinux

```
getenforce
setenforce 0

vim /etc/selinux/config
SELINUX=disabled

# 重启虚拟机
reboot
```

### 2、修改 hostname 和 hosts（所有虚拟机都执行）

##### 	1）、vim /etc/hostname

```
hp1
```

##### 	2）、使 hostname 生效

- 重启虚拟机

  ```
  reboot
  ```

- 或执行以下命令，然后重新登录

  ```
  hostnamectl set-hostname hp1
  ```

##### 	3）、vim /etc/hosts

```
192.168.1.11 hp1
192.168.1.12 hp2
192.168.1.13 hp3
```


### 3、设置免密

##### 	1）、生成秘钥对

```
ssh-keygen
```

##### 	2）、将公钥拷贝到其他节点实现免密

```
ssh-copy-id hp1
ssh-copy-id hp2
ssh-copy-id hp3
```

##### 4、安装 mysql 数据库

##### 	1）、安装 mariadb-server

```
yum install -y mariadb-server
```

##### 	2）、开机启用、启动 mariadb、查看状态

```
systemctl enable mariadb
systemctl start mariadb
systemctl status mariadb
```

##### 	3）、初始化 mysql 数据库

```
mysql_secure_installation

Enter current password for root (enter for none): (enter)
Change the root password? [Y/n] (y)
New password:  (qwe)
Re-enter new password: (qwe)
Remove anonymous users? [Y/n] (n)
Disallow root login remotely? [Y/n] (n)
Remove test database and access to it? [Y/n] (y)
Reload privilege tables now? [Y/n] (y)
```

##### 	4）、连接测试

```
mysql -uroot -pqwe
```

## 二、开始安装

### 1、解压相关软件包

```
mkdir -p /usr/local/src
tar -zxvf hadoop-2.6.0.tar.gz -C /usr/local/src
mv /usr/local/src/hadoop-2.6.0 /usr/local/src/hadoop

tar -zxvf apache-hive-1.1.0-bin.tar.gz -C /usr/local/src
mv /usr/local/src/apache-hive-1.1.0-bin /usr/local/src/hive

tar -zxvf hbase-1.2.0-bin.tar.gz -C /usr/local/src
mv /usr/local/src/hbase-1.2.0 /usr/local/src/hbase

tar -zxvf sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz -C /usr/local/src
mv /usr/local/src/sqoop-1.4.7.bin__hadoop-2.6.0 /usr/local/src/sqoop
               
tar -zxvf jdk1.8.0_111.tar.gz -C /usr/local/src
mv /usr/local/src/jdk1.8.0_111 /usr/local/src/jdk

tar -zxvf zookeeper-3.4.5.tar.gz -C /usr/local/src
mv /usr/local/src/zookeeper-3.4.5 /usr/local/src/zookeeper
```

### 2、修改环境变量

##### 1)、vim /etc/profile （在文件后面追加）

```
export JAVA_HOME=/usr/local/src/jdk
export HADOOP_HOME=/usr/local/src/hadoop
export ZOOKEEPER_HOME=/usr/local/src/zookeeper
export HBASE_HOME=/usr/local/src/hbase
export SQOOP_HOME=/usr/local/src/sqoop
export HIVE=/usr/local/src/hive

export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin:$HBASE_HOME/bin:$SQOOP_HOME/bin:$HIVE/bin
```

> 使环境变量生效

```
source /etc/profile
```

> 注意：别漏掉PATH=$PATH，否则 linux 环境会出问题

```
scp -r /etc/profile hp2:/etc
scp -r /etc/profile hp3:/etc

# 分别使环境变量生效
source /etc/profile
```

### 3、配置 zookeeper

##### 1）、进入配置文件目录

```
cd /usr/local/src/zookeeper/conf
```

##### 2）、复制配置文件

```
cp zoo_sample.cfg zoo.cfg
```

##### 3）、修改配置文件，（vim zoo.cfg）

> dataDir 数据存放路径

```
tickTime=2000
clientPort=2181
initLimit=5
syncLimit=2
<!-- 以上可选 -->

dataDir=/usr/local/src/zookeeper/data

server.0=hp1:2888:3888
server.1=hp2:2888:3888
server.2=hp3:2888:3888
```

> 接下来分别在 hp1、hp2、hp3 节点上的/usr/local/src/zookeeper/data 目录下创建一
> 个名为 myid 的文件，并在hp1节点上的 myid 文件里输入 0，在hp2节点上的myid输入
> 1，在hp3节点上的 myid 文件里输入 2

```
mkdir /usr/local/src/zookeeper/data
echo 0 >> /usr/local/src/zookeeper/data/myid
```

##### 4)、拷贝jdk、 zookeeper 配置到所有其他节点

```
scp -r /usr/local/src/jdk hp2:/usr/local/src/
scp -r /usr/local/src/jdk hp3:/usr/local/src/

scp -r /usr/local/src/zookeeper hp2:/usr/local/src/
scp -r /usr/local/src/zookeeper hp3:/usr/local/src/
```

##### 5）、每个节点启动zkserver

```
zkServer.sh start
```

```
[root@hp1 ~]# jps
12427 Jps
12380 QuorumPeerMain
```

> 验证 ZooKeeper 服务，三台节点必须是 1 个 leader 2 个 follower 的状态才算配置正确

```
zkServer.sh status
```

```
[root@hp2 ~]# zkServer.sh status 
JMX enabled by default
Using config: /usr/local/src/zookeeper/bin/../conf/zoo.cfg
Mode: leader
```

```
[root@hp1 ~]# zkServer.sh status 
JMX enabled by default
Using config: /usr/local/src/zookeeper/bin/../conf/zoo.cfg
Mode: follower
```

```
[root@hp3 ~]# zkServer.sh status 
JMX enabled by default
Using config: /usr/local/src/zookeeper/bin/../conf/zoo.cfg
Mode: follower
```

### 4、安装 hadoop

##### 1）、进入配置文件目录

```
cd /usr/local/src/hadoop/etc/hadoop
```

##### 2）、编辑  hadoop-env.sh，添加以下内容。（vim hadoop-env.sh）

```
export JAVA_HOME=/usr/local/src/jdk
export HADOOP_HOME=/usr/local/src/hadoop
```

##### 3）、编辑 slaves，将 slave 的主机名写入到该文件中。（vim slaves）

```
hp3
```

##### 4）、编辑 core-site.xml，（vim core-site.xml）

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://ns</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/src/hadoop/tmp</value>
    </property>
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>hp1:2181,hp2:2181,hp3:2181</value>
    </property>
</configuration>
```

##### 5）、编辑 hdfs-site.xml，（vim hdfs-site.xml）

```
<configuration>
    <!--指定hdfs的nameservice为ns，需要和core-site.xml中的保持一致 -->
    <property>
        <name>dfs.nameservices</name>
        <value>ns</value>
    </property>
    <!-- ns下面有两个NameNode，分别是nn1，nn2 -->
    <property>
        <name>dfs.ha.namenodes.ns</name>
        <value>nn1,nn2</value>
    </property>
    <!-- nn1的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.ns.nn1</name>
        <value>hp1:9000</value>
    </property>
    <!-- nn1的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.ns.nn1</name>
        <value>hp1:50070</value>
    </property>
    <!-- nn2的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.ns.nn2</name>
        <value>hp2:9000</value>
    </property>
    <!-- nn2的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.ns.nn2</name>
        <value>hp2:50070</value>
    </property>
    <!-- 指定NameNode的元数据在JournalNode上的存放位置 -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://hp1:8485;hp2:8485;hp3:8485/ns</value>
    </property>
    <!-- 指定JournalNode在本地磁盘存放数据的位置 -->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/usr/local/src/hadoop/hdfs/journal</value>
    </property>
    <!-- 开启NameNode故障时自动切换 -->
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
    <!-- 配置失败自动切换实现方式 -->
    <property>
        <name>dfs.client.failover.proxy.provider.ns</name>
       <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
        </property>
        <!-- 配置隔离机制，如果ssh是默认22端口，value直接写sshfence即可 -->
        <property>
        <name>dfs.ha.fencing.methods</name>
        <value>
        sshfence
        shell(/bin/true)
        </value>
    </property>
    <!-- 使用隔离机制时需要ssh免登陆 -->
    <property>
        <name>dfs.ha.fencing.ssh.private-key-files</name>
        <value>/root/.ssh/id_rsa</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/src/hadoop/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.name.dir</name>
        <value>file:/usr/local/src/hadoop/dfs/data</value>
    </property>
</configuration>
```

##### 6）、复制一份 mapred-site.xml 配置文件.

```
cp mapred-site.xml.template mapred-site.xml
```

##### 编辑配置文件 mapred-site.xml，（vim mapred-site.xml）

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

##### 7、）编辑 yarn-site 配置文件，（vim yarn-site.xml）

```
<configuration>
    <!-- 开启RM高可用 -->
    <property>
         <name>yarn.resourcemanager.ha.enabled</name>
         <value>true</value>
    </property>
    <!-- 指定RM的cluster id -->
    <property>
         <name>yarn.resourcemanager.cluster-id</name>
         <value>yrc</value>
    </property>
    <!-- 指定RM的名字 -->
    <property>
         <name>yarn.resourcemanager.ha.rm-ids</name>
         <value>rm1,rm2</value>
    </property>
    <!-- 分别指定RM的地址 -->
    <property>
         <name>yarn.resourcemanager.hostname.rm1</name>
         <value>hp1</value>
    </property>
    <property>
         <name>yarn.resourcemanager.hostname.rm2</name>
         <value>hp2</value>
    </property>
    <!-- 指定zk集群地址 -->
    <property>
         <name>yarn.resourcemanager.zk-address</name>
         <value>hp1:2181,hp2:2181,hp3:2181</value>
    </property>
    <property>
         <name>yarn.nodemanager.aux-services</name>
         <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

##### 8、）拷贝配置到所有其他节点

```
scp -r /usr/local/src/hadoop hp2:/usr/local/src/
scp -r /usr/local/src/hadoop hp3:/usr/local/src/
```

> 注意：hp2、hp3 的 /usr/local/src/ 目录不存在时，需先创建

##### 9)、HA 模式启动服务

- 1、首先启动各个节点的 Zookeeper，在各个节点执行以下命令

  ```
  zkServer.sh start
  ```

  ```
  [root@hp1 ~]# jps
  2147 QuorumPeerMain
  2277 Jps
  ```

- 2、在各个节点启动 journalnode 服务，在各个节点执行以下命令

  ```
  hadoop-daemon.sh start journalnode
  ```

  ```
  [root@hp1 ~]# jps
  2336 Jps
  2147 QuorumPeerMain
  2301 JournalNode
  ```

- 3、在主 namenode 节点（hp1）格式化 namenode 和 journalnode 目录

  ```
  hdfs namenode -format
  ```
  
  ```
  19/11/05 09:30:11 INFO common.Storage: Storage directory /usr/local/src/hadoop/dfs/name has been successfully formatted.
  19/11/05 09:30:13 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
  19/11/05 09:30:13 INFO util.ExitUtil: Exiting with status 0
  19/11/05 09:30:13 INFO namenode.NameNode: SHUTDOWN_MSG: 
  ```

> 成功的话，会看到 "successfully formatted"  或  "Exitting with status 0" 的提示，若为 "Exitting with status 1" 则是出错。

- 4、在某一个 namenode 节点（hp1）执行如下命令，创建命名空间

  ```
  hdfs zkfc -formatZK
  ```

  ```
  Proceed formatting /hadoop-ha/ns? (Y or N) 19/11/05 09:31:16 INFO ha.ActiveStandbyElector: Session connected.
  y
  19/11/05 09:31:46 INFO ha.ActiveStandbyElector: Recursively deleting /hadoop-ha/ns from ZK...
  19/11/05 09:31:47 INFO ha.ActiveStandbyElector: Successfully deleted /hadoop-ha/ns from ZK.
  19/11/05 09:31:47 INFO ha.ActiveStandbyElector: Successfully created /hadoop-ha/ns in ZK.
  ```

- 5、在主 namenode 节点（hp1）启动 namenode 进程

  ```
  hadoop-daemon.sh start namenode
  ```

  ```
  [root@hp1 ~]# hadoop-daemon.sh start namenode
  starting namenode, logging to /usr/local/src/hadoop/logs/hadoop-root-namenode-hp1.out
  [root@hp1 ~]# jps
  2147 QuorumPeerMain
  2500 NameNode
  2596 JournalNode
  2630 Jps
  ```

- 6、在备 namenode 节点 （hp2）执行第一行命令，这个是把备 namenode 节点的目录格式化并把元数据从主 namenode 节点 copy 过来，并且这个命令不会把 journalnode 目录再格式化了！然后用第二个命令启动备 namenode 进程！

  ```
  hdfs namenode -bootstrapStandby
  ```

  ```
  # 19/11/05 09:34:46 INFO util.ExitUtil: Exiting with status 0
  ```

  ```
  hadoop-daemon.sh start namenode
  ```

  ```
  [root@hp2 ~]# jps
  1619 JournalNode
  1749 NameNode
  1785 Jps
  1533 QuorumPeerMain
  ```

- 7、在两个 namenode 节点（hp1,hp2）都执行以下命令

  ```
  hadoop-daemon.sh start zkfc
  ```

  ```
  2147 QuorumPeerMain
  2500 NameNode
  2596 JournalNode
  2681 DFSZKFailoverController
  2734 Jps
  ```

- 8、在所有 datanode 节点都执行以下命令启动 datanode

  ```
  hadoop-daemon.sh start datanode
  ```
  
  ```
  1665 DataNode
  1733 Jps
  1514 QuorumPeerMain
  1595 JournalNode
  ```
  
- 9、启动 yarn (hp1)

  ```
  start-yarn.sh
  ```

- 10、查看 namenode 状态

  ```
  hdfs haadmin -getServiceState nn1
  hdfs haadmin -getServiceState nn2
  ```

- 11、查看集群状态

  > 如果 Live datanode 不为 0，则说明集群启动成功

  ```
  hdfs dfsadmin -report
  ```

- 12、web 页面访问 hadoop

  > hp1 为对应的ip 地址

  - hadoop 地址
  
  [http://hp1:50070](http://hp1:50070)
  
  - yarn 地址
  
  [http://hp1:8088](http://hp1:8088)
  
  - hbase 地址
  
  [http://hp1:16010](http://hp1:16010)

### 5、hadoop 使用测试

```
mkdir input

echo "hello world">./input/test1.txt
echo "hello hadoop">./input/test2.txt

hdfs dfs -mkdir /in

hdfs dfs -put input/ /in

hadoop jar /usr/local/src/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar wordcount /in/input /out

hdfs dfs -cat /out/*
```

### 6、配置sqoop

##### 1）、进入配置文件目录

```
cd /usr/local/src/sqoop/conf
```

##### 2）、复制环境变量 sqoop-env.sh 文件

```
cp sqoop-env-template.sh sqoop-env.sh
```

##### 3）、修改配置文件 （vim sqoop-env.sh ）

> 没用到 Hive 和 HBase 可以不用配置相关项，使用时会弹出警告

```
export HADOOP_COMMON_HOME=/usr/local/src/hadoop
export HADOOP_MAPRED_HOME=/usr/local/src/hadoop
export HIVE_HOME=/usr/local/src/hive
export ZOOKEEPER_HOME=/usr/local/src/zookeeper
export ZOOCFGDIR=/usr/local/src/zookeeper/conf
export HBASE_HOME=/usr/local/src/hbase
```

##### 4）、 复制 mysql 数据库驱动包到指定文件路径

```
cp mysql-connector-java-5.1.46-bin.jar /usr/local/src/sqoop/lib/
```

##### 5）、验证

```
sqoop version
```

##### 6）、链接测试

```
sqoop list-databases --connect jdbc:mysql://hp1:3306/ --username root --password qwe
```

### 7、配置hbase

##### 1）、进入配置文件目录

```
cd /usr/local/src/hbase/conf
```

##### 2）、编辑（vim hbase-env.sh）

```
export JAVA_HOME=/usr/local/src/jdk
```

##### 3）、编辑 （vim hbase-site.xml）

```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hp1:9000/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>hp1,hp2,hp3</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/usr/local/src/zookeeper/data/</value>
  </property>
</configuration>
```

##### 4）、编辑 （vim regionservers）

```
hp1
hp2
hp3
```

##### 5）、将hbase所有配置拷贝到其他所有节点

```
scp -r /usr/local/src/hbase/ hp2:/usr/local/src/
scp -r /usr/local/src/hbase/ hp3:/usr/local/src/
```

##### 6）、启动 hbase

> 启动 HBase 时需要确保 hdfs 已经启动，使用命令 hdfs dfsadmin -report 查看以下 HDFS
> 集群是否正常

> 如果正常，在 master 节点上执行以下命令启动 HBase 集群:

```
start-hbase.sh
 	
3924 HMaster
4038 HRegionServer
```

##### 7）、测试

```
hbase shell
```

##### 8）、web 访问 hbase

```
http://hp1:16010
```

### 8、配置 hive

##### 1）、进入配置文件目录

```
cd /usr/local/src/hive/conf
```

##### 2）、复制配置文件

```
cp hive-env.sh.template hive-env.sh
cp hive-default.xml.template hive-site.xml

# 创建文件夹，hive-site.xml 中配置需要
mkdir -p /usr/local/src/hive/tmp
```

##### 3）、编辑 （vim hive-env.sh）

```
export JAVA_HOME=/usr/local/src/jdk
export HADOOP_HOME=/usr/local/src/hadoop
export HIVE_HOME=/usr/local/src/hive
export HIVE_CONF_DIR=/usr/local/src/hive/conf
```

##### 4)、编辑 （vim hive-site.xml）

```
# 注意修改数据库连接密码 qwe

<configuration>
	<property>
		<name>system:java.io.tmpdir</name>
		<value>/usr/local/src/hive/tmp</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://hp1:3306/hive?createDatabaseIfNotExist=true</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.jdbc.Driver</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>root</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>qwe</value>
	</property>
</configuration>
```

```
sed -i "s#\${system:java.io.tmpdir}/\${system:user.name}#/usr/local/src/hive/iotmp#g" /usr/local/src/hive/conf/hive-site.xml
```

##### 5）、 复制 mysql 数据库驱动包到指定文件路径

```
cp mysql-connector-java-5.1.46-bin.jar /usr/local/src/hive/lib/
```

##### 6）、在 hdfs 中创建下面的目录 ，并赋予所有权限

```
hdfs dfs -mkdir -p /usr/local/src/hive/warehouse
hdfs dfs -mkdir -p /usr/local/src/hive/tmp
hdfs dfs -mkdir -p /usr/local/src/hive/log
hdfs dfs -chmod -R 777 /usr/local/src/hive/warehouse
hdfs dfs -chmod -R 777 /usr/local/src/hive/tmp
hdfs dfs -chmod -R 777 /usr/local/src/hive/log
```

##### 7）、初始化 hive 

```
schematool -dbType mysql -initSchema

Metastore connection URL:	 jdbc:mysql://hp:3306/hive?createDatabaseIfNotExist=true
Metastore Connection Driver :	 com.mysql.jdbc.Driver
Metastore connection User:	 root
Starting metastore schema initialization to 1.1.0
Initialization script hive-schema-1.1.0.mysql.sql
Initialization script completed
schemaTool completed
```

> 如果没有以上返回，则初始化失败，见常见问题

##### 8）、安装hive到此结束，进入hive命令行

```
hive
```

##### 9）、常见问题

[常见问题](http://www.itkeyword.com/doc/985187200597055x509)

> 因为版本原因，需要重新拷贝一个 jar 包

```
rm -rf /usr/local/src/hadoop/share/hadoop/yarn/lib/jline-0.9.94.jar

cp /usr/local/src/hive/lib/jline-2.12.jar /usr/local/src/hadoop/share/hadoop/yarn/lib/
```

> 初始化后如果想要再重启服务

```
hive --service metastore &
hive --service hiveserver2 &
```

> 这个在1.4.7中需要配置，否则在执行数据导入到hive时会报错
>
> ```
> Could not load org.apache.hadoop.hive.conf.HiveConf. Make sure HIVE_CONF_DIR is set correctly.
> ```

```
cp /usr/local/src/hive/lib/hive-exec-1.1.0.jar /usr/local/src/sqoop/lib/
```

### 9、重启 hadoop 相关环境

##### 1、启动各个节点（hp1、hp2、hp3）的zookeper服务

```
zkServer.sh start
```

```
zkServer.sh status
```

##### 2、启动各个节点（hp1、hp2、hp3）的 journalnode 服务

```
hadoop-daemon.sh start journalnode
```

##### 3、启动主节点（hp1）的 namenode 服务

```
hadoop-daemon.sh start namenode
```

##### 4、启动备用节点（hp2）的 namenode 服务

```
hdfs namenode -bootstrapStandby
```

```
hadoop-daemon.sh start namenode
```

##### 5、启动各个节点（hp1、hp2、hp3）的 datanode 服务

```
hadoop-daemon.sh start datanode
```

##### 6、在两个 namenode 节点（hp1、hp2）分别启动zkfc服务

```
hadoop-daemon.sh start zkfc
```

##### 7、启动 yarn 服务

```
start-yarn.sh
```

