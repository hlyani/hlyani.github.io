# yum 安装时文件冲突

##### 1、下载安装包到指定文件夹

```
mkdir /tmp/packages

yum install --downloadonly --downloaddir /tmp/packages
```

##### 2、使用rpm 忽略冲突文件安装

> ```
> --replacefiles                   ignore file conflicts between packages
> ```

```
cd /tmp/packages

rpm -ivh * --replacefiles
```

##### 3、查看

```
rpm -qa
```

