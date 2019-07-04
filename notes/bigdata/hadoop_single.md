# Hadoop 单机伪分布式部署

##### 1、安装软件（如果没有）

```
apt update
apt -y install vim openssh-server
```

##### 2、免密

> 修改ssh配置文件

```
vim /etc/ssh/sshd_config

...
PasswordAuthentication yes
....
PermitRootLogin yes
...
```

> 重启ssh服务

```
service ssh restart
```

> 生成秘钥对（根据提示回车）

```
ssh-keygen
```

> 将公钥拷贝到免密节点

```
ssh-copy-id hadoop1
```

> 验证（ssh连接如果没提示输入密码，则免密成功）

```
ssh hadoop1
```

##### 3、解压相关包

```
tar -zxvf jdk1.8.0_111.tar.gz
tar -zxvf hadoop-2.7.3.tar.gz
```

##### 4、修改环境变量

```
vim ~/.bashrc
export JAVA_HOME=/opt/jdk1.8.0_111
export HADOOP_HOME=/opt/hadoop-2.7.3
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

> 使添加环境变量生效

```
source /etc/profile
```

##### 5、验证

```
jps
hadoop version
java -version
hadoop version
```

##### 6、单机版hadoop测试

```
mkdir input
hadoop jar /opt/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep input output 'dfs[a-z.]+'
hadoop jar /opt/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep /root/hp output 'hello'
cat output/*
```

##### 7、编辑hosts、hostname

```
vim /etc/hosts
...
192.169.1.1 hadoop1
...

vim /etc/hostname
hp

hostnamectl set-hostname hp 或者 reboot
```

##### 8、解压相关软件包

```
cd /opt
tar -zxvf jdk1.8.0_111.tar.gz
tar -zxvf hadoop-2.7.3.tar.gz
```

##### 9、配置hadoop

```
cd hadoop-2.7.3/etc/hadoop
```

```
vim hadoop-env.sh

export JAVA_HOME=/opt/jdk1.8.0_111
```

> 将 slave 的主机名写入到该文件(这里是单节点伪分布式所以只需要加入本机host)

```
vim slaves

hadoop1
```

> 编辑相关配置文件

```
vim core-site.xml

...
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop1:9000</value>
  </property>
　<!-- 指定hadoop运行时产生文件的存储目录 -->
  <property>		
    <name>hadoop.tmp.dir</name>
    <value>file:/usr/local/hadoop/tmp</value>
    <description>Abase for other temporary directories.</description>
  </property>
</configuration>
```

```
vim hdfs-site.xml

...
<configuration>
  <property>
    <name>dfs.namenode.http-address</name>
      <value>hadoop1:50070</value>
  </property>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
      <value>hadoop1:50090</value>
  </property>
  <!-- 指定HDFS副本的数量 -->
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

##### 10、启动hadoop

> 首次启动需要先在master节点（这里的hadoop1）上执行namenode的格式化操作,成功的话，会看到"Exitting with status 0"的提示，若为"Exitting with status 1"则是出错。

```
hdfs namenode -format
```

> 完成 Hadoop 格式化后，在namenode节点上启动Hadoop各个服务,使用jps命令验证相关服务是否运行起来。

```
start-dfs.sh
```

```
jps
------
58993 NameNode
59601 Jps
59459 SecondaryNameNode
59304 DataNode
```
> 另外还需要在 Master 节点上通过命令 hdfs dfsadmin -report
> 查看 DataNode 是否正常启动，如果 Live datanode 不为 0，则说明集群启动成功，

```
 hdfs dfsadmin -report
```

* NameNode结点                http://hp:50070
* SecondaryNameNode    http://hp:50090

##### 11、运行hadoop伪分布式实例

> 单机模式，以grep例子读取的是本地数据，伪分布式读取的则是HDFS上的数据，要使用HDFS，首先要在

> HDFS中创建用户目录

```
hdfs dfs -mkdir -p /user/hadoop
hdfs dfs -mkdir input
```

> 传输文件达到input目录

```
hdfs dfs -put ./etc/hadoop/*.xml /input 
```

> 查看

```
hdfs dfs -ls /input
```

```
hadoop jar /opt/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep /input output 'dfs[a-z.]+'
hdfs dfs -cat output/*
```

> 取回本地

```
hdfs dfs -get output/*
```

> 在运行mapreducer前，不能有output目录，如果已经存在，则会报错误 "output already exists"，要先删除output目录

```
hdfs dfs -rm -r output
```

> 关闭hadoop：

```
sbin/stop-dfs.sh
```

##### 12、关于yarn，伪分布式不启动yarn也可以，一般不会影响程序执行

```
cp mapred-site.xml.template mapred-site.xml
```

```
vim mapred-site.xml

<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <!--
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>hp:10020</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hp:19888</value>
  </property>
  -->
</configuration>
```

```
vim yarn-site.xml

<configuration>
<!--
  <property>
    <name>yarn.resoursemanager.hostname</name>
    <value>hp</value>
  </property>
  -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  <oproperty>
</configuration>
```

##### 13、如果是集群，拷贝配置到所有其他节点，伪分布式跳过此步

##### 14、开启yarn

```
start-yarn.sh
```

```
jps
------
58993 NameNode
59649 ResourceManager
59459 SecondaryNameNode
60070 Jps
59767 NodeManager
59304 DataNode
```
> 启动yarn有个好处是可以通过web界面查看任务运行情况
> http://192.168.0.10:8088/cluster

> YARN 主要是为集群提供更好的资源管理与任务调度，然而这在单机上体现不出价
> 值，反而会使程序跑得稍慢些。因此在单机上是否开启 YARN 就看实际情况了。
> • 不启动 YARN 需重命名 mapred-site.xml
> • 如果不想启动 YARN，务必把配置文件 mapred-site.xml 重命名，改成 mapredsite.xml.template，需要用时改回来就行。否则在该配置文件存在，而未开启
> YARN 的情况下，运行程序会提示 “Retrying connect to server:
> 0.0.0.0/0.0.0.0:8032” 的错误，这也是为何该配置文件初始文件名为 mapredsite.xml.template。

> 关闭yarn：

```
stop-yarn.sh
```

> 通过命令 jps 可以查看各个节点所启动的进程。正确的话，在 Master 节点上可以看到
> NameNode,ResourceManager,SecondaryNameNode,JobHistoryServer 进程。
> 在 Slave 节点可以看到 DataNode 和 NodeManager 进程。

##### 15、开启历史服务器，才能在web中查看任务运行情况

```
mr-jobhistory-daemon.sh start historyserver
```

```
jps
------
58993 NameNode
59649 ResourceManager
60147 Jps
59459 SecondaryNameNode
59767 NodeManager
59304 DataNode
60108 JobHistoryServer
```

##### 16、验证hadoop

> 另外还需要在Master节点(hadoop1)上通过命令

```
hdfs dfsadmin -report
```

> 查看DataNode是否正常启动，如果Live datanode不为0，则说明集群启动成功

* HDFS管理界面(NameNode结点)            http://hadoop1:50070
* (SecondaryNameNode)                          http://hadoop1:50090
* ResourceManager管理界面（yarn）   http://hadoop1:8088
* NameNode结点                                        http://hp:50070
* SecondaryNameNode                            http://hp:50090
* cluser                                                          http://192.168.0.10:8088/cluster

##### 17、系统正常启动，跑个程序试试
[hdfs 常用命令](http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html)

```
mkdir input
cd input
echo "hello world">test1.txt
echo "hello hadoop">test2.txt
cd ..
bin/hadoop dfs -put input in
bin/hadoop jar build/hadoop-0.20.2-examples.jar wordcount in out
bin/hadoop dfs -cat out/*
```

> Hadoop配置文件说明
> Hadoop 的运行方式是由配置文件决定的（运行 Hadoop 时会读取配置文件），因此如果需要从伪分布式模式切换回非分布式模式，需要删除 core-site.xml 中的配置项。

> 此外，伪分布式虽然只需要配置 fs.defaultFS 和 dfs.replication 就可以运行（官方教程如此），不过若没有配置 hadoop.tmp.dir 参数，则默认使用的临时目录为 /tmp/hadoo-hadoop，而这个目录在重启时有可能被系统清理掉，导致必须重新执行 format 才行。所以我们进行了设置，同时也指定 dfs.namenode.name.dir 和 dfs.datanode.data.dir，否则在接下来的步骤中可能会出错。

##### 18、python利用streaming编写mapreduce程序

```
#先编写一个map.py

# coding: utf8
import sys

for line in sys.stdin:
   line = line.strip()
   film_d = line.split(";")
   print(film_d[0])
```

```
#再编写一个red.py：

# coding: utf8
import sys

cur_film = "惊天魔盗团2"
cur_count = 0
for line in sys.stdin:
    if cur_film  in line:
        cur_count  += 1
print('%s总共出现了，%s次。' % ( cur_film,cur_count))
```

```
hdfs dfs -put film-csv.txt /

chmod 777 mapper.py 
chmod 777 reducer.py

hadoop jar /opt/hadoop-2.7.3/share/hadoop/tools/lib/hadoop-streaming-2.7.3.jar -file /root/mapper.py -mapper "python mapper.py" -file /root/reducer.py -reducer "python reducer.py" -input /film-csv.txt -output /output
```

##### 19、实例

```
#查看帮助命令
hdfs dfs -help
```

```
#创建一个数据导入文件夹
hdfs dfs -mkdir -p /data/input
```

```
#在本地创建两个文本，并加入有规律内容
echo "hello world">test1.txt
echo "hello hadoop">test2.txt
```

```
#将文件上传至hdfs上
hdfs dfs -put ./*.txt /data/input
```

```
#查看hdfs上的文件
hdfs dfs -ls /data/input/
```

```
#运行wordcunt（grep）方法进行计算

hadoop jar /opt/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount /data/input/ output

#hadoop jar /opt/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep /data/input/ output 'hello'
```

```
#查看运行结果
hdfs dfs  -cat output/*
```

```
#将结果取回本地
hdfs dfs -get output ./output
```

```
#删除hdfs上的文件或文件夹
hdfs dfs -rm -r output
```

