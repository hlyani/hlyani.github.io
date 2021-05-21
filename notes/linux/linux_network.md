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

