# 制作openstack镜像

## 一、制作windows镜像

##### 1、准备软件

* virt-manager

* virtio-win-0.1.164.iso

[virtio](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.164-1/virtio-win-0.1.164.iso)

* spice-guest-tools-latest.exe

[spice-guest-tools](https://www.spice-space.org/download/binaries/spice-guest-tools/spice-guest-tools-latest.exe)

* CloudbaseInitSetup_0_9_11_x64.msi

[cloudbase](https://cloudbase.it/cloudbase-init/)

##### 2、封装系统sysprep

```
C:\Windows\System32\Sysprep
```

##### 3、压缩镜像的稀疏文件。

```
bsdtar -zcvf win10.qcow2.tar.gz win10.qcow2
```

##### 4、解压镜像。

```
tar -xvSf win10.qcow2.tar.gz
```

##### 5、转换镜像格式为raw。

```
qemu-img convert  -O raw win10.qcow2 win10.raw
```

##### 6、上传镜像到glance。

```
glance image-create --progress --disk-format raw --container-format bare --name win10 --property hw_video_model=vga --property  os_type=windows --file win10.raw
```

```
openstack image create --container-format bare --disk-format raw --public --protected --file win10.raw win10
```

## 二、制作linux镜像

##### 1、安装python虚拟环境并进入。

```
yum install -y epel-releaze python36-virtualenv squashfs-tools

virtualenv -p python3.6 venv
source venv/bin/activate
```

##### 2、linux镜像通过diskimage-builder制作。

[diskimage-builder](https://docs.openstack.org/diskimage-builder/latest/)

```
pip install diskimage-builder
```

##### 3、制作ubuntu镜像

> ubuntu 国内源 sources.list.template

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ DIB_RELEASE main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ DIB_RELEASE main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ DIB_RELEASE-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ DIB_RELEASE-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ DIB_RELEASE-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ DIB_RELEASE-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ DIB_RELEASE-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ DIB_RELEASE-security main restricted universe multiverse

# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ DIB_RELEASE-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ DIB_RELEASE-proposed main restricted universe multiverse
```

> build-ubuntu.sh

```
#!/usr/bin/env bash

set -eux

export DIB_RELEASE=xenial
export DIB_CLOUD_IMAGES="https://mirrors.ustc.edu.cn/ubuntu-cloud-images/${DIB_RELEASE}/current"
export DIB_DISTRIBUTION_MIRROR="https://mirrors.tuna.tsinghua.edu.cn/ubuntu"
cp sources.list.template sources.list
sed -i "s/DIB_RELEASE/${DIB_RELEASE}/g" sources.list
export DIB_APT_SOURCES="$(pwd)/sources.list"
export DIB_CLOUD_INIT_DATASOURCES="OpenStack"  # FIXME: we have not use config drive?
#export DIB_CLOUD_INIT_ALLOW_SSH_PWAUTH="yes"
#export DIB_DEV_USER_USERNAME="ubuntu"
#export DIB_DEV_USER_PASSWORD="123456"  # FIXME: weak password
#export DIB_DEV_USER_PWDLESS_SUDO="yes"
#export DIB_DEV_USER_SHELL="/bin/bash"

rm -f /tmp/image.log

disk-image-create -x ubuntu vm apt-sources cloud-init-datasources cloud-init selinux-permissive -o ubuntu-server-16.04-x86_64-$(date +%Y%m%d).raw -t raw --checksum -x --logfile /tmp/image.log

#glance image-create --visibility public --property os_type=linux--file build-ubuntu-server-7.5-x86_64-20180920.raw --container-format bare --disk-format raw --name ubuntu-server-7.5-x86_64-20180920 --progress
```

```
bash build-ubuntu.sh
```

##### 4、制作centos镜像。

> 提前下载好官方镜像到当前目录
>
> 如：CentOS-7-x86_64-GenericCloud-1808.qcow2

[centos-7-1808](https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-1808.qcow2.xz)

```
wget https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-1808.qcow2.xz
xz -d CentOS-7-x86_64-GenericCloud-1808.qcow2.xz
```

> build-centos.sh

```
#!/usr/bin/env bash

set -eux

#export ELEMENTS_PATH=tripleo-image-elements/elements:heat-agents:appcenter-config/elements
#export DIB_RELEASE=xenial
export DIB_LOCAL_IMAGE="CentOS-7-x86_64-GenericCloud-1808.qcow2" 
export DIB_CLOUD_INIT_DATASOURCES="OpenStack"  # FIXME: we have not use config drive?
export DIB_DISTRIBUTION_MIRROR=http://mirrors.163.com/centos
#export DIB_CLOUD_INIT_ALLOW_SSH_PWAUTH="yes"
#export DIB_DEV_USER_USERNAME="centos"
#export DIB_DEV_USER_PASSWORD="123456"  # FIXME: weak password
#export DIB_DEV_USER_PWDLESS_SUDO="yes"
#export DIB_DEV_USER_SHELL="/bin/bash"

rm -f /tmp/image.log

disk-image-create -x centos7 vm cloud-init-datasources cloud-init selinux-permissive -o centos-server-7.5-x86_64-$(date +%Y%m%d).raw -t raw --checksum -x --logfile /tmp/image.log

#glance image-create --visibility public --property os_type=linux--file build-centos-server-7.5-x86_64-20180920.raw --container-format bare --disk-format raw --name centos-server-7.5-x86_64-20180920 --progress
```

```
bash build-centos.sh
```

## 三、使用qemu-img制作镜像

### 1）、制作centos

##### 1、创建磁盘等

```
# qemu-img create -f qcow2 /tmp/centos.qcow2 10G
# virt-install --virt-type kvm --name centos --ram 1024 \
  --disk /tmp/centos.qcow2,format=qcow2 \
  --network network=default \
  --graphics vnc,listen=0.0.0.0 --noautoconsole \
  --os-type=linux --os-variant=centos7.0 \
  --location=/data/isos/CentOS-7-x86_64-NetInstall-1611.iso
```

##### 2、安装ACPI、让hypervisor可以重启和关闭虚拟机

```
# yum install -y acpid
# systemctl enable acpid
```

##### 3、安装cloud-init

```
yum install -y cloud-init
```

```/etc/cloud/cloud.cfg
# The top level settings are used as module
# and system configuration.

# A set of users which may be applied and/or used by various modules
# when a 'default' entry is found it will reference the 'default_user'
# from the distro configuration specified below
users:
   - default

# If this is set, 'root' will not be able to ssh in and they 
# will get a message to login instead as the above $user (ubuntu)
disable_root: true

# This will cause the set+update hostname module to not operate (if true)
preserve_hostname: false

# Example datasource config
# datasource: 
#    Ec2: 
#      metadata_urls: [ 'blah.com' ]
#      timeout: 5 # (defaults to 50 seconds)
#      max_wait: 10 # (defaults to 120 seconds)

# The modules that run in the 'init' stage
cloud_init_modules:
 - migrator
 - seed_random
 - bootcmd
 - write-files
 - growpart
 - resizefs
 - set_hostname
 - update_hostname
 - update_etc_hosts
 - ca-certs
 - rsyslog
 - users-groups
 - ssh

# The modules that run in the 'config' stage
cloud_config_modules:
# Emit the cloud config ready event
# this can be used by upstart jobs for 'start on cloud-config'.
 - emit_upstart
 - disk_setup
 - mounts
 - ssh-import-id
 - locale
 - set-passwords
 - grub-dpkg
 - apt-pipelining
 - apt-configure
 - package-update-upgrade-install
 - landscape
 - timezone
 - salt-minion
 - mcollective
 - disable-ec2-metadata
 - runcmd
 - byobu

# The modules that run in the 'final' stage
cloud_final_modules:
 - rightscale_userdata
 - scripts-per-once
 - scripts-vendor
 - scripts-per-once
 - scripts-per-boot
 - scripts-per-instance
 - scripts-user
 - phone-home
 - final-message
 - power-state-change

# System and/or distro specific settings
# (not accessible to handlers/transforms)
system_info:
   # This will affect which distro class gets used
   distro: ubuntu
   # Default user name + that default users groups (if added/used)
   default_user:
     name: ubuntu
     lock_passwd: false
     plain_text_passwd: '123456'
     gecos: Ubuntu
     groups: [adm, audio, cdrom, dialout, dip, floppy, netdev, plugdev, sudo, video]
     sudo: ["ALL=(ALL) NOPASSWD:ALL"]
     shell: /bin/bash
   ssh_svcname: ssh
```

##### 3、安装cloud-utils-growpart来允许分区来调整大小

```
yum install -y cloud-utils-growpart
```

##### 4、为了虚拟机能进入metadata服务器，禁用默认的zeroconf route

```
# echo "NOZEROCONF=yes" >> /etc/sysconfig/network
```

##### 5、配置控制台，编辑/etc/default/grub中的GRUB_CMDLIME_LINUX选择。删除 rhgb quiet 并且添加 console=tty0 console=ttyS0,115200n8

```
...
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=cl/root rd.lvm.lv=cl/swap console=tty0 console=ttyS0,115200n8"
```

```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

##### 6、关机

```
poweroff
```

##### 7、清除MAC地址等详细信息

```
yum -y install libguestfs-tools
```

```
virt-sysprep -d centos
```

##### 8、取消libvirt domain 的定义

```
virsh undefine centos
```

### 2）、其他

```
apt-get install kvm

qemu-img create -f raw /root/ubuntu16.04.1.raw 20G

virt-install --virt-type kvm --name ubuntu16.04.1 --ram 10240 \
  --cdrom=/root/ubuntu-16.04.1-server-amd64.iso \
  --disk /root/ubuntu16.04.1.raw,format=raw \
  --network network=default \
  --graphics vnc,listen=0.0.0.0 --noautoconsole \
  --os-type=linux --os-variant=ubuntutrusty

virsh list --all

virsh vncdisplay ubuntu16.04.1

virsh start trusty --paused

virsh attach-disk --type cdrom --mode readonly ubuntu16.04.1 "" hdc

virsh resume ubuntu16.04.1

apt-get install cloud-init

dpkg-reconfigure cloud-init

/sbin/shutdown -h now

virt-sysprep -d ubuntu16.04.1

# rm -rf /etc/udev/rules.d/70-persistent-net.rules

GRUB_CMDLINE_LINUX_DEFAULT="text console=tty0 console=ttyS0,115200n8"

update-grub

openstack image create "cirros" --file cirros-0.3.4-x86_64-disk.img --disk-format qcow2 --container-format bare --public

qemu-img convert -f qcow2 -O raw precise-cloudimg.img precise-cloudimg.raw

# 其他

virt-install --virt-type kvm --name centos --ram 10240 \
  --disk /root/CentOS7.raw,format=raw \
  --network network=default \
  --graphics vnc,listen=0.0.0.0 --noautoconsole \
  --os-type=linux --os-variant=rhel7 \
  --location=/root/CentOS-7-x86_64-DVD-1511.iso

qemu-img create -f qcow2 win7.qcow2 20G

virt-install --connect qemu:///system \
  --name win7 --ram 10240 --vcpus 4 \
  --network network=default,model=virtio \
  --disk path=/root/win7/win7.qcow2,format=qcow2,device=disk,bus=virtio \
  --cdrom /root/win7/cn_windows_7_ultimate_x64_dvd_x15-66043.iso \
  --disk path=/root/win7/virtio-win-0.1.126.iso,device=cdrom \
  --vnc --os-type windows --os-variant win2k8
```

