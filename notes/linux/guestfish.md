# guestfish 相关

##### 1、安装相关软件
```
yum install  -y libguestfs-tools
```

> 注意：默认安装是不安装windows系统支持的，如果需要修改windows系统镜像，需要再运行如下命令。

```
yum install libguestfs-winsupport
```

##### 2、查看镜像中的文件
```
virt-cat tmp-CentOS7.4_x86_64-20171128.img /etc/hosts
```
##### 3、镜像磁盘空间使用查看
```
virt-df
```
##### 4、列出指定目录内文件
```
virt-ls
```
##### 5、显示指定文件内容
```
virt-cat
```
##### 6、编辑指定文件
```
virt-edit
```
##### 7、将文件拷贝到虚拟机内部
```
virt-copy-in
```
##### 8、将虚拟机内部文件拷贝出来
```
virt-copy-out
```
##### 9、tar压缩文件拷贝进虚拟机并解压
```
virt-tar-in
```
##### 10、镜像内指定目录文件拷贝并压缩
```
virt-tar-out
```
##### 11、解压或者上传文件到虚拟机
```
virt-tar
```
##### 12、交互的shell
```
guestfish --rw -a CentOS-6-x86_64-GenericCloud.qcow2 

><fs> run
><fs> list-filesystems
>/dev/sda1: ext4
><fs> mount /dev/sda1 /
><fs> edit /etc/fstab
><fs> umount /
><fs> exit
```
##### 13、guestfish修改镜像格式和大小，修改镜像格式和大小主要使用以下命令
```
virt-convert - convert virtual machines between formats
```
##### 14、转化虚拟机镜像格式
```
virt-resize - Resize a virtual machine disk
```
##### 15、修改虚拟机镜像磁盘
```
raw转qcow2格式
需要先用qemu-img命令创建一个一样大小的空qcow2格式镜像文件，然后使用virt-convert命令，原始镜像可以是 vmware镜像vmx，kvm进行，ovf的镜像。
virt-convert -i raw -o qcow2 old.img new.qcow2

将指定的分区扩大5G，创建一个新的镜像，比原来大5G，然后扩展
virt-resize --expand /dev/sda2 olddisk newdisk

将boot增加200M，剩下的空间扩充给/dev/sda2
virt-resize --resize /dev/sda1=+200M --expand /dev/sda2 \
olddisk newdisk

lv扩展
virt-resize --expand /dev/sda2 --LV-expand /dev/vg_guest/lv_root \
olddisk newdisk

扩展分区，并将raw格式转化成qcow2格式
qemu-img create -f qcow2 newdisk.qcow2 15G
virt-resize --expand /dev/sda2 olddisk newdisk.qcow2
```
> 注意：
	1、如果是扩展分区，目标磁盘文件必须大于原生磁盘；
	2、磁盘缩小比较复杂，一般要求缩小到的空间远大于文件系统的大小。