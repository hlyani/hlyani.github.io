# Hbase 相关

### 一、介绍

#### 与传统数据库的对比

##### 1、传统数据库遇到的问题：

 1）数据量很大的时候无法存储；
 2）没有很好的备份机制；
 3）数据达到一定数量开始缓慢，很大的话基本无法支撑；

##### 2、HBASE优势：

 1）线性扩展，随着数据量增多可以通过节点扩展进行支撑；
 2）数据存储在hdfs上，备份机制健全；
 3）通过zookeeper协调查找数据，访问速度快。

##### 3、HBase中的表一般有这样的特点：

​	1、大：一个表可以有上亿行，上百万列

​	2、面向列:面向列(族)的存储和权限控制，列(族)独立检索。

​	3、稀疏:对于为空(null)的列，并不占用存储空间，因此，表可以设计的非常稀疏。

### 二、hbase使用

[hbase shell 常用命令](http://blog.csdn.net/scutshuxue/article/details/6988348)

```
hbase
```

##### 1、创建表

```
create 'users','user_id','address','info'
```

> 说明:表users,有三个列族user_id,address,info

##### 2、列出全部表

```
list  
```

##### 3、得到表的描述

```
describe 'users'  
```

##### 4、创建表

```
create 'users_tmp','user_id','address','info'  
```

##### 5、删除表

```
disable 'users_tmp'  
drop 'users_tmp'  
```

##### 6、添加记录

> put ‘表名’,’行键(标识)’,’列族:字段’,’数值’

```
put 'users','yani','info:age','24';  

put 'users','yani','info:birthday','1987-06-17';  
```

##### 7、获取一条记录

1.取得一个id的所有数据

```
get 'users','yani'  
```


2.获取一个id，一个列族的所有数据
```
get 'users','yani','info'  
```

3.获取一个id，一个列族中一个列的所有数据

```
get 'users','yani','info:age'  
```

##### 8、更新记录

```
put 'users','yani','info:age' ,'29'  

get 'users','yani','info:age'  
```

##### 9、获取单元格数据的版本数据

```
get 'users','yani',{COLUMN=>'info:age',VERSIONS=>1}  
```

##### 10、获取单元格数据的某个版本数据

```
get 'users','yani',{COLUMN=>'info:age',TIMESTAMP=>1364874937056}  
```

##### 11、全表扫描

```
scan 'users'  
```

##### 12、删除yani值的'info:age'字段

```
delete 'users','yani','info:age'  

get 'users','yani'  
```

##### 13、删除整行

```
deleteall 'users','yani'
```

##### 14、统计表的行数

```
count 'users'  
```

##### 15、清空表

```
truncate 'users'  
```

