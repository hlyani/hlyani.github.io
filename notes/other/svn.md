# svn 相关


## 一、ubuntu安装和配置svn
##### 1. 安装svn
```
apt-get install subversion
```
##### 2. 建立svn仓库
```
1). 建立svn目录：mkdir /home/.svn(使用隐藏目录）
2). cd /home/.svn
3). mkdir astar
4). 创建仓库astar：svnadmin create /home/.svn/astar，执行完毕后astar目录有svnadmin创建的目录和文件
5). mkdir test
6). 创建仓库test：svnadmin create /home/.svn/test，执行完毕后test目录有svnadmin创建的目录和文件
```
##### 3. 配置和管理svn
```
1). 每个仓库的配置文件在$repos/conf/下，vi svnserve.conf，配置项在[general]下：
anon-access：匿名用户的权限，可以为read，write和none，默认值read。不允许匿名用户访问：anon-access = none
auth-access：认证用户的权限，可以为read，write和none，默认值write。
password-db：密码数据库的路径，去掉前边的#
authz-db：认证规则库的路径，去掉前边的#。
注意：这些配置项的行都要顶格，否则会报错。修改配置后需要重启svn才能生效。
2). 配置passwd文件
这是每个用户的密码文件，比较简单，就是“用户名=密码”，采用的是明码。如allen=111111
3). 配置authz文件
1. [groups] section：为了便于管理，可以将一些用户放到一个组里边，比如：owner=allen,ellen
2. groups下边的sections表示对一个目录的认证规则，比如对根目录的认证规则的section为[/]。设置单用户的认证规则时一个用户一行，如：
[/]
allen=rw　　#allen对根目录的权限为rw
ellen=r　　  #ellen对根目录的权限为r
如果使用group，需要在group名字前加@,如
@owner=rw　　#group owner中的用户均为rw，等价于上边的两句话
启动时如果从/home/.svn/astar启动，/就是astar目录，用如上方式以astar目录为根设置权限。
如果从/home/.svn/启动，每个仓库根还是自己的起始目录。可以采用如上方式设置astar的权限，也可以采用如下方式：
[astar:/]
@owner=rw
设置test的权限如下：
[test:/]
@harry_and_sally = rw
简言之，每个仓库的根目录(/)就是自己的起始目录；[repos:/]这种方式只适用于多仓库的情况；[/]适合于单仓库和单仓库的方式。
3. 不能跨越仓库设置权限。
```
##### 4. 启动和停止svn
```
1). 启动（如：svnserve --listen-port 9999 -d -r /home/.svn）：
1. 从astar目录启动，svnserve -d -r /home/.svn/astar，根目录(/)是astar，authz中规则的配置使用section[/]。访问方式为：
svn://192.168.0.87/
2. 从.svn目录启动，svnserve -d -r /home/.svn，根目录(/)是.svn，authz中对astar的配置使用section[astar:/] ,对test的配置使用section[test:/]。访问方式为：
svn://192.18.0.87/astar
svn://192.18.0.87/test
如果需要svn自启动，把命令加入/etc/rc.local中
2). 检查svn服务器是否已经启动（svn默认使用3690端口）：netstat -an | grep 3690
3). 停止：killall svnserve
```
##### 5、创建项目

```
svnadmin create /var/opt/svn/TEST

svn://192.168.0.127:3690/TEST

vim conf/authz
[groups]
owner = hl

[/]
hl=rw
@owner=rw

[TEST:/]
hl=rw
@owner=rw

vim conf/password
[users]
hl = hl

vim conf/svnserve.conf
anon-access = read:wq
auth-access = write
password-db = passwd
authz-db = authz
realm = /var/opt/svn/TEST
```

##### 6. svn client 连接