# [大数据相关环境安装]()

## 一、基础准备

### 1、关闭防火墙等

```
略
```

### 2、修改 host 和 hostname

##### 	1）、vim /etc/host

```
192.168.1.11 hp1
192.168.1.12 hp2
192.168.1.13 hp3
```

##### 	2）、vim /etc/hostname

```
hp1
```

##### 	3）、使 hostname 生效

- 重启虚拟机

  ```
  reboot
  ```

- 或执行以下命令，然后重新登录

  ```
  hostnamectl set-hostname hp1
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

##### 1)、vim /etc/profile

```
export JAVA_HOME=/usr/local/src/jdk
export HADOOP_HOME=/usr/local/src/hadoop
export ZOOKEEPER_HOME=/usr/local/src/zookeeper
export HBASE_HOME=/usr/local/src/hbase
export SQOOP_HOME=/usr/local/src/sqoop
export HIVE=/usr/local/src/hive

export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin:$HBASE_HOME/bin:$SQOOP_HOME/bin:$HIVE/bin
```

> 注意：别漏掉PATH=$PATH，否则 linux 环境会出问题

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
> 个名为 myid 的文件，并在hp节点上的 myid 文件里输入 0，在hadoop2节点上的myid输入
> 1，在hadoop3节点上的 myid 文件里输入 2

```
mkdir /usr/local/src/zookeeper/data
echo 0 >> /usr/local/src/zookeeper/data/myid
```

##### 4)、拷贝 zookeeper 配置到所有其他节点

```
scp -r /usr/local/src/hadoop hp2:/usr/local/src/
scp -r /usr/local/src/hadoop hp3:/usr/local/src/
```

##### 5）、每个节点启动zkserver

```
zkServer.sh start

2402 DataNode
3667 QuorumPeerMain
3684 Jps
2106 NameNode
2539 SecondaryNameNode
2683 ResourceManager
3085 JobHistoryServer
2767 NodeManager
```

> 验证 ZooKeeper 服务，三台节点必须是 1 个 leader 2 个 follower 的状态才算配置正确
> zkServer.sh status 

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
  <!-- ha 当需要指定zk的时候
      <property>
          <!-- 指定zookeeper所在的节点 -->
          <name>ha.zookeeper.quorum</name>
          <value>hp1:2181,hp2:2181,hp3:2181</value>
      </property>
  -->
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hp1:9000</value>
  </property>
  <property>		
    <name>hadoop.tmp.dir</name>
    <value>file:/usr/local/hadoop/tmp</value>
    <description>Abase for other temporary directories.</description>
  </property>
</configuration>
```

##### 5）、编辑 hdfs-site.xml，（vim hdfs-site.xml）

```
<configuration>
  <!-- ha 这里仅当需要配置多个namenode时配置
    <property>
        <!--这里配置逻辑名称 -->
        <name>dfs.nameservices</name>
        <value>hpc</value>
    </property>
    <property>
        <!-- 配置namenode 的名称，多个用逗号分割  -->
        <name>dfs.ha.namenodes.hpc</name>
        <value>hp1,hp2</value>
    </property>
  -->
  <property>
    <name>dfs.namenode.http-address</name>
      <value>hp1:50070</value>
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
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>hp1:10020</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hp1:19888</value>
  </property>
</configuration>
```

##### 7、）编辑 yarn-site 配置文件，（vim yarn-site.xml）

```
<configuration>
  <property>
    <name>yarn.resoursemanager.hostname</name>
    <value>hp1</value>
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

> 注意：hp2、hp3 的文件 /usr/local/src/不存在，需先创建

##### 9)、HA 模式 (不是高可用略过该步)

- 启动 zookeeper

  ```
  zkServer.sh start
  ```

- 在其中一个 namenode 上格式化 zookeeper，将 hadoop 纳入 zookeeper 管理

  ```
  hdfs zkfc -formatZK
  ```

-  启动 journalnode ，需要启动所有节点的 journalnode 

  ```
  hdfs --daemon start journalnode
  ```

- 格式化 namenode

  > 如果有多个namenode名称，可以使用  hdfs namenode -format xxx 指定

  ```
  hdfs namenode -format 
  ```


- 启动 namenode

  ```
  hdfs --daemon start namenode
  ```

- 其他 namenode 同步

  >  如果是使用高可用方式配置的 namenode，使用下面命令同步(需要同步的 namenode 执行) 

  ```
  hdfs namenode -bootstrapStandby
  ```

  >  如果不是使用高可用方式配置的 namenode，使用下面命令同步 

  ```
  hdfs namenode -initializeSharedEdits
  ```

##### 10）、格式化 namenode

> 首次启动需要先在 master 节点上执行 namenode 的格式化操作

```
hdfs namenode -format
```

> 成功的话，会看到 "successfully formatted" 和 "Exitting with status 0" 的提示，若为 "Exitting with status 1" 则是出错。

##### 11）、完成 Hadoop 格式化后，在 master（namenode） 节点上启动 Hadoop 各个服务

- 启动 hdfs

```
start-dfs.sh

2641 Jps
2402 DataNode
2106 NameNode
2539 SecondaryNameNode
```

- 启动 yarn

```
start-yarn.sh

2402 DataNode
2836 Jps
2106 NameNode
2539 SecondaryNameNode
2683 ResourceManager
2767 NodeManager
```

- 启动 historyserver（可选）

```
mr-jobhistory-daemon.sh start historyserver

2402 DataNode
2106 NameNode
2539 SecondaryNameNode
2683 ResourceManager
3116 Jps
3085 JobHistoryServer
2767 NodeManager
```

> 通过命令 jps 可以查看各个节点所启动的进程。正确的话，在 Master 节点上可以看到
> NameNode、ResourceManager、SecondaryNameNode、JobHistoryServer 进程。
>
> 在 Slave 节点可以看到 DataNode 和 NodeManager 进程。

- 查看 DataNode 是否正常启动，在 master 节点执行以下命令

  ```
  hdfs dfsadmin -report
  ```

  > 如果 Live datanode 不为 0，则说明集群启动成功，

- web 访问 hadoop

  ```
  http://hp:50070 或 http://192.168.1.11:50070
  ```

### 5、配置sqoop

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
sqoop list-databases --connect jdbc:mysql://hp:3306/ --username root --password qwe
```

### 6、hadoop 使用测试

```
mkdir input
cd input
echo "hello world">test1.txt
echo "hello hadoop">test2.txt
cd ..
./bin/hadoop dfs -put input in
./bin/hadoop jar build/hadoop-0.20.2-examples.jar wordcount in out
./bin/hadoop dfs -cat out/*
```

> Hadoop配置文件说明
> Hadoop 的运行方式是由配置文件决定的（运行 Hadoop 时会读取配置文件），因此如果需要从伪分布式模式切换回非分布式模式，需要删除 core-site.xml 中的配置项。
>
> 此外，伪分布式虽然只需要配置 fs.defaultFS 和 dfs.replication 就可以运行（官方教程如此），不过若没有配置 hadoop.tmp.dir 参数，则默认使用的临时目录为 /tmp/hadoo-hadoop，而这个目录在重启时有可能被系统清理掉，导致必须重新执行 format 才行。所以我们进行了设置，同时也指定 dfs.namenode.name.dir 和 dfs.datanode.data.dir，否则在接下来的步骤中可能会出错。

### 7、配置hbase

##### 1）、进入配置文件目录

```
cd /usr/local/src/hbase/conf
```

##### 2）、编辑（vim hbase-env.sh）

```
export JAVA_HOME=/usr/local/src/jdk
export HBASE_MANAGES_ZK=false
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

##### 5）、启动 hbase

> 启动 HBase 时需要确保 hdfs 已经启动，使用命令 hdfs dfsadmin -report 查看以下 HDFS
> 集群是否正常

> 如果正常，在 master 节点上执行以下命令启动 HBase 集群:

```
start-hbase.sh
 	
2402 DataNode
3667 QuorumPeerMain
4115 Jps
3924 HMaster
4038 HRegionServer
2106 NameNode
2539 SecondaryNameNode
2683 ResourceManager
3085 JobHistoryServer
2767 NodeManager
```

##### 6）、web 访问 hbase

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
#数据库连接密码 qwe

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

> 如果没有以上返回，则初始化失败

[常见问题](http://www.itkeyword.com/doc/985187200597055x509)

```
rm /usr/local/src/hadoop/share/hadoop/yarn/lib/jline-0.9.94.jar

cp /usr/local/src/hive/lib/jline-2.12.jar /usr/local/src/hadoop/share/hadoop/yarn/lib/
```

##### 8）、安装hive到此结束，进入hive命令行

```
hive
```

