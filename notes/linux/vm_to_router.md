# 将虚拟机作为router

##### 1、开启 ipv4 转发

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

##### 2、配置 iptables

```
iptables -A FORWARD -o tun0 -i eth0 -s 192.168.0.0/24 -m conntrack --ctstate NEW -j ACCEPT

iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

iptables -A POSTROUTING -t nat -j MASQUERADE
```

##### 3、将 iptables 保存

```
iptables-save |tee /etc/iptables.sav
```

##### 4、配置该虚拟机的网关就可以使用