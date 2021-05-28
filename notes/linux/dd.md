# dd 相关

# 一、参数

> dd：用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换

* if=文件名，input file
* of=文件名，output file
* ibs=bytes，一次读入bytes个字节，即指定一个块大小为bytes个字节

* obs=bytes，一次输出bytes个字节，即指定一个块大小为bytes个字节
* bs=bytes，同时设置读入、输出的块大小为bytes个字节
* cbs=bytes，一次转换bytes个字节，即指定转换缓冲区大小
* skip=blocks，从输入文件开头跳过blocks个块后再开始复制
* seek=blocks，从输出文件开头跳出blocks个块后再开始复制
* count=blocks，仅拷贝blocks个块，块大小等于lbs指定的字节数
* conv=conversion，用指定的参数转换文件
  * ascii，转换ebcdic为ascii
  * ebcdic，转换ascii为 ebcdic
  * ibm，转换ascii为alternate ebcdic
  * block，把每一行转换为长度为cbs，不足部分为空格填充
  * unblock，使每一行的长度都为cbs，不足部分用空格填充
  * lcase，把大写字符转换为小写字符
  * ucase，把小写字符转换为大写字符
  * swab，交换输入的每对字符
  * noerror，出错时不停止
  * notrunc，不截短输出文件
  * sync，将每个输入块填充到ibs个字节，不足部分用空（NUL）字符补齐

> /dev/null，无底洞，可以向它输入任何数据。是一个空设备，也成为位桶（bit bucket），任何写入它的输出都会被抛弃，如果不想让消息以标准输出显示或写入文件，可以将消息重定向到位桶。

> /dev/zero，是一个输入设备，可以用来初始化文件。该设备无穷尽的提供0。

# 二、示例

##### 1、将本地/dev/sda整盘备份到/dev/sdb

```
dd if=/dev/sda of=/dev/sdb
```

##### 2、将/dev/sda全盘数据备份到指定路径的image文件

```
dd if=/dev/sda of=/root/image

dd if=/dev/sda of=/root/image bs=2M status='progress'
```

##### 3、将备份文件恢复到指定盘

```
dd if=/root/image of=/dev/sda
```

##### 4、备份/dev/sda全盘数据，并利用gzip工具进行压缩，保存到指定路径

```
dd if=/dev/sda | gzip > /root/image.gz
```

##### 5、将压缩的备份文件恢复到指定盘

```
gzip -dc /root/image.gz | dd of=/dev/sda
```

##### 6、拷贝内存内容到硬盘

```
dd if=/dev/mem of=/root/mem.bin bs=1024(指定块大小为1K)
```

##### 7、拷贝光盘内容到指定文件夹，并保存为cd.iso文件

```
dd if=/dev/cdrom(hdc) of=/root/cd.iso
```

##### 8、增加swap分区文件大小

```
第一步：创建一个大小为256M的文件
dd if=/dev/zero of=/swapfile bs=1024 count=262144

第二步：把这个文件变成swap文件
mkswap /swapfile

第三步：启用这个swap文件
swapon /swapfile

第四步：编辑/etc/fstab文件
/swapfile swap swap default 0 0
```

##### 9、销毁磁盘数据

```
dd if=/dev/urandom of=/dev/sda
```

##### 10、测试磁盘的读写速度

```
dd if=/dev/zero bs=1024 count=1000000 of=/root/1GB.file
dd if=/dev/1GB.file bs=64K | dd of=/dev/null
```

##### 11、确定磁盘的最佳块大小

```
dd if=/dev/zero bs=1024 count=1000000 of=/root/1GB.file
dd if=/dev/zero bs=2048 count=500000 of=/root/1GB.file
dd if=/dev/zero bs=4096 count=250000 of=/root/1GB.file
dd if=/dev/zero bs=8192 count=125000 of=/root/1GB.file
```

##### 12、修复磁盘

> 当硬盘较长时间(一年以上)放置不使用后，磁盘上会产生magnetic flux point，当磁头读到这些区域时会遇到困难，并可能导致I/O错误。当这种情况影响到硬盘的第一个扇区时，可能导致硬盘报废。上边的命令有可能使这些数 据起死回生。并且这个过程是安全、高效的。

```
dd if=/dev/sda of=/dev/sda
```

##### 13、禁止标准输出

```
cat $filename > /dev/null
```

##### 14、禁止标准错误

```
rm $badname 2>/dev/null
```

##### 15、禁止标准输出和标准错误输出

```
cat $filename 2>/dev/null >/dev/null
#cat $filename &>/dev/null
```

##### 16、自动清空日志文件内容

```
cat /dev/null > /var/log/messages

# :>/var/log/messages 有同样的效果，但不会产生新的进程（因为:是内建的）
```

