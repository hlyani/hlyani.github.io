# CentOS 修改默认启动内核

##### 1、查看当前内核
```
uname -r
```
##### 2、显示已经安装的内核
```
rpm -qa | grep kernel
```
##### 3、安装指定内核
```
rpm -ivh http://soft.91yun.org/ISO/Linux/CentOS/kernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force
```
##### 4、卸载内核
```
yum remove kernel-3.10.0-229.1.2.el7.x86_64.rpm
```
##### 5、启动内核修改
```
1.查看启动项 cat /boot/grub2/grub.cfg
/查看启动项 cat /boot/grub2/grubenv

2. 设置默认启动项 grub2-set-default "CentOS Linux (3.10.0-693.17.1.el7.x86_64) 7 (Core)"
3.10.0-693.17.1.el7.x86_64

3. 查看默认启动项 grub2-editenv list

4. 生成配置 grub2-mkconfig -o /boot/grub2/grub.cfg
#备注： 在生成grub.cfg之前，最好先备份原始的grub.cfg文件
```
###### 6、重新安装内核即可
```
yum -y update
```