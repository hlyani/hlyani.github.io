# 虚拟内存盘

##### 1、创建文件夹，并将文件夹作为内存盘
```
mkdir /mnt/ramdisk
mount -t tmpfs tmpfs /mnt/ramdisk  -o size=2G,defaults,noatime,mode=777
```
##### 2、设置开机挂载
```
vim /etc/rc.local
mount -t tmpfs tmpfs /mnt/ramdisk  -o size=2G,defaults,noatime,mode=777
```
##### 3、查看
```
df -h
```
> 凡是标注着tmpfs的都是虚拟硬盘，例如建立的 /mnt/ramdisk