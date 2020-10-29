# xenserver 相关

##### 1、xenserver xva 转 raw
```
wget https://github.com/eriklax/xva-img/archive/master.zip
unzip master.zip
cd xva-img-master
apt-get -y install build-essential libssl-dev
cmake .
make install
mkdir my-virtual-machine 
tar -xf <.xva file> -C my-virtual-machine 
chmod -R 755 my-virtual-machine
xva-img -p disk-export my-virtual-machine/Ref\:1/ disk.raw
#dd if=win8.1.img of=/dev/desktop-xen/Win8.1 bs=64M
```
##### 2、xenserver虚拟机开机自动启动
```
查看所有的pool并设置pool自动启动： 
1、xe pool-list 查看所有的pool：
[root@xenserver ~]# xe pool-list
uuid ( RO)                : c7d7a7e4-77ad-e6a6-c935-4cba102881a8
          name-label ( RW): 
    name-description ( RW): 
              master ( RO): b35d1618-ad4e-4830-89da-d93788e9f082
          default-SR ( RW): 85280950-f08d-9e4d-5e51-f0ec4e221a7a

2、设置pool的自动启动：
[root@xenserver ~]# xe pool-param-set uuid=c7d7a7e4-77ad-e6a6-c935-4cba102881a8 other-config:auto_poweron=true

列出所有的虚拟机并设置自动启动： 
1、xe vm-list 列出所有的虚拟机：
[root@xenserver ~]# xe vm-list
uuid ( RO)           : adad6140-1cc8-30e9-dc4d-05fb426eaf4e
     name-label ( RW): MYSQL-MASTER
    power-state ( RO): running
uuid ( RO)           : 8e342f09-3a87-604e-11f4-96b37b8bcc40
     name-label ( RW): Windows Server 2003 (64-bit)
    power-state ( RO): running
uuid ( RO)           : d7432a76-0486-492c-84f6-eab02c52af54
     name-label ( RW): Control domain on host: xenserver
    power-state ( RO): running
2、设置所有虚拟机开机自动启动：
[root@xenserver ~]# for i in `xe vm-list params=uuid --minimal|sed 's/,/ /g'`;do xe vm-param-set uuid=$i other-config:auto_poweron=true;done
3、如果只需要设置单台虚拟机自动启动，则根据虚拟机的UUID来指定auto_poweron=true，例如我要指定上面MYSQL-MASTER这台虚拟机自动启动，则操作如下：
[root@xenserver ~]# xe vm-param-set uuid=adad6140-1cc8-30e9-dc4d-05fb426eaf4e other-config:auto_poweron=true
```
##### 3、xenserver Bongding LACP
```
bond45_uuid=`xe network-create name-label=bond45`
host_uuids=`xe host-list|grep uuid|awk '{print $5}'`

for host_uuid in $(echo $host_uuids | awk '{print;}')
do
    host_eth4_pif=`xe pif-list host-uuid=$host_uuid device=eth4|grep -E '^uuid'|awk '{print $5}'`
    host_eth5_pif=`xe pif-list host-uuid=$host_uuid device=eth5|grep -E '^uuid'|awk '{print $5}'`
    xe bond-create network-uuid=$bond45_uuid pif-uuids=$host_eth4_pif,$host_eth5_pif mode=lacp
done
```
##### 4、为xenserver挂载磁盘
```
fdisk /dev/nvme0n1

n    新建分区
p    分区类型为主分区
enter
enter

t    修改分区格式
8e   类型改为8e LVM
p    查看当前分区
w    写入分区

partprobe    使分区表生效，无需重启

pvcreate /dev/nvme0n1 --config global{metadata_read_only=0}  使用pvcreate转换

pvdisplay    查看已经存在的pv

vgcreate nvme0n1VG /dev/nvme0n1 --config global{metadata_read_only=0}  创建VG，可利用已经存在的VG名（myVG），同一VG名下的一组PV构成一个VG

vgdisplay   查看VG  创建完成VG之后，才能从VG中划分一个LV

lvcreate -l 100%FREE -n nvme0n1LV  nvme0n1VG --config global{metadata_read_only=0} 创建LV，并把VG所有剩余空间分给LV

lvdisplay   显示LV的信息

mkfs.ext4 /dev/nvme0n1VG/nvme0n1LV    对LV进行格式化（使用mksf进行格式化操作），然后LV才能存储资料

#blkid /dev/nvme0n1VG/nvme0n1LV

#mkdir /nvme0n1

#echo 'UUID=a5d3a67a-aad4-4eea-91ce-1b1fc56ffe9f  /nvme0n1 ext4 defaults 0 0' >/etc/fstab

#mount -a
#mount /dev/nvme0n1VG/nvme0n1LV /nvme0n1

#df -hP

host_uuids=`xe host-list|grep uuid|awk '{print $5}'`

xe sr-create content-type=user device-config:device=/dev/nvme0n1VG/nvme0n1LV host-uuid=12244cc8-1958-40e1-a4a3-24a1a3542293 name-label="Local storage2" shared=false type=lvm 

df -h   检查linux服务器的文件系统的磁盘空间占用情况

cat /proc/partitions

ll /dev/disk/by-id

# xe sr-create content-type=user device-config:device=/dev/disk/by-id/<scsi-xxxxxxxxxxxxxxxxxxxxxxxxx> host-uuid=<host-uuid> name-label=”Local Storage 2” shared=false type=lvm  
- Or -    
# xe sr-create content-type=user device-config:device=/dev/disk/b y-id/<cciss-xxxxxxxxxxxxxxxxxxxxxxxxx> host-uuid=<host-uuid> name-label=”Local Storage 2” shared=false type=lvm    
- Or -    
# xe sr-create content-type=user device-config:device=/dev/<sdx> host-uuid=<host-uuid> name-label=”Local Storage 2” shared=false type=lvm

xe sr-create content-type=user device-config:/dev/XSLocalEXT-83a61471-2405-72b3-a594-f1042329fd0b/83a61471-2405-72b3-a594-f1042329fd0b host-uuid=e2cde32e-aa28-4f25-8941-67fc0d398d3d name-label="Local storage" shared=false type=lvm  

找到要删除的sr
xe sr-list 
xe pbd-list sr-uuid=sr-uuid

断开连接
xe pbd-unplug uuid=pbd-uuid

删除sr
xe sr-destroy uuid=sr-uuid
xe sr-forget uuid=sr-uuid

host_uuids=`xe host-list|grep uuid|awk '{print $5}'`
lvdisplay

xe sr-create content-type=user device-config:device=/dev/<sdx> host-uuid=<host-uuid> name-label="Local Storage" shared=false type=lvm

xe sr-create content-type=user device-config:device=/dev/XSLocalEXT-37269b49-a03b-af83-5dc0-d298f7fa3a72/nvme0n1LV host-uuid=966b29da-7834-4438-a70e-b3eca14cea1e name-label="Local Storage2" shared=false type=lvm

vgdisplay

lvcreate -L 10GB -n localiso {VG Name} --config global{metadata_read_only=0}

mkdir /iso

mkfs.ext4 /dev/{VG Name}/localiso
echo 'UUID=b92d4e6d-135d-498d-8e52-58f4f4002dee  /iso ext4 defaults 0 0' >/etc/fstab

mount -a
df -hP
xe sr-create name-label=iso_image type=iso device-config:location=/iso device-config:legacy_mode=true content-type=iso
```