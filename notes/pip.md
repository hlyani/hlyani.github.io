# pip 源相关


# 一、清华源

[tsinghua](https://mirrors.tuna.tsinghua.edu.cn/)

## 1、临时使用

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```
## 3、更新pip
```
pip install -U pip -i https://pypi.tuna.tsinghua.edu.cn/simple
```
## 4、设置默认

```
pip install pip -U -i https://pypi.tuna.tsinghua.edu.cn/simple
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

## 5、写入配置文件

```
mkdir -p ~/.pip/
tee > ~/.pip/pip.conf <<EOF
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple

[install]
trusted-host=pypi.tuna.tsinghua.edu.cn
EOF
```

# 二、阿里源

[aliyun](https://developer.aliyun.com/mirror/)

## 1、临时使用

```
pip install -i https://mirrors.aliyun.com/pypi/simple some-package
```

## 3、更新pip

```
pip install -U pip -i https://mirrors.aliyun.com/pypi/simple
```

## 4、设置默认

```
pip install pip -U -i https://mirrors.aliyun.com/pypi/simple
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple
```

## 5、写入配置文件

```
mkdir -p ~/.pip/
tee > ~/.pip/pip.conf <<EOF
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
EOF
```

# 三、中科大

[ustc](http://mirrors.ustc.edu.cn/)

```
pip install -i https://mirrors.ustc.edu.cn/pypi/web/simple package
```

