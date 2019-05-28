# 网络端口聚合绑定

##### 1、centos网卡绑定
```
/etc/sysconfig/network-scripts/ifcfg-eth0

TYPE=Ethernet
BOOTPROTO=none
NAME=eth0
DEVICE=eth0
ONBOOT=yes
MASTER=bond0
SLAVE=yes

/etc/sysconfig/network-scripts/ifcfg-eth1

TYPE=Ethernet
BOOTPROTO=none
NAME=eth1
DEVICE=eth1
ONBOOT=yes
MASTER=bond0
SLAVE=yes

/etc/sysconfig/network-scripts/ifcfg-bond0
TYPE=Bond
BOOTPROTO=none
NAME=bond0
DEVICE=bond0
ONBOOT=yes
BONDING_MASTER=yes
BONDING_OPTS="mode=1 miimon=100"
IPADDR=10.10.10.12
NETMASK=255.255.255.0
GATEWAY=10.10.10.1
DNS1=114.114.114.114

systemctl restart network
```

> 其他方式

```
配置 bond0 绑定模式
创建 /etc/modprobe.d/bonding.conf，加入以下内容
alias bond0 bonding
options bond0 miimon=100 mode=1

载入 bonding 模块，重启 network 服务
modprob bonding
systemctl restart network
```

##### 2、ubuntu网卡绑定
```
apt-get install ifenslave

vim /etc/module
加bonding

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp2s0f0
iface enp2s0f0 inet manual
bond-master bond0

auto enp2s0f1
iface enp2s0f1 inet manual
bond-master bond0

auto bond0
iface bond0 inet static
        address 172.20.31.1
        netmask 255.255.255.0
        network 172.20.31.0
        broadcast 172.20.31.255
        gateway 172.20.31.254
        dns-nameservers 61.139.2.69
        bond-mode 4
        bond-miimon 100
        bond-lacp-rate 1
        bond-slaves enp2s0f0 enp2s0f1

auto lo
iface lo inet loopback

# The primary network interface
#auto p1p1
#iface p1p1 inet manual

auto p1p2
iface p1p2 inet manual

#auto em1
#iface em1 inet manual
#up ip link set $IFACE promisc on 
#down ip link set $IFACE promisc off
#mtu 9000

auto p1p1
iface p1p1 inet static
up ip link set $IFACE promisc on
#post-up ifenslave bond0 p1p1 p1p2
down ip link set $IFACE promisc off
#post-down ifenslave -d bond0 p1p1 p1p2
address 172.18.20.1
netmask 255.255.255.0
gateway 172.18.20.254
dns-nameservers 61.139.2.69
```