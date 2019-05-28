# 删除cinder僵尸实例

##### 1、使环境变量生效
```
source admin-openrc
```
##### 2、查看所有的磁盘
```
cinder list --all
```
##### 3、将磁盘的状态改为 available
```
cinder reset-state --state available <UUID>
```
##### 4、将磁盘状态改为 detached
```
cidner reset-state --attach-status detached <UUID>
```
##### 5、删除磁盘
```
cinder delete <UUID>
```