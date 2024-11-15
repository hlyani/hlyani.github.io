# Conda

# 一、下载

https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/

```
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

# 二、安装

```
chmod +x Miniconda3-latest-Linux-x86_64.sh
./Miniconda3-latest-Linux-x86_64.sh
```

# 三 、初始化

```
./bin/conda init
```

# 四、配置镜像源

```
conda config --show channels
conda config --show
conda config --set 配置项=值
conda config --remove-key 配置项
```

```
conda config --remove channels https://xxxxxxxxxxxxxxx
```

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
```

```
conda config --set show_channel_urls yes
```

# 五、使用

## 1.创建环境

```
conda create --name env310 python=3.10 numpy matplotlib
```

```
conda info --env
conda env list
```

```
conda create --name new_env_name --clone old_env_name
```

## 2.删除环境

```
conda remove --name env310 --all
```

```
#删除所有不再需要的文件和缓存
conda clean --all
```

## 3.激活/退出环境

```
conda activate env310
```

```
conda deactivate
```

## 4.安装/卸载库

```
conda list
conda install scipy
conda remove scipy

conda update scipy
conda search scipy
```

# 六、其他

```
#关闭自动激活状态
conda config --set auto_activate_base false

#开启自动激活状态
conda config --set auto_activate_base true
```

```
卸载 conda
#1.直接删除安装目录
rm  -rf  /data/weisx/conda

#2.撤消对shell初始化脚本的更改
conda init --reverse --all

#3.删除可能已在主目录中创建的以下隐藏文件和文件夹
rm -rf ~/.condarc ~/.conda ~/.continuum
```

