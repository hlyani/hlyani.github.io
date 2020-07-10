# tcpdump 相关

## 一、基础概念

### 1、类型

* host 192.168.201.128
* net 128.3
* port 20
* portrange 6000-6008

### 2、目标

* src
* dst
* src or dst
* src and dst

### 3、协议

* tcp
* udp
* icmp

### 4、操作符

* and &&
* or || 
* not !

## 二、示例

```
tcpdump -i ens33 port 8080 and host node1
```

```
只抓192.168网段 10个包
tcpdump -i ens33 -c 10 net 192.168
```

```
tcpdump -c 5 -nn -i eth0 icmp and src 192.168.100.62
```

```
解析包数据
tcpdump -c 2 -q -XX -vvv -nn -i ens33 tcp dst port 22
```

```
tcpdump -i eth0 tcp port 80 and host 192.168.0.129
```