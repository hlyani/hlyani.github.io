# Linux 相关

## 一、pigz

##### 1、安装

```
apt-get -y install pigz
```

##### 2、压缩

```
tar cvf - 目录名 | pigz -9 -p 24 > file.tgz
#pigz：用法-9是压缩比率比较大，-p是指定cpu的核数。
tar cvf - /opt/hadoop-2.7.3.tar.gz | pigz -9 -p 4 >hadoop-2.7.3.tgz
```

##### 3、解压

```
pigz -d file.tgz
tar -xf file
#tar -xf --format=posix file
```

## 二、Linux系统启动过程

> Linux系统的启动过程：内核引导、运行init、系统初始化、建立终端、用户登录系统

##### 1、内核引导

> 当计算机打开电源后，首先是BIOS开机自检，按照BIOS中设置的设备的启动设备（通常是硬盘）来启动。操作系统接管软件后，首先读入/boot目录下的内核文件。

```
操作系统 -> /boot
```

##### 2、运行 init

> init进程是系统所有进程的起点，没有这个进程，系统中任何进程都不会启动。init程序首先是需要读取配置文件/etc/inintab。

```
操作系统 -> /boot -> init进程
```

###### 运行级别

> 许多程序需要开机启动。windows叫做“服务”（service）在linux叫做“守护进程”（daemon）。
>
> init进程的一大任务，就是去运行这些开机启动程序。
>
> 但，不同场合需要启动不同的程序，比如用作服务器时，需要启动apache，用作桌面就不需要。
>
> linux允许为不同的场合，分配不同的开机启动程序，这叫做“运行级别”（runlevel）。也就是，启动时根据“运行级别”，确定要运行哪些程序。

```
操作系统 -> /boot -> init进程 -> 运行级别
```

linux系统有7个运行级别（runlevel）：

* 运行级别0：系统停机状态，系统默认运行级别不能设为0，否则不能正常启动
* 运行级别1：单用户工作状态，root权限，用户系统维护，禁止远程登录
* 运行级别2：多用户状态（没有NFS）
* 运行级别3：完全的多用户状态（有NFS），登录后进入控制台命令行模式
* 运行级别4：系统未使用，保留
* 运行级别5：x11控制台，登录后进入图形GUI模式
* 运行级别6：系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动

##### 3、系统初始化

> 在init的配置文件中有这么一行： si::sysinit:/etc/rc.d/rc.sysinit 它调用执行了/etc/rc.d/rc.sysinit，而rc.sysinit是一个bash shell的脚本，它主要是完成一些系统初始化的工作，rc.sysinit是每一个运行级别都要首先运行的终于脚本。
>
> 它主要完成的工作有：激活交换分区，检查磁盘，加载硬件模块以及其它一些需要优先执行的任务。

```
15:5:wait:/etc/rc.d/rc 5
```

> 这一行表示以5为参数运行/etc/rc.d/rc,/etc/rc.d/rc是一个shell脚本，它接受5作为参数，去执行/etc/rc.d/rc5.d/目录下的所有的rc启动脚本，/etc/rc.d/rc5.d/目录中的这些启动脚本实际上都是一些连接文件，而不是真正的rc启动脚本，真正的rc启动脚本实际上都是放在/etc/rc.d/init.d/目录下。而这些rc启动搅拌有着类似的用法，它们接受start、stop、restart、status等参数。

```
操作系统 -> /boot -> init进程 -> 运行级别 ->/etc/init.d
```

##### 4、建立终端

> rc执行完毕后，返回init。这时基本系统环境已经设置好了，各种守护进程也已经启动了。init接下来会打开6个终端，以便用户登录系统。

##### 5、用户登录系统

> 一般来说，用户的登录方式有三种：

（1）命令行登录

（2）ssh登录

（3）图形界面登录

```
操作系统 -> /boot -> init进程 -> 运行级别 -> /etc/init.d -> 用户登录
```

##### 6、图形模式与文字模式的切换方式

```
操作系统 -> /boot -> init进程 -> 运行级别 -> /etc/init.d -> 用户登录 -> Login shell
```

##### 7、Linux 关机

```
正确的关机流程为：sync > shutdown > reboot > halt
```

## 三、dd命令

##### 1、参数

- if=文件名：输入文件名，缺省为标准输入。即指定源文件。
- of=文件名：输出文件名，缺省为标准输出。即指定目的文件。
- ibs=bytes：一次读入bytes个字节，即指定一个块大小为bytes个字节。
  obs=bytes：一次输出bytes个字节，即指定一个块大小为bytes个字节。
  bs=bytes：同时设置读入/输出的块大小为bytes个字节。
- cbs=bytes：一次转换bytes个字节，即指定转换缓冲区大小。
- skip=blocks：从输入文件开头跳过blocks个块后再开始复制。
- seek=blocks：从输出文件开头跳过blocks个块后再开始复制。
- count=blocks：仅拷贝blocks个块，块大小等于ibs指定的字节数。
- conv=<关键字>，关键字可以有以下11种：
  - conversion：用指定的参数转换文件。
  - ascii：转换ebcdic为ascii
  - ebcdic：转换ascii为ebcdic
  - ibm：转换ascii为alternate ebcdic
  - block：把每一行转换为长度为cbs，不足部分用空格填充
  - unblock：使每一行的长度都为cbs，不足部分用空格填充
  - lcase：把大写字符转换为小写字符
  - ucase：把小写字符转换为大写字符
  - swab：交换输入的每对字节
  - noerror：出错时不停止
  - notrunc：不截短输出文件
  - sync：将每个输入块填充到ibs个字节，不足部分用空（NUL）字符补齐。
- --help：显示帮助信息
- --version：显示版本信息

##### 1、将本地的/dev/hdb整盘备份到/dev/hdd

```
dd if=/dev/hdb of=/dev/hdd
```

##### 2、将/dev/hdb全盘数据备份到指定路径image文件

```
dd if=/dev/hdb of=/root/image
```

##### 3、见备份文件恢复到指定盘

```
dd if=/root/image of=/dev/hdb
```

##### 4、备份/dev/hdb全盘数据，并利用gzip工具进行压缩。保存到指定路径

```
dd if=/dev/hdb | gzip > /root/image.gz
```

##### 5、将压缩的备份文件恢复到指定盘

```
gzip -dc /root/image.gz | dd of=/dev/hdb
```

##### 6、备份与恢复MBR

> 备份磁盘开始的512个字节大小的MBR信息到指定文件

```
dd if=/dev/hda of=/root/image count=1 bs=512
```

> count=1指仅拷贝一个块，bs=512指块大小为512字节。

恢复，将备份的MBR信息写到磁盘的开始部分

```
dd if=/root/image of=/dev/hda
```

##### 7、备份软盘

```
dd if=/dev/fd0 of=disk.img count=1 bs=1440k
```

##### 8、拷贝内存内容到硬盘

```
dd if=/dev/mem of=/root/mem.bin bs=1024(块大小1k)
```

##### 9、拷贝光盘内容到指定文件夹，并保存为cd.iso文件

```
dd if=/dev/cdrom of=/root/cs.iso
```

##### 10、增加swap分区文件大小

```
1、创建一个大小为256M的文件
dd if=/dev/zero of=/swapfile bs=1024 count=262144

2、把这个文件变成swap文件
mkswap /swapfile

3、启用这个swap文件
swapon /swapfile

4、编辑/etc/fstab文件，使在每次开机时自动加载swap文件
/swapfile swap swap default 0 0
```

##### 11、销毁磁盘数据(利用随机数据填充硬盘)

```
dd if=/dev/urandom of=/dev/hda1
```

##### 12、测试硬盘读写速度

> 通过以下两个命令输出的命令执行时间，可以计算出硬盘的读写速度。

```
dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
dd if=/root/1Gb.file bs=64k | dd of=/dev/null
```

##### 13、/dev/null和/dev/zero的区别

* /dev/null：无底洞，是空设备，可以向它输入任何数据，通吃。
* /dev/zero：是一个输入设备，可以用它来初始化文件。该设备无穷尽的提供0，可以使用任何需要的数目。可以用于向设备或文件写入字符串0。

```
dd if=/dev/zero of=./test.txt bs=1k count=1

#禁止标准输出
cat /etc/hosts >/dev/null

#禁止标准错误
cat /etc/hosts 2>/dev/null

#禁止标准输出和标准错误的输出
cat /etc/hosts 2>/dev/null >/dev/null
```

## 四、别名

```
alias aming='pwd'
unalias aming
```

## 五、";","&&","||"

```
使用 ”;” 时，不管command1是否执行成功都会执行command2；

使用 “&&” 时，只有command1执行成功后，command2才会执行，否则command2不执行；

使用 “||” 时，command1执行成功后command2 不执行，否则去执行command2，总之command1和command2总有一条命令会执行。
```

## 六、/proc

> /proc 文件系统是一个伪的文件系统，就是说它是一个实际上不存在的目录，因而这是一
> 个非常特殊的目录。它并不存在于某个磁盘上，而是由核心在内存中产生。这个目录用于提
> 供关于系统的信息。下面说明一些最重要的文件和目录(/proc 文件系统在proc man页中有更详
> 细的说明)。

1. ##### /proc/x

   关于进程x的信息目录，这一x是这一进程的标识号。每个进程在/proc 下有一个名为自
   己进程号的目录。

2. ##### /proc/cpuinfo

   存放处理器( c p u )的信息，如c p u的类型、制造商、型号和性能等。

3. ##### /proc/devices

   当前运行的核心配置的设备驱动的列表。

4. ##### /proc/dma

   显示当前使用的d m a通道。

5. ##### /proc/filesystems

   核心配置的文件系统信息。

6. ##### /proc/interrupts

   显示被占用的中断信息和占用者的信息，以及被占用的数量。

7. ##### /proc/ioports

   当前使用的i / o端口。

8. ##### /proc/kcore

   系统物理内存映像。与物理内存大小完全一样，然而实际上没有占用这么多内存；它仅仅是在程序访问它时才被创建。(注意：除非你把它拷贝到什么地方，否则/proc 下没有任何东西占用任何磁盘空间。)

9. ##### /proc/kmsg

   核心输出的消息。也会被送到s y s l o g。

10. ##### /proc/ksyms

    核心符号表。

11. ##### /proc/loadavg

    系统“平均负载”； 3个没有意义的指示器指出系统当前的工作量。

12. ##### /proc/meminfo

    各种存储器使用信息，包括物理内存和交换分区( s w a p )。

13. ##### /proc/modules

    存放当前加载了哪些核心模块信息。

14. ##### /proc/net

    网络协议状态信息。

15. ##### /proc/self

    存放到查看/proc 的程序的进程目录的符号连接。当2个进程查看/proc 时，这将会是不同的连接。这主要便于程序得到它自己的进程目录。

16. ##### /proc/stat

    系统的不同状态，例如，系统启动后页面发生错误的次数。

17. ##### /proc/uptime

    系统启动的时间长度。

18. ##### /proc/version

    核心版本

## 七、grep

* -E ：开启扩展（Extend）的正则表达式。

* -i ：忽略大小写（ignore case）。

* -v ：反过来（invert），只打印没有匹配的，而匹配的反而不打印。

* -n ：显示行号

* -w ：被匹配的文本只能是单词，而不能是单词中的某一部分，如文本中有liker，而我搜寻的只是like，就可以使用-w选项来避免匹配liker

* -c ：显示总共有多少行被匹配到了，而不是显示被匹配到的内容，注意如果同时使用-cv选项是显示有多少行没有被匹配到。

* -o ：只显示被模式匹配到的字符串。

* --color :将匹配到的内容以颜色高亮显示。

* -A  n：显示匹配到的字符串所在的行及其后n行，after

* -B  n：显示匹配到的字符串所在的行及其前n行，before

* -C  n：显示匹配到的字符串所在的行及其前后各n行，context

##### 1、-A 显示匹配到的字符串所在的行及其后n行，after

```
grep -A 2 "core id" /proc/cpuinfo
```

##### 2、-B 显示匹配到的字符串所在的行及其前n行，before

```
grep -B 2 "core id" /proc/cpuinfo
```

##### 3、-C 显示匹配到的字符串所在的行及其前后n行，context

```
grep -C 2 "core id" /proc/cpuinfo
```

##### 4、grep -E "word1|word2|word3" file.txt

##### 5、egrep '^root|bash$' passwd

## 八、tar 分卷压缩

```
举例说明：
要将目录logs打包压缩并分割成多个1M的文件，可以用下面的命令：
tar cjf - logs/ |split -b 1m - logs.tar.bz2.
完成后会产生下列文件：
logs.tar.bz2.aa, logs.tar.bz2.ab, logs.tar.bz2.ac
要解压的时候只要执行下面的命令就可以了：
cat logs.tar.bz2.a* | tar xj

再举例：
要将文件test.pdf分包压缩成500 bytes的文件：
tar czf - test.pdf | split -b 500 - test.tar.gz
最后要提醒但是那两个"-"不要漏了，那是tar的ouput和split的input的参数。
tar cjf - logs/ |split -b 1m - logs.tar.bz2.
完成后会产生下列文件：
logs.tar.bz2.aa, logs.tar.bz2.ab, logs.tar.bz2.ac
要解压的时候只要执行下面的命令就可以了：
cat logs.tar.bz2.a* | tar xj
```

## 九、修改用户名、密码

```
vim /etc/hostname
vim /etc/hosts

vim /etc/passwd
vim /etc/shadow

passwd {your_name}
```

