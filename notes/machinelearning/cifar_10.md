# cifar-10 相关

> 该数据集共有60000张彩色图像，这些图像是32*32，分为10个类，每类60000张图。这里面有50000张用于训练，构成了5个训练批，每一批10000张图；另外10000用于测试，单独构成一批。测试批的数据里，取自10类中的每一类，每一类随机取1000张。抽剩下的就随机排列组成了训练批。注意一个训练批中的各类图像并不一定数量相同，总的来看训练批，每一类都有5000张图。

[cifa-10简介](https://www.cnblogs.com/Jerry-Dong/p/8109938.html)
[cifa-10官网](http://www.cs.toronto.edu/~kriz/cifar.html)
[keras离线下载cifar数据集](https://blog.csdn.net/nima1994/article/details/79910597)

##### 1、下载cifar-10
```
wget https://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz

通过源码keras\utils\data_utils.py可以看到下载后的文件保存至~/.keras/datasets/"fname".tar.gz。

可以借助第三方工具下载数据集，然后（对于Windows）移动到C:\Users\你的用户名\.keras\datasets目录下，并改名（包括后缀名）为cifar-10-batches-py.tar.gz。

```
##### 2、相关代码
```
from keras.datasets import cifar10

(x_train, y_train), (x_test, y_test) = cifar10.load_data()

print(x_train[:4])
print(y_train[:4])
print(x_test[:4])
print(y_test[:4])

#coding:utf-8
import pickle
import numpy as np
import os

CIFAR_DIR = "/root/.keras/datasets/cifar-10-batches-py"
#print(os.listdir(CIFAR_DIR))

with open(os.path.join(CIFAR_DIR,"data_batch_1"),'rb') as f:
    data = pickle.load(f,encoding='latin1')
#    print(type(data))
#    print(dir(data))
#    print(data.keys())
#    print(type(data['data']))
#    print(type(data['batch_label']))
#    print(type(data['labels']))
#    print(type(data['filenames'])
# 32 * 32 = 1024 * 3 = 3072
#    print(data['data'].shape)
#    print(data['data'][0:2])
#    print(data['labels'][0:2])
#    print(data['batch_label'])
#    print(data['filenames'][0:2])


image_arr = data['data'][1]
image_arr = image_arr.reshape((3,32,32))
image_arr = image_arr.transpose((1,2,0))

# 使数据可视化
import matplotlib.pyplot as plt
from matplotlib.pyplot import imshow

plt.imshow(image_arr)
plt.legend()
plt.show()
```