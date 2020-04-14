# LVM 逻辑卷相关

## 一、LVM简介

>  LVM是 Logical Volume Manager(逻辑卷管理)的简写。LVM将一个或多个硬盘的分区在逻辑上集合，相当于一个大硬盘来使用，当硬盘的空间不够使用的时候，可以继续将其它的硬盘的分区加入其中，这样可以实现磁盘空间的动态管理，相对于普通的磁盘分区有很大的灵活性。
>
> LVM是在磁盘分区和文件系统之间添加的一个逻辑层，来为文件系统屏蔽下层磁盘分区布局，提供一个抽象的盘卷，在盘卷上建立文件系统。

### 二、 LVM基本术语

* 物理存储介质（The physical media）：这里指系统的存储设备：硬盘，如：/dev/hda1、/dev/sda等等，是存储系统最低层的存储单元。
* 物理卷（physical volume）：物理卷就是指硬盘分区或从逻辑上与磁盘分区具有同样功能的设备(如RAID)，是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数。
* 卷组（Volume Group）：LVM卷组类似于非LVM系统中的物理硬盘，其由物理卷组成。可以在卷组上创建一个或多个“LVM分区”（逻辑卷），LVM卷组由一个或多个物理卷组成。
* 逻辑卷（logical volume）：LVM的逻辑卷类似于非LVM系统中的硬盘分区，在逻辑卷之上可以建立文件系统(比如/home或者/usr等)。
* PE（physical extent）：每一个物理卷被划分为称为PE(Physical Extents)的基本单元，具有唯一编号的PE是可以被LVM寻址的最小单元。PE的大小是可配置的，默认为4MB。
* LE（logical extent）：逻辑卷也被划分为被称为LE(Logical Extents) 的可被寻址的基本单位。在同一个卷组中，LE的大小和PE是相同的，并且一一对应。

简单来说就是：

* PV：是物理的磁盘分区

* VG：LVM中的物理的磁盘分区，也就是PV，必须加入VG，可以将VG理解为一个仓库或者是几个大的硬盘。

* LV：也就是从VG中划分的逻辑分区

## 三、 安装LVM

>  首先确定系统中是否安装了lvm工具：rpm –qa|grep lvm

##### 1、 创建分区

使用分区工具（如：fdisk等）创建LVM分区，方法和创建其他一般分区的方式是一样的，区别仅仅是LVM的分区类型为8e。

```
fdisk /dev/xvdb

n    新建分区
p    分区类型为主分区
4
enter
enter

t    修改分区格式
8e   类型改为8e LVM
p    查看当前分区

w    写入分区

partprobe    使分区表生效，无需重启

pvcreate /dev/xvdb4    使用pvcreate转换

pvdisplay    查看已经存在的pv

vgcreate myVG /dev/xvdb4  创建VG，可利用已经存在的VG名（myVG），同一VG名下的一组PV构成一个VG

vgdisplay   查看VG  创建完成VG之后，才能从VG中划分一个LV

lvcreate -l 100%FREE -n myLV  myVG  创建LV，并把VG所有剩余空间分给LV

lvdisplay   显示LV的信息

mkfs -t ext3 /dev/myVG/myLV    对LV进行格式化（使用mksf进行格式化操作），然后LV才能存储资料

mkdir /home/hl

mount /dev/myVG/myLV /home/hl

df -h   检查linux服务器的文件系统的磁盘空间占用情况

vim /etc/fstab

blkid /dev/myVG/myLV 

echo 'UUID=a5d3a67a-aad4-4eea-91ce-1b1fc56ffe9f  /home/hl ext4 defaults 0 0' >/etc/fstab

mount -a
```

##### 2、开机挂载及/etc/fstab格式

> 当我们在挂载磁盘的时候，除了利用磁盘的代号之外 (/dev/hdxx) 也可以直接利用磁盘的 label 来作为挂载的磁盘挂载点喔！基本上， 就是那个 /etc/fstab 档案的设定,Label 来做为磁盘挂载的依据， 这样有好有坏：
> 优点：不论硬盘代号怎么变，不论您将硬盘插在那个 IDE 接口 (IDE1 或 IDE2 或 master 或 slave 等)，由于系统是透过 Label ，所以，磁盘插在那个接口将不会有影响。
> 缺点：如果插了两颗硬盘，刚好两颗硬盘的 Label 有重复的，那就惨了～ 因为系统会无法判断那个磁盘分割槽才是正确的！

> 开机挂载 /etc/fstab 及 /etc/mtab：
>  系统挂载的一些限制：
>  根目录 / 是必须挂载的,而且一定要先于其它 mount point 被挂载进来。
>  其它 mount point 必须为已建立的目录,可任意指定,但一定要遵守必须的系统目录架构原则
>  所有 mount point 在同一时间之内,只能挂载一次。
>  所有 partition 在同一时间之内,只能挂载一次。
>  如若进行卸载,您必须先将工作目录移到 mount point(及其子目录) 之外。

```
[root@linux ~]# cat /etc/fstab

# Device Mount_point filesystem parameters dump fsck

LABEL=/       /          ext3     defaults   1     1  （以标头名称挂载）
 /dev/hda5     /home     ext3      defaults   1    2
 /dev/hda3     swap      swap     defaults   0     0
 /dev/hdc /media/cdrom     auto     pamconsole,exec,noauto,managed 0 0
 /dev/devpts       /dev/pts devpts gid=5,mode=620 0 0
 /dev/shm   /dev/shm     tmpfs      defaults   0    0
 /dev/proc     /proc     proc      defaults   0     0
 /dev/sys     /sys     sysfs      defaults   0     0
```

> 其实这个 /etc/fstab 就是将我们使用 mount 来挂载一个装置到系统的某个挂载点， 所需要下达的指令内容，将这些内容通通写到 /etc/fstab 里面去，而让系统一开机就主动挂载。 那么 mount 下达指令时，需要哪些参数？不就是『装置代号、挂载点、档案系统类别、参数』等等， 而我们的 /etc/fstab 则加入了两项额外的功能，分别是备份指令 dump 的执行与否， 与是否开机进行 fsck 扫瞄磁盘。
>  前面的4个已经很熟悉了，每个档案系统还有很多参数可以加入的，例如中文编码的 iocharset=big5,codepage=950 之类的，当然还有很多常见的参数，具体可以看mount中的详细介绍，具体说一下后2个：dump和fsck。

> 能否被 dump 备份指令作用： 
>      在 Linux 当中，可以利用 dump 这个指令来进行系统的备份的。而 dump 指令则会针对 /etc/fstab 的设定值，去选择是否要将该 partition 进行备份的动作呢！ 0 代表不要做 dump 备份， 1 代表要进行 dump 的动作。 2 也代表要做 dump 备份动作， 不过，该 partition 重要度比 1 小。

> 是否以 fsck 检验扇区：
>      开机的过程中，系统预设会以 fsck 检验我们的 partition 内的 filesystem 是否完整 (clean)。 不过，某些 filesystem 是不需要检验的，例如虚拟内存 swap ，或者是特殊档案系统， 例如 /proc 与 /sys 等等。所以，在这个字段中，我们可以设定是否要以 fsck 检验该 filesystem 喔。 0 是不要检验， 1 是要检验， 2 也是要检验，不过 1 会比较早被检验啦！ 一般来说，根目录设定为 1 ，其它的要检验的 filesystem 都设定为 2 就好了。

##### 3、其他操作

```
1、增加逻辑卷的大小
lvextend -l 100%FREE /dev/myVG/myLV

2、减小逻辑卷的大小
lvreduce -L 1G /dev/myVG/myLV

lvreduce -l -256 /dev/myVG/myLV

3、删除逻辑卷
lvremove /dev/myVG/myLV

4、删除物理卷
vgremove /dev/myVG

5、链接目录
ln -s /dev/myVG/myLV /home/hl

6、取消文件夹的绑定
umount /home/hl
```

##### 4、扩展根目录逻辑卷

```
pvcreate /dev/sdb
vgextend cenots /dev/sdb
lvextend -l +100%FREE /dev/centos/root
xfs_growfs /dev/centos/root
```

