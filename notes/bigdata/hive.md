# Hive 相关

## 一、hive相关资料

[](http://blog.csdn.net/u013310025/article/details/70306421)
[](https://www.cnblogs.com/guanhao/p/5641675.html)
[](http://blog.csdn.net/wisgood/article/details/40560799)
[](http://blog.csdn.net/seven_zhao/article/details/46520229)

## 二、安装

##### 1、获取主机相关信息

```
export password='qwe'
export your_ip=$(ip ad|grep inet|grep -v inet6|grep -v 127.0.0.1|awk '{print $2}'|cut -d/ -f1)
export your_hosts=$(cat /etc/hosts |grep $(echo $your_ip)|awk '{print $2}')
```

##### 2、安装mysql

```
echo "mysql-server-5.5 mysql-server/root_password password $password" | debconf-set-selections
echo "mysql-server-5.5 mysql-server/root_password_again password $password" | debconf-set-selections

apt-get -y install mariadb-server python-pymysql --force-yes

echo "[mysqld]
bind-address = $your_ip
default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8" | tee > /etc/mysql/conf.d/openstack.cnf

sed -i "s/127.0.0.1/0.0.0.0/g" /etc/mysql/mariadb.conf.d/50-server.cnf

service mysql restart
```

##### 3、创建hive用户和赋予权限

```
mysql -uroot -p$password <<EOF 
CREATE DATABASE hive;
CREATE USER 'hive' IDENTIFIED BY "$password";
GRANT ALL PRIVILEGES ON  *.* TO 'hive'@'%' WITH GRANT OPTION;
flush privileges;
EOF
```

##### 4、增加hive环境变量

```
hive_flag=$(grep "hive" /etc/profile)
if [ ! -n "$hive_flag" ]; then
    sed -i "s/\$PATH:/\$PATH:\/opt\/apache-hive-2.3.2-bin\/bin:/g" /etc/profile
else
    echo "Already exist!"
fi
```

##### 5、使脚本中环境变量生效

```
source /etc/profile
```

##### 6、修改hive配置

```
echo "$(grep "JAVA_HOME=" /etc/profile)
$(grep "HADOOP_HOME=" /etc/profile)
export HIVE_HOME=/opt/apache-hive-2.3.2-bin
export HIVE_CONF_DIR=/opt/apache-hive-2.3.2-bin/conf" |tee >> /opt/apache-hive-2.3.2-bin/conf/hive-env.sh

sed -i "s/hadoop3/$your_hosts/g" /opt/apache-hive-2.3.2-bin/conf/hive-site.xml
```

##### 7、在hdfs 中创建下面的目录 ，并赋予所有权限

```
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -mkdir -p /user/hive/tmp
hdfs dfs -mkdir -p /user/hive/log
hdfs dfs -chmod -R 777 /user/hive/warehouse
hdfs dfs -chmod -R 777 /user/hive/tmp
hdfs dfs -chmod -R 777 /user/hive/log

mkdir -p /user/hive/tmp
```

##### 8、初始化hive 

```
schematool -dbType mysql -initSchema
```

## 三、使用

##### 1、创建hive表

```
create table film
(name string,
time string,
score string,
id int,
time1 string,
score1 string,
name2 string,
score2 string)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ';'
STORED AS TEXTFILE;
```

##### 2、将本地文本导入hive

```
load data local inpath '/root/my.txt' overwrite into table film; 
```

##### 3、创建表movie

```
create table patition_table(name string,salary float,gender string)  partitioned by (dt string,dep string)  row format delimited fields terminated by ',' stored as textfile; 

create database movie;

create table movie(name string,data string,record int);
```

##### 4、删除表

```
DROP TABLE if exists movies;
```

##### 5、创建表

```
CREATE TABLE movies(
    name string,
    data string,
    record int
) COMMENT '2014全年上映电影的数据记录' FIELDS TERMINATED BY '\t' STORED AS TEXTFILE;

load data local inpath 'dat0204.log' into table movies;
```

##### 6、hive 中使用dfs命令

```
hive> dfs -ls /user/hive/warehouse/wyp ;

select * from movies;

hive -e "select * from test" >> res.csv  

或者是：  

hive -f sql.q >> res.csv  

其中文件sql.q写入你想要执行的查询语句  
```

##### 7、导出到本地文件系统

```
hive> insert overwrite local directory '/home/wyp/wyp'
hive> select * from wyp;
```

##### 8、导出到HDFS中

> 和导入数据到本地文件系统一样的简单，可以用下面的语句实现：

```
hive> insert overwrite directory '/home/wyp/hdfs'
hive> select * from wyp;
```

> 将会在HDFS的/home/wyp/hdfs目录下保存导出来的数据。注意，和导出文件到本地文件系统的HQL少一个local，数据的存放路径就不一样了。

##### 9、将提取到的数据保存到临时表中

```
insert overwrite table movies

本地加载  load data local inpath '/Users/tifa/Desktop/1.txt' into table test;

从hdfs上加载数据  load data inpath '/user/hadoop/1.txt' into table test_external;  
 抹掉之前的数据重写  load data inpath '/user/hadoop/1.txt' overwrite into table test_external; 
```

