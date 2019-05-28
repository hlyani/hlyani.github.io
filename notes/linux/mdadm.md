# 软 raid

> 由于EFI并不能安装在RAID中，以上的操作只能确保系统从第一块硬盘启动，而不能从第二块硬盘启动。如果第一块硬盘出现问题，则系统将不能启动。以下过程，将使能从第二块硬盘启动。当第一块硬盘异常的时候，系统可以从第二块硬盘正常启动。

```
将/sda1的内容，克隆到/sdb1中，如下所示：

dd if=/dev/sda1 of=/dev/sdb1

最后，将/sdb1加入到启动目录中，如下：

efibootmgr -c -g -d /dev/sdb -p 1 -L "Ubuntu #2" -l '\EFI\Ubuntu\grubx64.efi'
至此，安装的系统将可以分别从/sda和/sdb硬盘上启动。

mkfs.ext4 /dev/sda
mkfs.ext4 /dev/sdb
mdadm -C /dev/md0 -a yes -n 2 -l 1 /dev/sda /dev/sdc
mdadm -Ds >/etc/mdadm.conf
mdadm -Ds >/etc/mdadm/mdadm.conf
update-initramfs -u
mkfs.ext4 -F /dev/md0
```

```
mdadm --detail --scan

mdadm -Ds

mdadm -Ds >> /etc/mdadm/mdadm.conf
```

## 软RAID管理命令mdadm详解

##### 一、创建模式

选项：-C
专用选项：
-l 级别
-n 设备个数
-a {yes|no} 自动为其创建设备文件
-c 指定数据块大小（chunk）
-x 指定空闲盘（热备磁盘）个数，空闲盘（热备磁盘）能在工作盘损坏后自动顶替
注意：创建阵列时，阵列所需磁盘数为-n参数和-x参数的个数和

```
1、创建raid0：
1.1 创建raid
mdadm -C /dev/md0 -a yes -l 0 -n 2 /dev/sdb{1,2}
注意：用于创建raid的磁盘分区类型需为fd
1.2 格式化：
mkfs.ext4 /dev/md0
注意：在格式化时，可以指定-E选项下的stride参数指定条带是块大小的多少倍，有在一定程度上提高软RAID性能，如块默认大小为4k，而条带大小默认为64k，则stride为16，这样就避免了RAID每次存取数据时都去计算条带大小，如：
mkfs.ext4  -E stride=16 -b 4096 /dev/md0
其中stride=chunk/block，为2的n次方
2、创建raid1：
2.1 创建raid
 mdadm -C /dev/md1 -a yes -n 2 -l 1 /dev/sdb{5,6}
注意：这个提示是说软raid不能用作启动分区。
 2.2 格式化：
mkfs.ext4  /dev/md1
3、创建raid5：
由于没有磁盘空间，我将原来做raid1的测试磁盘全部删除后重新建立四个分区用于raid5测试，分别为sdb5-8
3.1 创建raid5
mdadm -C /dev/md2 -a yes -l 5 -n 3 /dev/sdb{5,6,7}
3.2 格式化：
mkfs.ext4 /dev/md2
3.3 增加热备磁盘：
 mdadm /dev/md2 -a /dev/sdb8
4、查看md状态：
4.1 查看RAID阵列的详细信息：
选项： -D = --detail
mdadm -D /dev/md#   查看指定RAID设备的详细信息
4.2 查看raid状态
cat /proc/mdstat

注意：在创建raid前，应该先查看磁盘是否被识别，如果内核还为识别，创建Raid时会报错：
cat /proc/partitions
如果没有被识别，可以执行命令：
kpartx /dev/sdb或者partprobe/dev/sdb
```

## 二、管理模式

```
选项：-a(--add)，-d(--del),-r(--remove),-f(--fail)
1、模拟损坏：
mdadm /dev/md1 -f /dev/sdb5
2、移除损坏的磁盘：
mdadm /dev/md1 -r /dev/sdb5
3、添加新的硬盘到已有阵列：
mdadm /dev/md1 -a /dev/sdb7
注意：
3.1、新增加的硬盘需要与原硬盘大小一致
3.2、如果原有阵列缺少工作磁盘（如raid1只有一块在工作，raid5只有2块在工作），这时新增加的磁盘直接变为工作磁盘，如果原有阵列工作正常，则新增加的磁盘为热备磁盘。
4、停止阵列：
选项：-S = --stop
mdadm -S /dev/md1
```

##### 三、监控模式

选项：-F
不常用，不做详细说明。

##### 四、增长模式，用于增加磁盘，为阵列扩容：

```
选项：-G
示例，将上述raid5的热备磁盘增加到阵列工作磁盘中
[root@localhost ~]# mdadm -G /dev/md2  -n 4
注意：-n 4 表示使用四块工作磁盘
再次使用-D选项查看阵列详细信息如下：

mdadm -D /dev/md2
```

##### 五、装配模式，软RAID是基于系统的，当原系统损坏了，需要重新装配RAID

```
选项：-A
示例：将上述已经停止的阵列重新装配：
mdadm -A /dev/md1 /dev/sdb5 /dev/sdb6
实现自动装配：
mdadm运行时会自动检查/etc/mdadm.conf  文件并尝试自动装配，因此第一次配置raid后可以将信息导入到/etc/mdadm.conf  中，命令如下：

[root@localhost ~]# mdadm -Ds >/etc/mdadm.conf
```

```
RAID --- 磁盘阵列,简言之,用来提高硬盘的利用率和速度
RAID种类(理论):
RAID 0 : 读写性能(最少两块硬盘)  --- 硬盘使用量是所有硬盘大小之和,性能是所有硬盘之和
RAID 1 : 读写性能,冗余性(最少两块硬盘) --- 空间利用率:所有磁盘中最小的那块(n/2); 读性能接近RAID0,写性能较raid 0 弱一些;有 冗余能力
RAID 5 : 读写性能,冗余性(至少3块硬盘)  --- 空间利用率:1-1/n .读性能接近RAID0 ,写性能较RAID0弱一些 . 冗余能力:可接受一块硬盘的损坏;
RAID 6 : 读写性能,冗余性(至少4块硬盘) --- 空间利用率:1 - 2/n .读写性能较RAID5,读性能比RAID5还要弱一些; 冗余能力:可接受2块硬盘损坏;
mdadm 常用参数解释

选项(高亮的是很常用的)： 

-f  :  fail   , 将一个磁盘设置为故障状态
-l : LEVEL , 设置磁盘阵列的级别
-r : 移除故障设备
-a : 添加新设备进入磁盘阵列
-S : 停止一个磁盘阵列
-v : --verbose：显示细节
-D, --detail： 打印一个或多个md device 的详细信息
-x :--spare-devices 指定一个备份磁盘,也就是指定初始阵列的冗余device 数目即spare device数目；

- n : 指定磁盘的个数
  -A : --assemble：加入一个以前定义的阵列 
  -B : --build：创建一个没有超级块的阵列(Build a legacy array without superblocks.) 
  -C : --create：创建一个新的阵列 
  -F : --follow, --monitor：选择监控(Monitor)模式 
  -G : --grow：改变激活阵列的大小或形态 
  -I : --incremental：添加一个单独的设备到合适的阵列，并可能启动阵列 
  --auto-detect：请求内核启动任何自动检测到的阵列 
  -h : --help：帮助信息，用在以上选项后，则显示该选项信息 
  --help-options：显示更详细的帮助 
  -V : --version：打印mdadm的版本信息 
  -b : --brief：较少的细节。用于 --detail 和 --examine 选项
  -Q : --query：查看一个device，判断它为一个 md device 或是 一个 md 阵列的一部分
  -E : --examine：打印 device 上的 md superblock 的内容
  -c : --config= ：指定配置文件，缺省为 /etc/mdadm.conf 
  -s : --scan：扫描配置文件或 /proc/mdstat以搜寻丢失的信息。配置文件/etc/mdadm.conf

使用mdadm 创建RAID (级别只是修改个数字,其他参数基本一样..)
```

