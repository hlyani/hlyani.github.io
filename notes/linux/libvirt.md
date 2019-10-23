# [libvirt 相关](/etc/sysconfig/libvirtd )

### 一、安装相关

##### 1、安装libvrit-python

```
yum install -y libvirt-devel gcc-c++ qemu-kvm

pip install libvirt-python

# 测试
python -c 'import libvirt'
```



##### 2、安装libvirt客户端

```
yum install -y libvirt-client

# 测试
virsh list
```

### 二、libvirt python api连接

[libvirt connections](https://libvirt.org/docs/libvirt-appdev-guide-python/en-US/html/libvirt_application_development_guide_using_python-Connections.html#idp2797296)

##### 1、本地连接

```
import sys
import libvirt

conn = libvirt.open('qemu:///system')
if conn == None:
    print('Failed to open connection to qemu:///system', file=sys.stderr)
    exit(1)
conn.close()
exit(0)
```

##### 2、远程tcp免密连接

> 1、修改 /etc/libvirt/libvirtd.conf

```
listen_tls = 0
listen_tcp = 1
listen_addr = "0.0.0.0"
auth_tcp = "none"
```

> 2、修改/etc/sysconfig/libvirtd，注释掉LIBVIRTD_ARGS="--listen"

```
#LIBVIRTD_ARGS="--listen"
```

> 3、重启libvirtd

```
systemctl restart libvirtd
```

> 4、查看进程和端口

```
ps aux | grep libvirtd

root     17419  0.0  0.0 2494136 50312 ?       Ssl  Oct22   0:01 /usr/sbin/libvirtd --listen
```

```
netstat -lntp | grep libvirtd

tcp        0      0 0.0.0.0:16509           0.0.0.0:*               LISTEN      17419/libvirtd
```

> 5、连接测试

```
python

import libvirt
conn = libvirt.open("qemu+tcp://192.168.0.1/system")
conn.listDefinedDomains()
conn.listDomainsID()
```

### 三、隔离 CPU（isolcpus）

>  `isolcpus=2,3`，禁止普通进程运行在编号为2,3的cpu上 

##### 1、修改配置 /etc/default/grub

```
GRUB_CMDLINE_LINUX="... isolcpus=2,3"

# 使生效
grub2-mkconfig -o /boot/grub2/grub.cfg
```

或

```
df -h
...
/dev/sdb2                1014M   72M  943M   8% /boot
/dev/sdb1                 200M   12M  189M   6% /boot/efi
...

lscpu
...
CPU(s):                6
On-line CPU(s) list:   0-5
...

cat /boot/efi/EFI/centos/grub.cfg
...
linuxefi /vmlinuz-4.19.31-rt18 root=/dev/mapper/deltaos-root ro crashkernel=auto rd.lvm.lv=deltaos/root rd.lvm.lv=deltaos/swap rhgb quiet LANG=en_US.UTF-8 isolcpus=2,3
...
```

##### 2、查看

```
cat /boot/grub2/grub.cfg |grep isolcpus
或
cat /boot/efi/EFI/centos/grub.cfg |grep isolcpus
```

##### 3、重启服务器，并查看

```
[root@dao8 ~]# ps -eLo psr|grep 0|wc -l
74

[root@dao8 ~]# ps -eLo psr|grep 3|wc -l
12
```

>  具体查看这12个进程 

```
[root@dao8 ~]# ps -eLo psr,ruser,pid,lwp,args|awk '{if($1==2)print $0}'
  2 root        15    15 [kswork]
  2 root        28    28 [cpuhp/2]
  2 root        29    29 [migration/2]
  2 root        30    30 [posixcputmr/2]
  2 root        31    31 [rcuc/2]
  2 root        32    32 [ktimersoftd/2]
  2 root        33    33 [ksoftirqd/2]
  2 root        35    35 [kworker/2:0H-xf]
  2 root        62    62 [rcu_tasks_kthre]
  2 root        63    63 [kauditd]
  2 root        67    67 [writeback]
  2 root        78    78 [ata_sff]
  2 root        82    82 [kworker/u13:0]
  2 root        85    85 [nfsiod]
  2 root        96    96 [user_dlm]
  2 root       135   135 [irq/120-aerdrv]
  2 root       137   137 [i915/signal:0]
  2 root       138   138 [i915/signal:1]
  2 root       139   139 [i915/signal:2]
  2 root       140   140 [i915/signal:6]
  2 root       141   141 [drbd-reissue]
  2 root       146   146 [fc_rport_eq]
  2 root       148   148 [fnic_fip_q]
  2 root       152   152 [bnx2i_thread/2]
  2 root       157   157 [scsi_eh_0]
  2 root       158   158 [scsi_tmf_0]
  2 root       159   159 [nvme-wq]
  2 root       167   167 [scsi_eh_2]
  2 root       169   169 [scsi_eh_3]
  2 root       170   170 [scsi_tmf_3]
  2 root       171   171 [scsi_eh_4]
```

>  其中：`migration`用于进程在不通cpu间迁移,两个`kworker`用户处理`workqueues`，`ksoftirqd`用户调度CPU软中断的进程，也就说没有普通进程，说明CPU隔离生效了 

##### 4、把某线程固定到某CPU上

```
taskset -p 0x4 3497
```

>  注：`0x4`二进制是`0100`，可知是CPU2，`3497`是线程号`lwp` 如果是`0x3`，即`0011`，那么可能绑定到cpu0或cpu1 