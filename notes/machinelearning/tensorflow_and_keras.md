# tensorflow 和 keras 相关

## 一、tensorflow 相关
> TensorFlow 是一个编程系统, 使用图 (graph) 来表示计算任务。
> 在被称之为 会话 (Session) 的上下文 (context) 中执行图。
> 使用 tensor 表示数据。
> 通过 变量 (Variable) 维护状态。
> 使用 feed 和 fetch 可以为任意的操作(arbitrary operation) 赋值或者从其中获取数据。

```
$ python

>>> import tensorflow as tf
>>> hello = tf.constant('Hello, TensorFlow!')
>>> sess = tf.Session()
>>> print(sess.run(hello))
Hello, TensorFlow!
>>> a = tf.constant(10)
>>> b = tf.constant(32)
>>> print(sess.run(a+b))
42

import tensorflow as tf
# 创建一个变量, 初始化为标量 0.
state = tf.Variable(0, name="counter")

# 创建一个 op, 其作用是使 state 增加 1

one = tf.constant(1)
new_value = tf.add(state, one)
update = tf.assign(state, new_value)

# 启动图后, 变量必须先经过`初始化` (init) op 初始化,
# 首先必须增加一个`初始化` op 到图中.
init_op = tf.initialize_all_variables()

# 启动图, 运行 op
with tf.Session() as sess:
  # 运行 'init' op
  sess.run(init_op)
  # 打印 'state' 的初始值
  print(sess.run(state))
  # 运行 op, 更新 'state', 并打印 'state'
  for _ in range(3):
    sess.run(update)
    print(sess.run(state))
```

##### 1、安装 tensorflow
```
pip install tensorflow

pip install tensorflow # Python 2.7; CPU support (no GPU support) 
pip install tensorflow-gpu # Python 2.7; GPU support 

[win10安装tensorflow]
https://www.jianshu.com/p/1fad663dabc3
https://zhuanlan.zhihu.com/p/35717544
```
##### 2、测试是否安装成功
```
python -c "import tensorflow as tf; tf.enable_eager_execution(); print(tf.reduce_sum(tf.random_normal([1000, 1000])))"
```
##### 3、测试是否支持GPU
```
python -c "import tensorflow as tf; tf.Session(config=tf.ConfigProto(log_device_placement=True))"
```
##### 4、相关博客
> tensorflow图片识别

[https://blog.csdn.net/gaohuazhao/article/details/72886450]
[https://blog.csdn.net/qq_39521554/article/details/79335733]
[https://blog.csdn.net/ShadowN1ght/article/details/78076081]
[https://www.cnblogs.com/seven-M/p/8516080.html]

> tensorflow 人脸识别

[https://blog.csdn.net/qq_42633819/article/details/81191308]
[https://blog.csdn.net/niutianzhuang/article/details/79191167]
[https://blog.csdn.net/justin_kang/article/details/79627951]

## 二、keras 相关
##### 1、安装keras
```
pip install keras
```
##### 2、相关博客
[keras官网]: https://keras.io/zh/

[face recognise]:http://www.feiguyunai.com/index.php/2017/11/25/pythonai-dl-facerecong01/

[初学者怎样使用Keras进行迁移学习]: https://www.toutiao.com/a6648764315258061315/?tt_from=weixin&utm_campaign=client_share&wxshare_count=1&timestamp=1552000777&app=news_article&utm_source=weixin&iid=65083704169&utm_medium=toutiao_ios&group_id=6648764315258061315
## 三、tensorboard 相关 
##### 1、tensorboard相关
[tensorboard]: https://www.cnblogs.com/tengge/p/6376073.html
```
tensorboard
执行上述代码，会在“当前路径/logs”目录下生成一个events.out.tfevents.{time}.{machine-name}的文件。在当前目录新建“查看训练过程.bat”，里面输入。

tensorboard --logdir=K:\AI\content\test\hl

python /home/bids/.local/lib/python2.7/site-packages/tensorboard/tensorboard.py --logdir='/tmp/keras_log'
```
## 四、其他
##### 1、其他常用库安装
```
pip install matplotlib numpy
```
## 五、TensorFlow GPU训练环境搭建
##### 1.安装NVIDIA显卡驱动

1. 禁用nouveau，在终端（Ctrl-Alt+T）输入：

```
sudo gedit /etc/modprobe.d/blacklist.conf
```

在最后一行添加：

```
blacklist nouveau
```

保存退出，在终端（Ctrl-Alt+T）执行命令：

```
sudo update-initramfs -u
重启之后执行
lsmod | grep nouveau #没有输出则说明配置成功
```

2. 安装驱动，Ctrl-Alt+F1进入命令行界面之后输入用户名和密码登录 ，找到驱动文件NVIDIA_xxx.run所在目录（默认为当前用户目录的Downloads目录下）并赋予该文件可执行权限，然后进行安装：

```
cd Downloads
sudo chmod a+x NVIDIA_xxx.run
sudo service lightdm stop #关闭图形界面
sudo ./xxx.run -no-nouveau-check -no-opengl-files
```

完成后重启。

##### 2.安装NVIDIA CUDA Toolkit 9.0

1. Ctrl-Alt+F1进入命令行界面之后输入用户名和密码登录 ，找到CUDA9.0文件cuda_xxx.run所在目录（默认为当前用户目录的Downloads目录下）并赋予该文件可执行权限，然后进行安装：

```
cd Downloads
sudo chmod a+x cuda_xxx.run
sudo service lightdm stop #关闭图形界面
sudo ./xxx.run
```

注意：安装过程中当询问是否安装显卡驱动时选n，因为先前已安装完显卡驱动无需再进行安装。

2. 配置环境变量：

```
sudo service lightdm start #开启图形界面
```

登录系统，打开终端（Ctrl-Alt+T）：

```
sudo gedit /etc/profile
```

在最后添加：

```
export PATH=/usr/local/cuda-9.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

保存退出，立即生效：

```
source /etc/profile
```

完成后重启。

3. 验证CUDA，打开终端（Ctrl-Alt+T）输入：

```
nvcc -V
```

出现相应版本号信息则说明安装成功。

##### 3.安装NVIDIA cuDNN 7.4

1.打开终端（Ctrl-Alt+T），找到cuDNN文件libcudnn7_xxx.deb、libcudnn7-dev_xxx.deb、libcudnn7-doc_xxx.deb所在目录（默认为当前用户目录的Downloads目录下），进行安装：

```
sudo dpkg -i libcudnn7_xxx.deb
sudo dpkg -i libcudnn7-dev_xxx.deb
sudo dpkg -i libcudnn7-doc_xxx.deb
```

若不报错则说明安装成功。

2. 验证cuDNN是否已安装并可以正常运行，复制cuDNN sample到当前用户目录下：

```
cp -r /usr/src/cudnn_samples_v7/ $HOME
```

进入cuDNN相应测试样本的路径：

```
cd $HOME/cudnn_samples_v7/mnistCUDNN
```

编译该测试样本：

```
make clean && make
```

运行该测试样本：

```
./mnistCUDNN
```

若cuDNN安装并可正常运行则会出现：

```
Test passed!
```

