# 创建网桥

##### 1、安装网桥相关依赖

```
yum -y install tunctl bridge-utils
```

##### 2、创建网桥配置文件

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

##### 3、修改原有网卡配置文件

```
cat <<EOF > /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE="Ethernet"
DEVICE="eth0"
ONBOOT="yes"
BRIDGE=br0           
EOF
```

##### 4、重启网络

```
systemctl restart network
```

##### 5、网桥显示

```
brctl show
```

