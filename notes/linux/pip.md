# pip 源相关


## 一、清华源

##### 1、临时使用
```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```
##### 3、设置默认
```
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```
##### 4、更新pip
```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
```
## 一、阿里源
```
~/.pip/pip.conf
中添加或修改:

[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```