# 创建网桥

# 一、CentOS

## 1、安装网桥相关依赖

```
yum -y install tunctl bridge-utils
```

## 2、创建网桥配置文件

```
cat <<EOF > /etc/sysconfig/network-scripts/ifcfg-br0
TYPE="Bridge"
DEVICE="br0"
ONBOOT="yes"
BOOTPROTO="static"
IPADDR="192.168.1.10"
NETMASK="255.255.255.0"
GATEWAY="192.168.1.1"
EOF
```

## 3、修改原有网卡配置文件

```
cat <<EOF > /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE="Ethernet"
DEVICE="eth0"
ONBOOT="yes"
BRIDGE=br0           
EOF
```

## 4、重启网络

```
systemctl restart network
```

## 5、网桥显示

```
brctl show
```

# 二、Ubuntu

## 1、安装依赖

```
apt -y install tunctl bridge-utils
```

## 2、配置

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

## 3、应用

```
netplan apply
```

## 4、查看状态

```
networkctl status br0
```

