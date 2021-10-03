# OpenStack 相关

##### 1、查看虚拟机console地址

```
openstack console url show novatest
```

##### 2、虚拟机开机启动

```
resume_guests_state_on_host_boot=true
```

##### 3、将虚拟机状态改为active

```
nova reset-state --active fcfcb78b-4a58-49da-af39-d6c08a109db7
```

##### 4、扩展传统模式根磁盘

```
qemu-img info aa.qcow2 
qemu-img resize aa.qcow2 +10G

lsblk
resize2fs /dev/vda1
yum install xfsprogs
xfs_growfs -d /mnt

lsblk

df -h
```

##### 5、使用virsh挂载卷

```
pvcreate /dev/xvdb4    使用pvcreate转换

pvdisplay    查看已经存在的pv

vgcreate myVG /dev/xvdb4  创建VG，可利用已经存在的VG名（myVG），同一VG名下的一组PV构成一个VG

vgdisplay   查看VG  创建完成VG之后，才能从VG中划分一个LV

lvcreate -l 100%FREE -n myLV  myVG

mkfs -t ext4 /dev/vg/instances
mkfs -t ext4 /dev/vg/edu
blkid /dev/myVG/myLV 
lvcreate -L 500G -n edu  vg

virsh attach-disk --domain instance-00000110 --source /dev/mysdb/edu --target vdb --persistent
```

##### 6、resize、迁移功能

```
修改nova配置
nova.conf中，将allow_resize_to_same_host=True注释掉

一、

$ ssh-keygen -f id_rsa -b 1024 -P ""  
$ cp -a /var/lib/nova/.ssh/id_rsa.pub /var/lib/nova/.ssh/authorized_keys  

$ scp /var/lib/nova/.ssh/id_rsa root@otherHost:/var/lib/nova/.ssh/id_rsa  
$ scp /var/lib/nova/.ssh/authorized_keys root@otherHost:/var/lib/nova/.ssh/authorized_keys  

# chown nova:nova /var/lib/nova/.ssh/id_rsa /var/lib/nova/.ssh/authorized_keys  


二、

vim /etc/libvirt/libvirtd.conf
listen_tls = 0
listen_tcp = 1
tcp_port = "16509"
listen_addr = "0.0.0.0"
unix_sock_group = "root"
auth_tcp = "none"

vim /etc/init/libvirt-bin.conf
env libvirtd_opts="-d -l"

vim /etc/init/nova-compute.conf
exec start-stop-daemon --start --chuid root --exec /usr/bin/nova-compute -- --config-file=/etc/nova/nova.conf --config-file=/etc/nova/nova-compute.conf

service libvirt-bin restart

virsh -c qemu+tcp://compute2/system

```

##### 7、修改linux密码

#cloud-config
```
userdata = '''

chpasswd:
  list: |
    root:{password}
  expire: False

'''

nova boot --user-data=userdata
```

##### 8、修改cpu quota

```
openstack quota list --compute
openstack quota set --cores 200 c2978d74024447b7bf0cacbd4b4e9af6
#openstack quota set --cores -1 c2978d74024447b7bf0cacbd4b4e9af6
```

