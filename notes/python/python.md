# Python 相关



# 一、安装

```
wget https://www.python.org/ftp/python/3.13.0/Python-3.13.0.tgz
./configure --enable-optimizations --with-ensurepip=install --with-openssl=/usr/local/openssl-1.1.1 --with-openssl-rpath=auto
make altinstall -j 20
```

# 二、查看 gpu 是否可用

```
python -c  "import torch; print(torch.cuda.is_available()); print(torch.cuda.device_count())"
python -c "import torch.distributed as dist; print(dist.is_nccl_available())"
```







# 其他

```
for i, v in enumerate(['tic', 'tac', 'toe']):

for q, a in zip(questions, answers):

>>> for x in range(1, 11):
...     print(repr(x).rjust(2), repr(x*x).rjust(3), end=' ')
...     # 注意前一行 'end' 的使用
...     print(repr(x*x*x).rjust(4))

for x in range(1, 11):
...     print('{0:2d} {1:3d} {2:4d}'.format(x, x*x, x*x*x))

```