# mysql 相关

##### 1、允许远程访问
```
use mysql;
update db set host = '%' where user = 'root'; 
flush privileges;
grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;

#修改ip
vim /etc/mysql/my.cnf

#重启服务
service mysql restart
```
##### 2、修改密码
```
1、连接数据库
2、use mysql:
3、update user set password=password("0127") where user="root";
4、flush privileges;

忘记roo密码

1、关闭正在运行的MySQL服务。
2、打开DOS窗口，转到mysql\bin目录。
3、 输入mysqld --skip-grant-tables 回车。--skip-grant-tables 的意思是启动MySQL服务的时候跳过权限表认证。
4、再开一个DOS窗口（因为刚才那个DOS窗口已经不能动了），输入mysql回车，如果成功，将出现MySQL提示符 >。 
6、连接权限数据库： use mysql; 。 
7、改密码：update user set password=password("root") where user="root";（别忘了最后加分号） 。 
8、刷新权限（必须步骤）：flush privileges;　。 
9、退出  quit。
```

##### 3、使用docker创建mysql服务

```
docker run -itd --name mariadb --restart=always -v /opt/mysql:/etc/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mariadb

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY '123456' WITH GRANT OPTION;
flush privileges;

mysql -h 127.0.0.1 -u root -p123456
```

