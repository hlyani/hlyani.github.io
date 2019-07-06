# numpy 相关

[](http://www.runoob.com/numpy/numpy-tutorial.html)
[](https://blog.csdn.net/qq351469076/article/details/78817378)

## 一、安装

```
pip install numpy
import numpy as np
```

## 二、使用

##### 1、创建0-23, 共2个三行四列的数组
```
a = np.arange(24).reshape((2,3,4))

array([[[ 0,  1,  2,  3],
        [ 4,  5,  6,  7],
        [ 8,  9, 10, 11]],
   [[12, 13, 14, 15],
    [16, 17, 18, 19],
    [20, 21, 22, 23]]])
    
data = np.arange(28).reshape(7,4)	

import numpy as np
a=np.array([1,2,3])
print(a)
[1 2 3]
```

##### 2、多于一个维度  
```
import numpy as np 
a = np.array([[1,  2],  [3,  4]])  
print (a)
[[1 2]
 [3 4]]

import numpy as np 

a = np.arange(24)  
print (a.ndim)             # a 现只有一个维度
```

##### 3、现在调整其大小

```
#个数，行，列
b = a.reshape(2,4,3)  # b 现在拥有三个维度
print (b.ndim)

a = np.array([[1,2,3],[4,5,6]])  
print(a)
[[1 2 3]
 [4 5 6]]
print (a.shape)
#2列，3行
（2,3）
#维度（秩）
print (len(a.shape))
2

a = np.array([[1,2,3],[4,5,6]]) 
a.shape =  (3,2)  
print (a)
输出结果为：

[[1 2]
 [3 4]
 [5 6]]
```

##### 4、NumPy 也提供了 reshape 函数来调整数组大小

```
a = np.array([[1,2,3],[4,5,6]]) 
b = a.reshape(3,2)  
print (b)
输出结果为：

[[1, 2] 
 [3, 4] 
 [5, 6]]
```

##### 5、numpy.empty 方法用来创建一个指定形状（shape）、数据类型（dtype）且未初始化的数组：

```
 x = np.empty([3,2], dtype = int) 
print (x)
```

##### 6、numpy.zeros，创建指定大小的数组，数组元素以 0 来填充：

```
numpy.zeros(shape, dtype = float, order = 'C')

y = np.zeros((5,), dtype = np.int) 
print(y)
[0 0 0 0 0]
```

##### 7、numpy.ones，创建指定形状的数组，数组元素以 1 来填充：

```
numpy.ones(shape, dtype = None, order = 'C')
```

##### 8、将列表转换为 ndarray:

```
x =  [1,2,3] 
a = np.asarray(x)  
print (a)
[1 2 3] 
```

##### 9、将元组转换为 ndarray:

```
x =  (1,2,3) 
a = np.asarray(x)  
print (a)
```

##### 10、将元组列表转换为 ndarray:

```
x =  [(1,2,3),(4,5)] 
a = np.asarray(x)  
print (a)
[(1, 2, 3) (4, 5)]
```

##### 11、设置了 dtype 参数：

```
x =  [1,2,3] 
a = np.asarray(x, dtype =  float)  
print (a)
[1. 2. 3.]
```

##### 12、NumPy 从数值范围创建数组，numpy 包中的使用 arange 函数创建数值范围并返回 ndarray 对象，函数格式如下：

```
numpy.arange(start, stop, step, dtype)
x = np.arange(10,20,2)  
print (x)
[10 12 14 16 18]
```

##### 13、分片，索引

```
a = np.arange(10)
s = slice(2,7,2)   # 从索引 2 开始到索引 7 停止，间隔为2
print (a[s])
[2  4  6]

a = np.arange(10)  
b = a[2:7:2]   # 从索引 2 开始到索引 7 停止，间隔为 2
print(b)

a = np.arange(10)
print(a[2:])

a = np.array([[1,2,3],[3,4,5],[4,5,6]])  
print (a[...,1])   # 第2列元素
print (a[1,...])   # 第2行元素
print (a[...,1:])  # 第2列及剩下的所有元素

a[...,1：]
...省略行的索引，索引为列的索引，索引为1的行开始到最后

x = np.array([[1,  2],  [3,  4],  [5,  6]]) 
y = x[[0,1,2],  [0,1,0]]  
print (y)
[1  4  5]

x = np.array([[  0,  1,  2],[  3,  4,  5],[  6,  7,  8],[  9,  10,  11]])  
rows = np.array([[0,0],[3,3]]) 
cols = np.array([[0,2],[0,2]]) 
y = x[rows,cols]  
print (y)
[[ 0  2]
 [ 9 11]]

a = np.array([[1,2,3], [4,5,6],[7,8,9]])
b = a[1:3, 1:3]
c = a[1:3,[1,2]]
d = a[...,1:]
print(b)
print(c)
print(d)
输出结果为：

[[5 6]
 [8 9]]
[[5 6]
 [8 9]]
[[2 3]
 [5 6]
 [8 9]]

x = np.array([[  0,  1,  2],[  3,  4,  5],[  6,  7,  8],[  9,  10,  11]])  
print (x[x >  5])

大于 5 的元素是：
[ 6  7  8  9 10 11]
```

* add()	对两个数组的逐个字符串元素进行连接
* multiply()	返回按元素多重连接后的字符串
* center()	居中字符串
* capitalize()	将字符串第一个字母转换为大写
* title()	将字符串的每个单词的第一个字母转换为大写
* lower()	数组元素转换为小写
* upper()	数组元素转换为大写
* split()	指定分隔符对字符串进行分割，并返回数组列表
* splitlines()	返回元素中的行列表，以换行符分割
* strip()	移除元素开头或者结尾处的特定字符
* join()	通过指定分隔符来连接数组中的元素
* replace()	使用新字符串替换字符串中的所有子字符串
* decode()	数组元素依次调用str.decode
* encode()	数组元素依次调用str.encode