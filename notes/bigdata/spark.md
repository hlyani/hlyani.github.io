# spark 相关

### 一、安装 jdk

```
curl -O https://download.oracle.com/otn/java/jdk/8u221-b11/230deb18db3e4014bb8e3e8324f81b43/jdk-8u221-linux-x64.tar.gz?AuthParam=1571023421_c1f05abfd1ce9b457c29b0565494cc83

tar -zxvf jdk-8u221-linux-x64.tar.gz

vim /etc/profile
export JAVA_HOME=/root/jdk1.8.0_221
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile
```

### 二、hadoop安装

##### 1、下载

```
curl -O https://www-us.apache.org/dist/hadoop/common/hadoop-3.2.1/hadoop-3.2.1-src.tar.gz
tar -zxvf hadoop-3.2.1-src.tar.gz
```

##### 2、配置

```
cd /root/hadoop-3.2.1/etc/hadoop
```

###### mapred-site.xml

``` mapred-site.xml
exprot JAVA_HOME=${JAVA_HOME}
```

###### hdfs-site.xml

```hdfs-site.xml
<configuration>
  <property>
    <name>dfs.namenode.http-address</name>
      <value>master:50070</value>
  </property>
  <property>
    <name>dfs.replication</name>
      <value>2</value>
  </property>
  <property>
      <name>dfs.permissions</name>
      <value>true</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
      <value>file:/root/hadoop-3.2.1/dfs/name</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
      <value>file:/root/hadoop-3.2.1/dfs/data</value>
  </property>
</configuration>
```

###### yarn-env.sh

```yarn-env.sh
export JAVA_HOME=/root/jdk1.8.0_221
```

###### yarn-site.xml

```yarn-site.xml
<configuration>
  <property>
    <name>yarn.resoursemanager.hostname</name>
    <value>master</value>
  </property>
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
        <value>master:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>master:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>master:8035</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>master:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>master:8088</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>30720</value>
    </property>
</configuration>
```

###### works

```works
node1
node2
```

###### core-site.xml

```core-site.xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://master:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>file:/usr/local/hadoop/tmp</value>
    <description>Abase for other temporary directories.</description>
  </property>
</configuration>
```

###### mapred-site.xml

``` mapred-site.xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>master:10020</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>master:19888</value>
  </property>
</configuration>
```

##### 3、启动

```
/root/hadoop-3.2.1/sbin/start-all.sh
```

## 三、安装spark

##### 1、下载

[spark下载地址]: http://spark.apache.org/downloads.html

```
curl -O https://www.apache.org/dyn/closer.lua/spark/spark-2.4.4/spark-2.4.4-bin-hadoop2.7.tgz

tar -zxvf spark-2.4.4-bin-hadoop2.7.tgz
```

##### 2、配置

```
cd spark-2.4.4-bin-hadoop2.7/conf
cp spark-env.sh.template spark-env.sh
cp spark-defaults.conf.template spark-defaults.conf
cp slaves.template slaves
```

###### spark-env.sh

```spark-env.conf
export JAVA_HOME=/root/jdk1.8.0_221
export SPARK_MASTER_IP=192.168.0.128
```
###### slaves

```slaves
node1
node2
```

###### spark-defaults.conf

```spark-defaults.conf
spark.master                     spark://master:7077
spark.eventLog.enabled           true
spark.driver.cores               2
spark.driver.memory              2g
spark.executor.memory            7g
spark.rpc.message.maxSize        1024
```

##### 3、启动

```
/root/spark-2.4.4-bin-hadoop2.7/sbin/start-all.sh

jps
```

## 四、使用

##### 1、使用示例代码

```
spark-submit  --class org.apache.spark.examples.JavaWordCount /usr/local/src/spark/examples/jars/spark-examples_2.11-2.4.4.jar  /usr/local/src/spark/README.md
```

```
spark-submit  --class org.apache.spark.examples.JavaSparkPi /usr/local/src/spark/examples/jars/spark-examples_2.11-2.4.4.jar  /usr/local/src/spark/README.md
```

```
spark-submit /usr/local/src/spark/examples/src/main/python/pi.py 10
```

```
cd spark-2.4.4-bin-hadoop2.7/
./bin/spark-shell

[root@master spark-2.4.4-bin-hadoop2.7]# ./bin/pyspark 
Python 2.7.5 (default, Oct 30 2018, 23:45:53) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-36)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
19/10/14 14:36:04 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
19/10/14 14:36:05 WARN Utils: Service 'SparkUI' could not bind on port 4040. Attempting port 4041.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.4.4
      /_/

Using Python version 2.7.5 (default, Oct 30 2018 23:45:53)
SparkSession available as 'spark'.
```

[Running Spark on YARN]: https://spark.apache.org/docs/latest/running-on-yarn.html#spark-properties

![pi](../../imgs/pi.png)

###### vim /root/spark-2.4.4-bin-hadoop2.7/examples/src/main/python/pi.py

```
from __future__ import print_function

import sys
from random import random
from operator import add

from pyspark.sql import SparkSession


if __name__ == "__main__":
    """
        Usage: pi [partitions]
    """
    spark = SparkSession\
        .builder\
        .appName("PythonPi")\
        .getOrCreate()

    partitions = int(sys.argv[1]) if len(sys.argv) > 1 else 2
    n = 100000 * partitions

    def f(_):
        x = random() * 2 - 1
        y = random() * 2 - 1
        return 1 if x ** 2 + y ** 2 <= 1 else 0

    count = spark.sparkContext.parallelize(range(1, n + 1), partitions).map(f).reduce(add)
    print("Pi is roughly %f" % (4.0 * count / n))
    result = [4.0*count/n]
    spark.sparkContext.parallelize(result, 1).saveAsTextFile("/opt/hl")

    spark.stop()
```

```
textFile = spark.read.text("README.md")
textFile.count()
textFile.first()
linesWithSpark = textFile.filter(textFile.value.contains("Spark"))

hadoop dfsadmin -safemode leave

spark-submit --executor-memory 8G --num-executors 2 --master yarn --deploy-mode cluster --driver-memory 2G pie.py 10

yarn logs -applicationId application_1571131648061_0016
```

```
8080 spark
8088 yarn
50070 handoop
```

### 五、资源配置

 [Spark任务的core，executor，memory资源配置方法]( https://blog.51cto.com/10120275/2364992 ) 

##### 1、Spark应用当中术语的基本定义：

- **Partitions** : 分区是大型分布式数据集的一小部分。 Spark使用分区来管理数据，这些分区有助于并行化数据处理，并且使executor之间的数据交换最小化
- **Task**：任务是一个工作单元，可以在分布式数据集的分区上运行，并在单个Excutor上执行。并行执行的单位是任务级别。单个Stage中的Tasks可以并行执行
- **Executor**：在一个worker节点上为应用程序创建的JVM，Executor将巡行task的数据保存在内存或者磁盘中。每个应用都有自己的一些executors，单个节点可以运行多个Executor，并且一个应用可以跨多节点。Executor始终伴随Spark应用执行过程，并且以多线程方式运行任务。spark应用的executor个数可以通过SparkConf或者命令行 –num-executor进行配置
- **Cores**：CPU最基本的计算单元，一个CPU可以有一个或者多个core执行task任务，更多的core带来更高的计算效率，Spark中，cores决定了一个executor中并行task的个数
- **Cluster Manager**：cluster manager负责从集群中请求资源

##### 2、cluster模式执行的Spark任务包含了如下步骤：

1. driver端，SparkContext连接cluster manager（Standalone/Mesos/Yarn）
2. Cluster Manager在其他应用之间定位资源，只要executor执行并且能够相互通信，可以使用任何Cluster Manager
3. Spark获取集群中节点的Executor，每个应用都能够有自己的executor处理进程
4. 发送应用程序代码到executor中
5. SparkContext将Tasks发送到executors

以上步骤可以清晰看到executors个数和内存设置在spark中的重要作用。

##### 3、例子1

`硬件资源： 6 节点，每个节点16 cores, 64 GB 内存`
每个节点在计算资源时候，给操作系统和Hadoop的进程预留1core，1GB，所以每个节点剩下15个core和63GB
内存。
**core的个数**，决定一个executor能够并发任务的个数。所以通常认为，一个executor越多的并发任务能够得到更好的性能，但有研究显示一个应用并发任务超过5，导致更差的性能。所以core的个数暂设置为5个。
5个core是表明executor并发任务的能力，并不是说一个系统有多少个core，即使我们一个CPU有32个core，也设置5个core不变。
**executor个数**，接下来，一个executor分配 5 core,一个node有15 core，从而我们计算一个node上会有3 executor（15 / 5），然后通过每个node的executor个数得到整个任务可以分配的executors个数。
我们有6个节点，每个节点3个executor，6 × 3 = 18个executors，额外预留1个executor给AM，最终要配置17个executors。
最后spark-submit启动脚本中配置 –num-executors = 17
**memory**，配置每个executor的内存，一个node，3 executor， 63G内存可用，所以每个executor可配置内存为63 / 3 = 21G
从Spark的内存模型角度，Executor占用的内存分为两部分：ExecutorMemory和MemoryOverhead，预留出MemoryOverhead的内存量之后，才是ExecutorMemory的内存。
MemoryOverhead的计算公式： max(384M, 0.07 × spark.executor.memory)

因此 MemoryOverhead值为0.07 × 21G = 1.47G > 384M

最终executor的内存配置值为 21G – 1.47 ≈ 19 GB

至此， Cores = 5, Executors= 17, Executor Memory = 19 GB

### 六、RDD

[RDD](https://blog.csdn.net/dsdaasaaa/article/details/94181269)

##### 1、转换操作

>  RDD 的转换操作是返回新的 RDD 的操作。转换出来的 RDD 是惰性求值的，只有在行动操作中用到这些 RDD 时才会被计算。 

```
rdd1={1, 2, 5, 3}，rdd2={3,4,5}
```

```
rdd1.flatMap(x=>x.to(3))
1 -> {1,2,3}
2 -> {2,3}
3 -> {}
3 -> {3}
123233
```
```
spark-shell
var rdd1 = sc.makeRDD(Seq(10, 4, 2, 12, 3))
rdd1.flatMap(x=>x.to(4)).collect()
```


| 函数名         | 作用                                                         | 示例                     | 结果                 |
| :------------- | :----------------------------------------------------------- | :----------------------- | :------------------- |
| map()          | 将函数应用于 RDD 的每个元素，返回值是新的 RDD，有返回值      | rdd1.map(x=>x+l)         | {2,3,4,4}            |
| flatMap()      | 将函数应用于 RDD 的每个元素，将元素数据进行拆分，变成迭代器，返回值是新的 RDD，压平操作，先map后flat，将函数应用于RDD中的每一项，对于每一项都产生一个集合，并将集合中的元素压扁成一个集合。 | rdd1.flatMap(x=>x.to(3)) | {1,2,3,2,3,3,3}      |
| filter()       | 函数会过滤掉不符合条件的元素，返回值是新的 RDD               | rdd1.filter(x=>x!=1)     | {2,3,3}              |
| distinct()     | 将 RDD 里的元素进行去重操作                                  | rdd1.distinct()          | (1,2,3)              |
| union()        | 生成包含两个 RDD 所有元素的新的 RDD，并集                    | rdd1.union(rdd2)         | {1,2,3,3,3,4,5}      |
| intersection() | 求出两个 RDD 的共同元素，交集                                | rdd1.intersection(rdd2)  | {3}                  |
| subtract()     | 将原 RDD 里和参数 RDD 里相同的元素去掉                       | rdd1.subtract(rdd2)      | {1,2}                |
| cartesian()    | 求两个 RDD 的笛卡儿积                                        | rdd1.cartesian(rdd2)     | {(1,3),(1,4)……(3,5)} |
| groupBy()      | 分组                                                         |                          |                      |
| sortBy()       | 排序                                                         |                          |                      |

##### 2、行动操作

>  行动操作用于执行计算并按指定的方式输出结果。行动操作接受 RDD，但是返回非 RDD，即输出一个值或者结果。在 RDD 执行过程中，真正的计算发生在行动操作 

```
rdd={1,2,3,3}
```

| 函数名                   | 作用                                                         | 示例                                  | 结果                 |
| :----------------------- | :----------------------------------------------------------- | :------------------------------------ | :------------------- |
| collect()                | 返回 RDD 的所有元素                                          | rdd.collect()                         | {1,2,3,3}            |
| count()                  | RDD 里元素的个数                                             | rdd.count()                           | 4                    |
| countByValue()           | 各元素在 RDD 中的出现次数                                    | rdd.countByValue()                    | {(1,1),(2,1),(3,2})} |
| take(num)                | 从 RDD 中返回 num 个元素                                     | rdd.take(2)                           | {1,2}                |
| top(num)                 | 从 RDD 中，按照默认（降序）或者指定的排序返回最前面的 num 个元素 | rdd.top(2)                            | {3,3}                |
| reduce()                 | 并行整合所有 RDD 数据，如求和操作                            | rdd.reduce((x,y)=>x+y)                | 9                    |
| reduceByKey()            |                                                              |                                       |                      |
| groupByKey()             | 根据key进行分组                                              |                                       |                      |
| fold(zero)(func)         | 和 reduce() 功能一样，但需要提供初始值                       | rdd.fold(0)((x,y)=>x+y)               | 9                    |
| foreach(func)            | 对 RDD 的每个元素都使用特定函数，没有返回值                  | rdd1.foreach(x=>printIn(x))           | 打印每一个元素       |
| saveAsTextFile(path)     | 将数据集的元素，以文本的形式保存到文件系统中                 | rdd1.saveAsTextFile(file://home/test) |                      |
| saveAsSequenceFile(path) | 将数据集的元素，以顺序文件格式保存到指 定的目录下            | saveAsSequenceFile(hdfs://home/test)  |                      |
| first                    | 返回第一个元素                                               | rdd1.first()                          | 1                    |
| cogroup()                | 表示将多个rdd的同一个key对应的value组合到一起                |                                       |                      |

### 七、示例代码

> pi

```
from __future__ import print_function

import sys
from random import random
from operator import add

from pyspark.sql import SparkSession


if __name__ == "__main__":
    """
        Usage: pi [partitions]
    """
    spark = SparkSession\
        .builder\
        .appName("PythonPi")\
        .getOrCreate()

    partitions = int(sys.argv[1]) if len(sys.argv) > 1 else 2
    n = 100000 * partitions

    def f(_):
        x = random() * 2 - 1
        y = random() * 2 - 1
        return 1 if x ** 2 + y ** 2 <= 1 else 0

    count = spark.sparkContext.parallelize(range(1, n + 1), partitions).map(f).reduce(add)
    print("Pi is roughly %f" % (4.0 * count / n))

    spark.stop()
```

> wordcount

```
from __future__ import print_function

import sys
from operator import add

from pyspark.sql import SparkSession


if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: wordcount <file>", file=sys.stderr)
        sys.exit(-1)

    spark = SparkSession\
        .builder\
        .appName("PythonWordCount")\
        .getOrCreate()

    lines = spark.read.text(sys.argv[1]).rdd.map(lambda r: r[0])
    counts = lines.flatMap(lambda x: x.split(' ')) \
                  .map(lambda x: (x, 1)) \
                  .reduceByKey(add)
    output = counts.collect()
    for (word, count) in output:
        print("%s: %i" % (word, count))

    spark.stop()
```

##### 八、pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.hl</groupId>
    <artifactId>SparkPi</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>2.4.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.11</artifactId>
            <version>2.4.4</version>
        </dependency>
    </dependencies>
</project>
```

> VM options:

```
-Dspark.master=local
```

