# anaconda 相关


## 一、介绍

- conda是一种通用包管理系统，旨在构建和管理任何语言和任何类型的软件。举个例子：包管理与pip的使用类似，环境管理则允许用户方便地安装不同版本的python并可以快速切换。
- Anaconda则是一个打包的集合，里面预装好了conda、某个版本的python、众多packages、科学计算工具等等，就是把很多常用的不常用的库都已经装好了。
- Miniconda只包含最基本的内容——python与conda，以及相关的必须依赖项，对于空间要求严格的用户，Miniconda是一种选择。就只包含最基本的东西，其他的库得自己装。

##### 1、conda使用
https://www.jianshu.com/p/edaa744ea47d

##### 2、下载 anaconda
https://www.anaconda.com/distribution/#download-section

##### 3、miniconda安装地址
https://conda.io/en/latest/miniconda.html

## 二、 使用

#### 1、配置清华源
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/

conda config --set show_channel_urls yes
conda upgrade --all
```
##### 2、使用conda安装软件
```
conda install numpy
```
##### 3、创建python环境
```
conda create -n my_env python=3
```
##### 4、进入环境
```
在OSX/Linux上，使用
conda activate my_env

在Windows上，使用
activate my_env
```
##### 5、离开环境
```
在OSX/Linux上，使用
conda deactive

在windows上，使用
deactivate
```

##### 6、查看虚拟环境
```
conda env list
```

##### 7、查看安装了哪些包
```
conda list
```

##### 8、切换环境
```
activate my_env
```

##### 9、移除虚拟环境
```
conda env remove -n my_env
```