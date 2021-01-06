

# UnixBench 虚拟机性能测试工具

> `unixbench`是一个用于测试`unix`系统性能的工具，也是一个比较通用的`benchmark`， 此测试的目的是对类`Unix` 系统提供一个基本的性能指示，很多测试用于系统性能的不同方面，这些测试的结果是一个指数值（`index value`，如520），这个值是测试系统的测试结果与一个基线系统测试结果比较得到的指数值，这样比原始值更容易得到参考价值，测试集合里面所有的测试得到的指数值结合起来得到整个系统的指数值。
>
> 各项的测试有得分，然后有一个综合的得分，这样可以很方便的通过分数去比较。

# 1、下载

```
wget http://soft.laozuo.org/scripts/UnixBench5.1.3.tgz
# wget https://s3.amazonaws.com/cloudbench/software/UnixBench5.1.3.tgz
```

# 2、解压

```
tar -zxvf UnixBench5.1.3.tgz
```

# 3、编译

```
cd UnixBench/
make
```

# 4、运行

```
./Run
```

# 5、测试项


| 测试项目                              | 项目说明                                                     | 基准线      |
| ------------------------------------- | ------------------------------------------------------------ | ----------- |
| Dhrystone 2 using register variables  | 测试 string handling                                         | 116700.0lps |
| Double-Precision Whetstone            | 测试浮点数操作的速度和效率                                   | 55.0MWIPS   |
| Execl Throughput                      | 此测试考察每秒钟可以执行的 execl 系统调用的次数              | 43.0lps     |
| File Copy 1024 bufsize 2000 maxblocks | 测试从一个文件向另外一个文件传输数据的速率                   | 3960.0KBps  |
| File Copy 256 bufsize 500 maxblocks   | 测试从一个文件向另外一个文件传输数据的速率。                 | 1655.0KBps  |
| File Read 4096 bufsize 8000 maxblocks | 测试从一个文件向另外一个文件传输数据的速率。                 | 5800.0KBps  |
| Pipe-based Context Switching          | 测试两个进程（每秒钟）通过一个管道交换一个不断增长的整数的次数 | 12440.0lps  |
| Pipe Throughput                       | 一秒钟内一个进程可以向一个管道写 512 字节数据然后再读回的次数 | 4000.0lps   |
| Process Creation                      | 测试每秒钟一个进程可以创建子进程然后收回子进程的次数（子进程一定立即退出）。 | 126.0lps    |
| Shell Scripts (8 concurrent)          | 测试一秒钟内一个进程可以并发地开始一个shell 脚本的 n 个拷贝的次数，n 一般取值1，2，4，8. | 42.4lpm     |
| System Call Overhead                  | 测试进入和离开操作系统内核的代价，即一次系统调用的代价。     | 6.0lpm      |