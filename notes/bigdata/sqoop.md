# sqoop 相关

### 一、介绍

> sqoop是hadoop和关系数据库服务器之间传送数据的工具。

[sqoop 学习](https://www.cnblogs.com/qingyunzong/p/8807252.html)

[sqoop 架构](https://www.jianshu.com/p/4250382abbc6)

##### 1、sqoop的主要功能

- 导入、迁入
  - 导入数据：mysql、oracle 导入数据到 hadoop 的 hdfs、hive、hbase 等数据存储系统
- 导出、迁出
  - 导出数据：从 hadoop 的文件系统中导出数据到关系数据库 mysql 等

##### 2、sqoop 与 hive

- sqoop:
  工具：本质就是迁移数据，迁移的方式：就是把sqoop的迁移命令转换成MR程序
- hive：

  工具：本质就是执行计算，依赖于HDFS存储数据，把SQL转换成MR程序

- 与以下组件可能打交道
  HDFS、MapReduce、YARN、ZooKeeper、Hive、HBase、MySQL

### 二、安装部署

##### 1、进入配置文件目录

```
cd /usr/local/src/sqoop/conf
```

##### 2、复制环境变量 sqoop-env.sh 文件

```
cp sqoop-env-template.sh sqoop-env.sh
```

##### 3、修改配置文件 （vim sqoop-env.sh ）

> 没用到 Hive 和 HBase 可以不用配置相关项，使用时会弹出警告

```
export HADOOP_COMMON_HOME=/usr/local/src/hadoop
export HADOOP_MAPRED_HOME=/usr/local/src/hadoop
export HIVE_HOME=/usr/local/src/hive
export ZOOKEEPER_HOME=/usr/local/src/zookeeper
export ZOOCFGDIR=/usr/local/src/zookeeper/conf
export HBASE_HOME=/usr/local/src/hbase
```

##### 4、 复制 mysql 数据库驱动包到指定文件路径

```
cp mysql-connector-java-5.1.46-bin.jar /usr/local/src/sqoop/lib/
```

##### 5、验证

```
sqoop version
```

##### 6、链接测试

```
sqoop list-databases --connect jdbc:mysql://hp:3306/ --username root --password qwe
```

### 三、使用

##### 1、列出MySQL数据有哪些数据库

```
sqoop list-databases --connect jdbc:mysql://hadoop1:3306/ --username root --password root
```

##### 2、列出MySQL中的某个数据库有哪些数据表：

```
sqoop list-tables --connect jdbc:mysql://hadoop1:3306/mysql --username root --password root
```

##### 3、创建一张跟mysql中的help_keyword表一样的hive表hk：

```
sqoop create-hive-table \
--connect jdbc:mysql://hadoop1:3306/mysql \
--username root \
--password root \
--table help_keyword \
--hive-table hk
```


##### 4、Sqoop的数据导入
import命令：

​		关系型数据库(Mysql、Oracle) -> Hadoop(HDFS、HBase、Hive), 每条记录可表示为文本、二进制文件或SequenceFile格式；

语法格式

```
sqoop import (generic-args) (import-args)
```

常用参数

```
--connect<jdbc-uri>	JDBC连接符: jdbc:mysql://node1/movie jdbc:oracle:thin:@//ndoe/movie
--connection-manager <class-name> 连接管理者
--hadoop-mapred-home <dir>	指定$HADOOP_MAPRED_HOME路径 conf中指定后无需设置
--dirver	JDBC驱动器类 比如com.mysql.jdbc.Driver
--username	数据库用户
--password	数据库密码
--password-alias <password-alias>	Credential provider password alias
--password-file <password-file>	设置用于存放认证的密码信息文件的路径
--table	导出的表名
--where	配合table使用
--target-dir	HDFS目录名
--as-textfile --as-parquetfile --as-avrodatafile --as-sequencefile	保存格式，默认text
-m, -num-mappers	启动的Map Task数目 任务并行度， 默认1
-e，--query	取数sql
--fields-terminated-by	分割符
--verbose	日志
--append	将数据追加到HDFS上一个已存在的数据集上
-P 从命令行输入密码
--password <password> 密码
--username <username> 账号
--verbose 打印流程信息
--connection-param-file <filename> 可选参数
```

查看数据库

```
sqoop list-databases \
--connect jdbc:mysql://hadoop1:3306/  \
--username root \
--password root

sqoop list-tables 
```

##### 1)、从RDBMS导入到HDFS中

> 普通导入：导入mysql库中的help_keyword的数据到HDFS上
> 导入的默认路径：/user/hadoop/help_keyword

```
sqoop import   \
--connect jdbc:mysql://hadoop1:3306/mysql   \
--username root  \
--password root   \
--table help_keyword   \
-m 1
```

查看导入的文件

```
hadoop fs -cat /user/hadoop/help_keyword/part-m-00000
```

导入： 指定分隔符和导入路径

```
sqoop import   \
--connect jdbc:mysql://hadoop1:3306/mysql   \
--username root  \
--password root   \
--table help_keyword   \
--target-dir /user/hadoop11/my_help_keyword1  \
--fields-terminated-by '\t'  \
-m 2
```

导入数据：带where条件

```
sqoop import   \
--connect jdbc:mysql://hadoop1:3306/mysql   \
--username root  \
--password root   \
--where "name='STRING' " \
--table help_keyword   \
--target-dir /sqoop/hadoop11/myoutport1  \
-m 1
```

查询指定列

```
sqoop import   \
--connect jdbc:mysql://hadoop1:3306/mysql   \
--username root  \
--password root   \
--columns "name" \
--where "name='STRING' " \
--table help_keyword  \
--target-dir /sqoop/hadoop11/myoutport22  \
-m 1
selct name from help_keyword where name = "string"
```

导入：指定自定义查询SQL

```
sqoop import   \
--connect jdbc:mysql://hadoop1:3306/  \
--username root  \
--password root   \
--target-dir /user/hadoop/myimport33_1  \
--query 'select help_keyword_id,name from mysql.help_keyword where $CONDITIONS and name = "STRING"' \
--split-by  help_keyword_id \
--fields-terminated-by '\t'  \
-m 4
```

##### 2）、把MySQL数据库中的表数据导入到Hive中

Sqoop 导入关系型数据到 hive 的过程是先导入到 hdfs，然后再 load 进入 hive

普通导入：数据存储在默认的default hive库中，表名就是对应的mysql的表名：

```
sqoop import   \
--connect jdbc:mysql://hadoop1:3306/mysql   \
--username root  \
--password root   \
--table help_keyword   \
--hive-import \
-m 1
```

导入过程
第一步：导入mysql.help_keyword的数据到hdfs的默认路径
第二步：自动仿造mysql.help_keyword去创建一张hive表, 创建在默认的default库中
第三步：把临时目录中的数据导入到hive表中

查看数据

```
hadoop fs -cat /user/hive/warehouse/help_keyword/part-m-00000
```

指定行分隔符和列分隔符，指定hive-import，指定覆盖导入，指定自动创建hive表，指定表名，指定删除中间结果数据目录

```
sqoop import  \
--connect jdbc:mysql://hadoop1:3306/mysql  \
--username root  \
--password root  \
--table help_keyword  \
--fields-terminated-by "\t"  \
--lines-terminated-by "\n"  \
--hive-import  \
--hive-overwrite  \
--create-hive-table  \
--delete-target-dir \
--hive-database  mydb_test \
--hive-table new_help_keyword
```

#报错原因是hive-import 当前这个导入命令。 sqoop会自动给创建hive的表。 但是不会自动创建不存在的库

手动创建mydb_test数据块

```
hive> create database mydb_test;
OK
Time taken: 6.147 seconds
hive> 
```

之后再执行上面的语句没有报错

查询一下

```
select * from new_help_keyword limit 10;
```

上面的导入语句等价于

```
sqoop import  \
--connect jdbc:mysql://hadoop1:3306/mysql  \
--username root  \
--password root  \
--table help_keyword  \
--fields-terminated-by "\t"  \
--lines-terminated-by "\n"  \
--hive-import  \
--hive-overwrite  \
--create-hive-table  \ 
--hive-table  mydb_test.new_help_keyword  \
--delete-target-dir
```

增量导入

```
--incremental append --check-column 检查的字段 --last-value 起始字段last-value + 1
```

执行增量导入之前，先清空hive数据库中的help_keyword表中的数据

```
truncate table help_keyword;

sqoop import   \
--connect jdbc:mysql://hadoop1:3306/mysql   \
--username root  \
--password root   \
--table help_keyword  \
--target-dir /user/hadoop/myimport_add  \
--incremental  append  \
--check-column  help_keyword_id \
--last-value 500  \
-m 1
```

##### 3）、把MySQL数据库中的表数据导入到hbase

```
sqoop import \
--connect jdbc:mysql://hadoop1:3306/mysql \
--username root \
--password root \
--table help_keyword \
--hbase-table new_help_keyword \
--column-family person \
--hbase-row-key help_keyword_id
```

export命令：Hadoop(HDFS、HBase、Hive)->关系型数据库(Mysql、Oracle)
export连接配置参数同import

```
sqoop export 
--connect jdbc:mysql://hadoop1:3306/mysql   \
--table data
--export-dir /user/x/data/ \
--username root \
--password root \
--update-key id \
--update-mode allowinsert
```

