# 将 home 目录空间扩容到根目录

##### 1、查看

```
df -h
```

##### 2、取消挂载

```
[root@localhost ~]# umount /home/
```

##### 3、删除逻辑卷

```
[root@localhost ~]# lvremove /dev/mapper/centos-home
Do you really want to remove active logical volume centos/home? [y/n]: y
  Logical volume "home" successfully removed
```

##### 4、扩容 root 的逻辑卷

```
# lvextend -l 100%FREE /dev/mapper/centos-root

[root@localhost ~]# lvextend -L +120G  /dev/mapper/centos-root
  Size of logical volume centos/root changed from 50.00 GiB (12800 extents) to 170.00 GiB (43520 extents).
  Logical volume centos/root successfully resized.
```

##### 5、对挂载目录在线扩容

```
[root@localhost ~]# xfs_growfs /dev/mapper/centos-root
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=3276800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=13107200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=6400, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 13107200 to 44564480
```

##### 6、重新创建 home 的逻辑卷

```
[root@localhost ~]# lvcreate -L 47G -n home centos
  Logical volume "home" created.
[root@localhost ~]# mkfs.xfs  /dev/mapper/centos-home
meta-data=/dev/mapper/centos-home isize=512    agcount=4, agsize=3080192 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=12320768, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=6016, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@localhost ~]# mount  /dev/mapper/centos-home  /home
[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root  170G  899M  170G    1% /
devtmpfs                  63G     0   63G    0% /dev
tmpfs                     63G     0   63G    0% /dev/shm
tmpfs                     63G  9.5M   63G    1% /run
tmpfs                     63G     0   63G    0% /sys/fs/cgroup
/dev/sda2               1014M  135M  880M   14% /boot
/dev/sda1                200M  9.8M  191M    5% /boot/efi
tmpfs                     13G     0   13G    0% /run/user/0
/dev/mapper/centos-home   47G   33M   47G    1% /home
```

##### 7、查看

```
[root@localhost ~]# lsblk 
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda               8:0    0   223G  0 disk 
├─sda1            8:1    0   200M  0 part /boot/efi
├─sda2            8:2    0     1G  0 part /boot
└─sda3            8:3    0 221.8G  0 part 
  ├─centos-root 253:0    0   170G  0 lvm  /
  ├─centos-swap 253:1    0     4G  0 lvm  [SWAP]
  └─centos-home 253:2    0    47G  0 lvm  /home
sdb               8:16   0   8.9T  0 disk 
sdc               8:32   0   8.9T  0 disk
```

##### 8、检查 /etc/fstab 挂载信息

```
cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Thu Dec 12 10:05:59 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=90a54835-7cf0-4519-8b13-ff7b4a6076a7 /boot                   xfs     defaults        0 0
/dev/mapper/centos-home /home                   xfs     defaults        0 0
#/dev/mapper/centos-swap swap                    swap    defaults        0 0
```

