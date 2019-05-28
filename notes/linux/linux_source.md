# Linux 源制作

## 一、apt 下载软件做离线源

##### 1、拷贝所需安装软件包

```
通过apt-get安装的软件都在/var/cache/apt/archives目录下
cp /var/cache/apt/archives/* /home/package
```

##### 2、生成软件包信息（含有重要的包的依赖关系）

```
apt-get install dpkg-dev

dpkg-scanpackages package  /dev/null  | gzip > packs/Packages.gz

dpkg-scanpackages pools override > dists/trusty/main/binary-i386/Packages

dpkg-scanpackages pools override > dists/trusty/main/binary-amd64/Packages  

注：/dev/null位置的参数是指定一个文件，文件名不限，该文件的作用是用来重写覆盖deb软件包中控制文件的某些定义，它的第一行的格式，一行对应一个软件包：

package priority section
package指定你所要修改的软件包
priority 有low,medium,high三个值
section 用来指定软件包属于哪个section

如果不需要对deb软件包做任何修改你就可以像例子中那样直接指定一个/dev/null文件。
```

##### 3、添加本地源

```
apt命令每次都会读取/etc/apt/sources.list源列表(这个源列表可以添加好多源,每次都选中开头的有效源)，因此我们编辑该文件，在第一行添加我们自己的本地源，如：

deb http://172.18.20.161/ packs/

# deb file:///home packs/

要注意中间的空格
```

##### 4、打包本地源

```
将/etc/apt/sources.list文件拷贝到packages目录下，将packages文件夹打包、备份，以便使用。
```

##### 5、如何使用本地源

```
将packages压缩包放到/目录(该目录只要和添加的本地源路径一致即可，以便apt能找到源）下解压，备份本机的sources.list，将packages目录下的sources.list拷贝到/etc/apt/目录下。修改/etc/apt/sources.list 之后一般会运行下面两个命令进行更新升级：
sudo apt-get update
sudo apt-get dist-upgrade
其中 ：
update - 取回更新的软件包列表信息
dist-upgrade - 发布版升级

然后就可以离线安装了：apt-get install xxxx

## deb file:///opt/chuandge /packs/
```

## 二、使用 apt-mirror 建立本地 ubuntu 源

##### 1、安装apt-mirror

```
apt-get install apt-mirror
```

##### 2、创建放镜像的文件夹

```
例如将镜像等文件放在 /service/Ubuntu文件夹下：
并新建以下文件夹（mirror.list里面提示要事先新建以下文件夹的）：
/service/ubuntu
/service/ubuntu/mirror
/service/ubuntu/skel
/service/ubuntu/var
```

##### 3、配置apt-mirror：

```
vim /etc/apt/mirror.list

############# config ##################
#
# set base_path /var/spool/apt-mirror
#
# if you change the base path you must create the directories below with write privlages
#
# set mirror_path  $base_path/mirror
# set skel_path $base_path/skel
# set var_path    $base_path/var
# set cleanscript $var_path/clean.sh
# set defaultarch  <running host architecture>

#我们添加或更改如下内容：
set base_path /service/ubuntu

set mirror_path  $base_path/mirror
set skel_path $base_path/skel
set var_path    $base_path/var
set cleanscript $var_path/clean.sh
set nthreads    20
set _tilde 0
#
############# end config ##############

#我们把常用的软件同步过来就够用了
deb-i386 http://archive.Ubuntu.com/ubuntu hardy main restricted universe multiverse
deb-i386 http://archive.ubuntu.com/ubuntu hardy-updates main restricted universe multiverse
deb-i386 http://archive.ubuntu.com/ubuntu hardy-backports main restricted universe multiverse
deb-i386 http://archive.ubuntu.com/ubuntu hardy-security main restricted universe multiverse
deb-i386 http://archive.ubuntu.com/ubuntu hardy-proposed main restricted universe multiverse

#当某些软件包在服务器端进行了升级，或者服务器端不再需要这些软件包时，我们使用了 apt-mirror与服务器同步后，会在本地的$var_path/下生成一个clean.sh的脚本，列出了遗留在本地的旧版本和无用的软件包，你可 以手动运行这个脚本来删除遗留在本地的且不需要用的软件包
clean http://archive.ubuntu.com/ubuntu
```

```
如果用amd64位架构下的包，可以加上deb-amd64的标记
如果什么都不加，直接使用deb http.....这种格式，则在同步时，只同步当前系统所使用的架构下的软件包。比如一个64位系统，直接debhttp....只同步64位的软件 包。如果还嫌麻烦，直接去改set defaultarch  <running hostarchitecture> 这个参数就好，比如改成set defaultarch i386，这样就使用debhttp.....这种格式，则在同步时，只同步i386的软件包了。
如果你还想要源码，可以把源码也加到mirror.list里面同步过来，比如加上deb-src这样的标记。想要其他的东西也可以追加相应的标记来完成。
```

##### 4、配置好后指定的镜像进行同步

```
apt-mirror
```

> 如果是第一次同步，官方镜像可能需要几天时间才能同步完整，如果与国内源进行同步，只同步常用软件，平均1秒钟网速1MB（Byte）要同步30G左右的数据，大概需要5-8小时的时间才能同步完整。

##### 5、清理无用软件包，同步完成后，我们可以利用clean.sh清理无用软件包（本文档以set base_path /server/ubuntu为例）：

```
bash /service/ubuntu/var/clean.sh
```

##### 6、更新完毕后，使用apache发布源镜像了。

###### 1.配置apache2

```
vim httpd.conf

#<Directory />
#    AllowOverride none
#    Require all denied
#</Directory>

#改成
<Directory />
    Options FollowSymLinks
    AllowOverride None
    Order deny,allow
    allow from all
</Directory>

#在<IfModule alias_module> 节点中增加虚拟目录
<IfModule alias_module>
    Alias /ubuntu /workspace/ubuntu/mirror/mirrors.163.com/ubuntu
</IfModule>
#添加虚拟目录的权限
<Directory "/workspace/ubuntu/mirror/mirrors.163.com/ubuntu">
      Options Indexes FollowSymLinks
      AllowOverride None
        Require all granted
</Directory>

#由于有opengrok 做了个tomcat的端口转发
ProxyPass /source http://localhost:8081/source
ProxyPassReverse /source http://localhost:8081/source
```

###### 2.重启apache

```
service apache2 restart
```

###### 3.客户端配置source.list

```
vim /etc/source.list

deb [arch=amd64] http://192.168.19.184/ubuntu/ trusty main restricted
deb [arch=amd64] http://192.168.19.184/ubuntu/ trusty-updates main restricted
deb [arch=amd64] http://192.168.19.184/ubuntu/ trusty universe
deb [arch=amd64] http://192.168.19.184/ubuntu/ trusty-updates universe
deb [arch=amd64] http://192.168.19.184/ubuntu/ trusty multiverse
deb [arch=amd64] http://192.168.19.184/ubuntu/ trusty-updates multiverse
deb [arch=amd64] http://192.168.19.184/ubuntu/ trusty-backports main restricted universe multiverse
deb [arch=amd64] http://192.168.19.184/ubuntu/ trusty-security main restricted
deb [arch=amd64] http://192.168.19.184/ubuntu/ trusty-security universe
deb [arch=amd64] http://192.168.19.184/ubuntu/ trusty-security multiverse
```

###### 4.更新源

```
apt-get update
```

