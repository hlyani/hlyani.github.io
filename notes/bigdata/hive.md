# Hive 相关

## 一、hive相关资料

[http://blog.csdn.net/u013310025/article/details/70306421](http://blog.csdn.net/u013310025/article/details/70306421)
[https://www.cnblogs.com/guanhao/p/5641675.html](https://www.cnblogs.com/guanhao/p/5641675.html)
[http://blog.csdn.net/wisgood/article/details/40560799](http://blog.csdn.net/wisgood/article/details/40560799)
[http://blog.csdn.net/seven_zhao/article/details/46520229](http://blog.csdn.net/seven_zhao/article/details/46520229)

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

```
hive
```

##### 1、查看数据库，使用数据库

```
show databases;
use default;
```

##### 2、查看表

```
show tables;
```

##### 3、创建hive表

> ";" ASCI码为 "\073"
>
> ROW FORMAT DELIMITED 每条数据按行拆分
>
> FIELDS TERMINATED BY '\073' 每行数据字段按 ";"  拆分
>
> STORED AS TEXTFILE 保存为文本文件

```
create table film
(name string,
startTime date,
endTime date,
company string,
director string,
actor string,
type string,
price float,
score float)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\073'
STORED AS TEXTFILE;
```

##### 4、将本地文本导入 hive

> 先在 shell 中生成文本文件
>
> 以下是完整一点儿的电影信息
>
>  https://github.com/hlyani/demo/blob/master/film.csv 

```
echo "电影名称;上映时间;闭映时间;出品公司;导演;主角;影片类型;票房/万;评分;
《熊出没之夺宝熊兵》;2014.1.17;2014.2.23;深圳华强数字动漫有限公司;丁亮;熊大，熊二，;
《菜鸟》;2015.3.27;2015.4.12;麒麟影业公司;项华祥;柯有伦，崔允素，张艾青，刘亚鹏，张星宇;爱情/动作/喜剧;192.0 ;4.5 ;
《栀子花开》;2015.7.10;2015.8.23;世纪百年影业，文投基金，华谊兄弟，千和影业，剧角映画，合一影业等;何炅;李易峰，张慧雯，蒋劲夫，张予曦，魏大勋，李心艾，杜天皓，宋轶，王佑硕，柴格，张云龙;青春，校园，爱情;37900.8 ;4.0 ;
《我是大明星》;2015.12.20;2016.1.31;北京中艺博悦传媒;张艺飞;高天，刘波，谭皓，龙梅子;爱情 励志 喜剧;9.8 ;2.5 ;
《天将雄师》;2015.2.19;2015.4.6;耀莱文化，华谊兄弟，上海电影集团;李仁港;成龙，约翰·库萨克，阿德里安·布劳迪，崔始源 ，林鹏，王若心，筷子兄弟，西蒙子，冯绍峰，朱佳煜;动作，古装，剧情，历史;74430.2 ;5.9 ;" > /root/film.csv
```

> overwrite 覆盖之前数据

```
load data local inpath '/root/film.csv' overwrite into table film; 
```

##### 5、将hdfs中数据导入hive

```
hdfs dfs -put /root/film.csv /home
hdfs dfs -cat /home/film.csv
load data inpath '/home/film.csv'into table film;
```

##### 6、从 hive 导出到本地文件系统

```
insert overwrite local directory '/root/hl' select * from film;
```

> 或直接在shell中执行

```
hive -e "select * from film" >> film1.csv  
```

> 或者是以下命令， 其中文件sql.q写入你想要执行的查询语句

```
hive -f sql.q >> film1.csv  
```

##### 7、从 hive 导出到 hdfs 中

> 注意，和导出文件到本地文件系统的HQL少一个local，数据的存放路径就不一样了。

```
insert overwrite directory '/root/hl' select * from film;
```

##### 8、hive 中使用dfs命令

```
hive> dfs -ls /user;
```

##### 9、查看表结构信息

- 格式化查看表结构

```
desc formatted film;
```

- 查看表详细信息

```
desc film;
```

##### 10、查看表信息

```
select * from film;

# 查看前十项
select * from film limit 10;

select count(price) from film;
# distinct 去重
select count(distinct  price) from film;
select sum(price) from film;
select avg(price) from film;
```

##### 11、将提取到的数据保存到临时表中

```
insert overwrite table movies

本地加载  load data local inpath '/Users/tifa/Desktop/1.txt' into table test;

从hdfs上加载数据  load data inpath '/user/hadoop/1.txt' into table test_external;  
 抹掉之前的数据重写  load data inpath '/user/hadoop/1.txt' overwrite into table test_external; 
```

```
CREATE TABLE movies(
    name string,
    data string,
    record int
) COMMENT '2014全年上映电影的数据记录' FIELDS TERMINATED BY '\t' STORED AS TEXTFILE;

load data local inpath 'dat0204.log' into table movies;
```

##### 12、删除表

```
DROP TABLE if exists film;
```
