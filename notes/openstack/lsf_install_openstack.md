# Lfs上安装OpenStack

# 一、环境准备

## 1、更新内核

* 支持ip6tables

  ```
  ip6tables-restore v1.4.21: ip6tables-restore: unable to initialize table 'filter'
  
  Error occurred at line: 2
  Try `ip6tables-restore -h' or 'ip6tables-restore --help' for more information.
  ```

  > CONFIG_NF_TABLES=m
  > CONFIG_NF_TABLES_INET=m
  > CONFIG_IP_NF_SECURITY=m
  > CONFIG_IP6_NF_SECURITY=m

* 支持ovs

  > TASK [module-load : Load modules] *********************************************************************************************************************************
  > failed: [localhost] (item=openvswitch) => {"ansible_loop_var": "item", "changed": false, "item": {"name": "openvswitch"}, "msg": "modprobe: FATAL: Module openvswitch not found in directory /lib/modules/4.19.37-rt19\n", "name": "openvswitch", "params": "", "rc": 1, "state": "present", "stderr": "modprobe: FATAL: Module openvswitch not found in directory /lib/modules/4.19.37-rt19\n", "stderr_lines": ["modprobe: FATAL: Module openvswitch not found in directory /lib/modules/4.19.37-rt19"], "stdout": "", "stdout_lines": []}

> 更新内核

```
rm -rf /lib/modules/*
tar -zxvf 4.19.37-rt19.tar.gz -C /lib/modules/ 
cp vmlinuz-4.19.37-rt19 /boot/vmlinuz-4.19.37-rt19 
reboot
```

## 2、安装python3.8

### a、安装

```
#yum install gcc openssl-devel bzip2-devel libffi-devel
wget https://www.python.org/ftp/python/3.8.2/Python-3.8.2.tgz  
tar xzf Python-3.8.2.tgz  
cd Python-3.8.2  
./configure --enable-optimizations  
make altinstall  
```

### b、安装虚拟环境

```
/usr/local/bin/python3.8 -m venv /root/venv38
source /root/venv38/bin/activate
```

## 3、配置网络（两张网卡）

```
vim ifconfig.enp1s0f0
ONBOOT=yes
IFACE=enp1s0f0
STP=yes
VIRTINT=yes
CHECK_LINK=no
PREFIX=24
IP_FORWARD=true
INTERFACE_COMPONENTS=enp1s0f0
IP=192.168.0.34
GATEWAY=192.168.0.1
BROADCAST=192.168.0.255
SERVICE="ipv4-static"
```

```
vim ifconfig.enp1s0f1
ONBOOT=yes
IFACE=enp1s0f1
STP=yes
VIRTINT=yes
CHECK_LINK=no
PREFIX=24
IP_FORWARD=true
INTERFACE_COMPONENTS=enp1s0f1
SERVICE="ipv4-static"
```

## 4、pip源配置

```
mkdir -p /root/.pip/

vim /root/.pip/pip.conf
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
```

## 5、准备cinder nfs卷

```
mkdir /kolla_nfs
mkfs.ext4 /dev/sdb

vim /etc/fstab
/dev/sdb /kolla_nfs/ ext4 defaults 0 0

mount -a

#yum install -y nfs-utils

vim /etc/exports 
/kolla_nfs 192.168.5.0/24(rw,sync,no_root_squash)
#systemctl restart nfs

vim /etc/kolla/config/nfs_shares
node1:/kolla_nfs
node2:/kolla_nfs
```

```
# lvm
#pvcreate /dev/sdb
#vgcreate cinder-volumes /dev/sdb
#vim /etc/kolla/globals.yml
#enable_cinder: "yes"
#enable_cinder_backend_lvm: "yes"
#cinder_volume_group: "cinder-volumes"
```

## 6、ansible 优化

```
vim /etc/ansible/ansible.cfg
[defaults]
host_key_checking=False
pipelining=True
forks=100
```

## 7、拷贝其他主机上的/etc/modules-load.d 文件夹

```
scp -r 192.168.0.30:/etc/modules-load.d/ /etc/
```

## 8、启动docker

```
mkdir -p /var/lib/nova/mnt /var/lib/nova/mnt1
mount --bind /var/lib/nova/mnt1 /var/lib/nova/mnt
mount --make-shared /var/lib/nova/mnt
mount --make-shared /run
/etc/cgroupfs-mount.sh
dockerd &
```

## 9、docker load 相关镜像

```
(venv38) root [ ~ ]# docker images
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
kolla/centos-source-nova-compute                train               5b0613547d7c        8 days ago          1.89GB
kolla/centos-source-cinder-volume               train               7b72b6446cdc        8 days ago          1.56GB
kolla/centos-source-neutron-server              train               431c793225c7        8 days ago          1.02GB
kolla/centos-source-neutron-openvswitch-agent   train               19b7ac330af6        8 days ago          1GB
kolla/centos-source-cinder-api                  train               b49c2a941c36        8 days ago          1.09GB
kolla/centos-source-neutron-l3-agent            train               7cda11cc0053        8 days ago          1.04GB
kolla/centos-source-neutron-metadata-agent      train               abc6e7247cec        8 days ago          1GB
kolla/centos-source-neutron-dhcp-agent          train               003ca47a41f4        8 days ago          1GB
kolla/centos-source-nova-api                    train               b79a7994ba77        8 days ago          1.08GB
kolla/centos-source-cinder-scheduler            train               d626a789ffdf        8 days ago          1.02GB
kolla/centos-source-nova-novncproxy             train               1fcacbbd1017        8 days ago          1.06GB
kolla/centos-source-nova-conductor              train               09158e7ee9f5        8 days ago          1.02GB
kolla/centos-source-nova-scheduler              train               2698f00947df        8 days ago          1.02GB
kolla/centos-source-glance-api                  train               09e781e18202        8 days ago          951MB
kolla/centos-source-horizon                     train               5e434948e0a4        8 days ago          1.03GB
kolla/centos-source-placement-api               train               e67f15a6515e        8 days ago          921MB
kolla/centos-source-keystone                    train               60d1ef5e4b57        8 days ago          919MB
kolla/centos-source-keystone-fernet             train               daa9ade1ad37        8 days ago          920MB
kolla/centos-source-keystone-ssh                train               65f2184003de        8 days ago          921MB
kolla/centos-source-openvswitch-vswitchd        train               03947aba6136        8 days ago          428MB
kolla/centos-source-openvswitch-db-server       train               90bdff01909b        8 days ago          428MB
kolla/centos-source-kolla-toolbox               train               5bad6e6ae2f2        8 days ago          833MB
kolla/centos-source-nova-libvirt                train               ea7102c16951        8 days ago          1.26GB
kolla/centos-source-memcached                   train               2f5c7c833559        8 days ago          410MB
kolla/centos-source-fluentd                     train               efe54c6b7b37        8 days ago          667MB
kolla/centos-source-mariadb                     train               ce32f151ffcd        8 days ago          594MB
kolla/centos-source-rabbitmq                    train               cd84314358ba        8 days ago          489MB
kolla/centos-source-cron                        train               7624bb53fa55        8 days ago          409MB
```

# 二、安装

## 1、修改hosts、hostname

```
vim /etc/hosts
192.168.0.34 node1

vim /etc/hostname
node1
```

## 2、安装ansible

```
pip install ansible
```

## 3、获取kolla-ansible源码

> 可以从其他环境拉取下来，安装好，把venv环境打包到lfs环境中

```
git clone https://github.com/openstack/kolla-ansible.git
cd kolla-ansible/
git checkout stable/train
pip install -r requirements.txt
```

## 4、修改kolla-ansible

### a、修改内核模块路径

```
vim /root/venv38/lib/python3.8/site-packages/ansible/modules/system/modprobe.py
            #builtin_path = os.path.join('/lib/modules/', uname_kernel_release.strip(),
            builtin_path = os.path.join('/lib/modules/', '/lib/modules/4.19.37-rt19',
```

### b、修改ansible路径

> (venv38) root [ ~ ]# kolla-ansible --help
> /root/venv38/bin/kolla-ansible: line 7: which: command not found
> ERROR: Ansible is not installed in the current (virtual) environment.

```
vim /root/venv38/bin/kolla-ansible
ansible_path=/root/venv38/bin/ansible
```

### c、prechecks报错

```
vim kolla-ansible/ansible/roles/prechecks/vars/main.yml
  Lfs:
    - "2"
```

## 5、kvm权限问题

> 部署完后进入nova_libvirt容器

```
virt-host-validate
```

```
virsh capabilities | grep domain
```

```
cat /var/cache/libvirt/qemu/capabilities/
cat /usr/share/libvirt/cpu_map.xml
```

> dmesg
>
> docker logs nova_libvirt

> 容器内libvirt debug信息

```
2020-07-23 06:48:34.871+0000: 46: debug : virFileCacheValidate:289 : Creating data for '/usr/libexec/qemu-kvm'
2020-07-23 06:48:34.872+0000: 46: debug : virFileMakePathHelper:3093 : path=/var/cache/libvirt/qemu/capabilities mode=0777
2020-07-23 06:48:34.872+0000: 46: debug : virFileMakePathHelper:3093 : path=/var/cache/libvirt/qemu mode=0777
2020-07-23 06:48:34.872+0000: 46: debug : virFileCacheLoad:149 : No cached data '/var/cache/libvirt/qemu/capabilities/3c76bc41d59c0c7314b1ae8e63f4f765d2cf16abaeea081b3ca1f5d8732f7bb1.xml' for '/usr/libexec/qemu-kvm'
2020-07-23 06:48:34.872+0000: 46: debug : virFileClose:111 : Closed fd 20
2020-07-23 06:48:34.872+0000: 46: info : virObjectNew:248 : OBJECT_NEW: obj=0x7f5ff00f5570 classname=virQEMUCaps
2020-07-23 06:48:34.872+0000: 46: debug : virQEMUCapsInitQMPCommandRun:4400 : Try to probe capabilities of '/usr/libexec/qemu-kvm' via QMP, machine none,accel=kvm:tcg
2020-07-23 06:48:34.872+0000: 46: debug : virCommandRunAsync:2585 : About to run LC_ALL=C PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin HOME=/root /usr/libexec/qemu-kvm -S -no-user-config -nodefaults -nographic -machine none,accel=kvm:tcg -qmp unix:/var/lib/libvirt/qemu/capabilities.monitor.sock,server,nowait -pidfile /var/lib/libvirt/qemu/capabilities.pidfile -daemonize
2020-07-23 06:48:34.873+0000: 46: debug : virFileClose:111 : Closed fd 20
2020-07-23 06:48:34.873+0000: 46: debug : virFileClose:111 : Closed fd 23
2020-07-23 06:48:34.873+0000: 46: debug : virFileClose:111 : Closed fd 25
2020-07-23 06:48:34.873+0000: 46: debug : virCommandRunAsync:2588 : Command result 0, with PID 57
2020-07-23 06:48:34.960+0000: 46: debug : virCommandRun:2436 : Result exit status 0, stdout: '' stderr: '2020-07-23 06:48:34.873+0000: 57: debug : virFileClose:111 : Closed fd 23
2020-07-23 06:48:34.873+0000: 57: debug : virFileClose:111 : Closed fd 25
2020-07-23 06:48:34.873+0000: 57: debug : virFileClose:111 : Closed fd 20
2020-07-23 06:48:34.873+0000: 57: debug : virExecCommon:474 : Setting child uid:gid to 42427:42427 with caps 0
Could not access KVM kernel module: Permission denied
qemu-kvm: failed to initialize KVM: Permission denied
qemu-kvm: Back to tcg accelerator
```

```
vim /usr/lib/udev/rules.d/80-kvm.rules 
KERNEL=="kvm", GROUP="kvm", MODE="0666", OPTIONS+="static_node=kvm"

vim /lib/udev/rules.d/65-kvm.rules
KERNEL=="kvm", GROUP="kvm", MODE="0666"
```

## 6、修改/etc/kolla/globals.yml文件

```
cp -r /root/venv38/share/kolla-ansible/etc_examples/kolla/ /etc/
```

```
kolla_base_distro: "centos"
kolla_install_type: "source"
openstack_release: "train"
openstack_tag: "{{ openstack_release }}"
kolla_internal_vip_address: "192.168.0.34"
network_interface: "eno2"
neutron_external_interface: "eno3"
keepalived_virtual_router_id: "34"
enable_haproxy: "no"
enable_chrony: "no"
enable_cinder: "yes"
enable_cinder_backup: "no"
enable_cinder_backend_nfs: "yes"
enable_heat: "no"
enable_nova_ssh: "no"
external_ceph_cephx_enabled: "no"
glance_backend_file: "yes"
cinder_volume_group: "cinder-volumes"
nova_compute_virt_type: "kvm"
memcached_dimensions:
  ulimits:
    nofile:
      soft: 98304
      hard: 98304
```

> memecached 问题
>
> + exec /usr/bin/memcached -v -l 192.168.0.34 -p 11211 -c 5000 -U 0 -m 256
> failed to set rlimit for open files. Try starting as root or requesting smaller maxconns value.

```
vim /etc/kolla/globals.yml
memcached_dimensions:
  ulimits:
    nofile:
      soft: 98304
      hard: 98304
```

## 7、修改all-in-one文件

```
cp /root/venv38/share/kolla-ansible/ansible/inventory/* /root/
```

```
vim all-in-one
[control]
node1       ansible_connection=local ansible_python_interpreter=/root/venv38/bin/python3

[network]
node1       ansible_connection=local ansible_python_interpreter=/root/venv38/bin/python3

[compute]
node1       ansible_connection=local ansible_python_interpreter=/root/venv38/bin/python3

[storage]
node1       ansible_connection=local ansible_python_interpreter=/root/venv38/bin/python3

[monitoring]
node1       ansible_connection=local ansible_python_interpreter=/root/venv38/bin/python3

[deployment]
node1       ansible_connection=local ansible_python_interpreter=/root/venv38/bin/python3
```

## 8、安装

```
kolla-ansible -i /root/all-in-one deploy
```

## 9、卸载

```
kolla-ansible -i /root/all-in-one destroy --yes-i-really-really-mean-it
```

