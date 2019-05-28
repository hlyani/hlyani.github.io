# ftp 相关

##### 1、安装ftp
```
apt-get update
apt-get install vsftpd
service vsftpd restart
```
##### 2、新建"/home/uftp"目录作为用户主目录
```
mkdir /home/uftp
```
##### 3、新建用户uftp并设置密码
```
useradd -d /home/uftp -s /bin/bash uftp
passwd uftp
```
##### 4、编辑配置文件（/etc/vsftpd.conf）
```/etc/vsftpd.conf
添加:
userlist_deny=NO   //指定一个userlist，放允许ftp登陆的本地用户
userlist_enable=YES
userlist_file=/etc/allowed_users  //记录允许本地登陆用户名的文件
eccomp_sandbox=NO

改：
local_enable=YES
write_enable=YES
```
##### 5、编辑其他文件
```
vim /etc/allowed_users
加入：uftp

vim /etc/ftpusers   //记录不能访问FTP服务器的用户清单
删uftp
```
##### 6、重启服务
```
service vsftpd restart
```
###### 7、其他问题
```
* 用户名：uftp 密码：  端口21

* 553 Could not create file. 

setsebool -P ftpd_disable_trans 1
service vsftpd restart
```