# linux 网络配置

# 一、netplan

```
vim /etc/netplan/01-netcfg.yaml 

network:
    version: 2
    renderer: networkd
#    renderer: NetworkManager
    ethernets:
        enp0s3:
            dhcp4: no
    #        dhcp6: no
            addresses: [192.168.1.222/24]
            gateway4: 192.168.1.1
            nameservers:
                addresses: [114.114.114.114,8.8.8.8]
        enp4s0:
            dhcp4: true
```

```
netplan try
netplan apply
netplan --debug apply
netplan -d apply
networkctl status
```

```
bonds:
    bond0:
        dhcp4: yes
        interfaces:
            - enp3s0
            - enp4s0
        parameters:
            mode: active-backup
            primary: enp3s0
```

```
bridges:
    br0:
        dhcp4: yes
        interfaces:
            - enp3s0         
```

```
vlans:
    vdev:
        id: 101
        link: net1
        addresses:
            - 10.0.1.10/24
    vprod:
        id: 102
        link: net2
        addresses:
            - 10.0.2.10/24
    vtest:
        id: 103
        link: net3
        addresses:
            - 10.0.3.10/24
    vmgmt:
        id: 104
        link: net4
        addresses:
            - 10.0.4.10/24
```

# 二、network

```
vim /etc/network/interfaces

auto enp10s0
iface enp10s0 inet static
address 192.168.1.162
netmask 255.255.255.0
gateway 192.168.1.100
dns-nameservers 1.0.0.1,1.1.1.1

systemctl restart networking
```

```
vim /etc/sysconfig/network-scripts/ifcfg-ens33

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.1.22
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=114.114.114.114

systemctl restart network
```

# 三、systemd-networkd

> .network 文件，为匹配的设备提供一个网络配置
> .netdev 文件，为匹配的环境创建一个虚拟网络设备
> .link 文件，当网络设备出现时，udev 将查找第一个匹配的.link文件

> [Match] 
> Host= 主机名
> Virtualization= 检查是否运行于虚拟机中
>
> [NetDev] 
> Name= 接口名称。必须提供
> Kind= 例如：bridge, bond, vlan, veth, sit，等等。必须提供

```
cat 100-bind.network
[Match]
Name=eth0

[Network]
Bridge=br0

cat 100-br0.netdev
[NetDev]
Name=br0
Kind=bridge

cat 100-br0.network
[Match]
Name=br0

[Network]
Address=192.168.0.21/24
Gateway=192.168.0.1
DNS=114.114.114.114
```

```
/etc/systemd/network/MyDhcp.network
[DHCPv4]
UseHostname=false
```

```
/etc/systemd/network/MyDhcp.network
[Match]
Name=en*

[Network]
DHCP=ipv4
```

```
/etc/systemd/network/10-dhcp.network
[Match]
Name=enp*
[Network]
DHCP=yes

静态网络可以配置成
/etc/systemd/network/20-static.network
[Match]
Name=enp0s3
[Network]
Address=192.168.0.22/24
Gateway=192.168.0.1
DNS=192.168.0.1
```

```
systemctl restart systemd-networkd
```

