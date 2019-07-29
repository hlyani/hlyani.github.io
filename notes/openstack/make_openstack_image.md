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

