# pip 本地源

# 1、安装 pip2pi 工具

```
pip install pip2pi
```

# 2、建立索引，会创建 simple 文件夹

```
dir2pi path
```

# 3、更新索引

```
多个包: pip2acmeco -r requirements.txt 
单个包: pip2acmeco package==1.0.0
```

# 4、同步软件包

```
1、创建目录
mkdir  /work/pypi/Packages/
2、同步单个软件包
pip2tgz /work/pypi/Packages requests
3、批量同步
pip2tgz /work/pypi/Packages -r ./requirements.txt
4、创建索引
dir2pi /work/pypi/Packages/
```

