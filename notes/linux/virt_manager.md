# virt-manager 相关

# 一、CentOS

##### 1、安装相关软件

```
yum install -y virt-manager xorg-x11-xauth
```
```
yum install -y qemu-kvm qemu-img virt-manager libvirt libvirt-python libvirt-client virt-install virt-viewer bridge-utils
```

# 二、Ubuntu

##### 1、安装软件

```
apt install -y qemu qemu-kvm qemu-system-arm qemu-efi-aarch64 qemu-utils libvirt-daemon libvirt-clients bridge-utils virt-manager
```

##### 2、重启 libvirt

```
systemctl restart libvirtd
```

##### 3、配置网桥

```
vim /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    eno1:
      dhcp4: no
      dhcp6: no
      #      addresses:
      #      - 192.168.0.241/24
      #      gateway4: 192.168.0.1
      #      nameservers:
      #        addresses:
      #        - 114.114.114.114
    eno2:
      dhcp4: no
      dhcp6: no
  version: 2
  bridges:
    br0:
      interfaces: [eno1]
      addresses: [192.168.0.241/24]
      gateway4: 192.168.0.1
      nameservers:
        addresses: [114.114.114.114]
```

##### 4、重启网络、查看网桥状态

```
netplan apply
networkctl status br0
```

##### 5、使用

```
virt-manager
```

# 三、常见问题

##### 1.virt-manager方格乱码的问题

```
解决CentOS和Ubuntu下virt-manager方格乱码的问题，virt-manager显示乱码的解决方法，安装相应字体： 
ubuntu下：
apt install font-manager 
apt install fonts-arphic-ukai 
apt install ttf-wqy-zenhei xfonts-wqy ttf-wqy-microhei 
apt install fonts-cwtex-fs 
apt install ttf-hanazono 
apt install ttf-mscorefonts-installer 

CentOS下安装：
yum install -y dejavu-lgc-sans-fonts
```

##### 2.打印运行日志

```
virt-manager --no-fork
```

##### 3.virt-installERROR internal error process exited while connecting to monitor

```
ERROR internal error process exited while connecting to monitor: char device redirected to /dev/pts/2
kvm: -drive file=/home/muge0913/workstation/kvm/test.img,if=none,id=drive-ide0-0-0,format=raw: could not open disk image
/home/muge0913/workstation/kvm/test.img: Permission denied

解决:编辑/etc/libvirt/qemu.conf添加内容如下，这样root就有操作的权限了。
user = “root”
# The group for QEMU processes run by the system instance. It can be
# specified in a similar way to user.
group = “root”
# Whether libvirt should dynamically change file ownership
# to match the configured user/group above. Defaults to 1.
# Set to 0 to disable file ownership changes.
dynamic_ownership = 0

service libvirt-bin restart
```

##### 4、virt-manager 输入字符响应两次的问题

```
1、打开Xbrowser.exe之后，点击最上面的工具。
2、点击Xconfig开始。
3、右键点击Default Profile,选择属性
4、在打开的界面选择高级。取消XKEYBOARD的勾选。
```

##### 5、删除默认网桥 virbr0

```
virbr0
virsh 
net-list 
net-autostart --disable default
net-destroy default
systemctl restart libvirtd
```

