# tcpdump 相关

# 一、基础概念

```
tcpdump [option] [proto] [direction] [type]
```

> * proto: ip, ip6, arp, rarp, atalk, aarp, decnet, sca, lat, mopdl,  moprc,  iso,  stp, ipx,  netbeui 
> * direction：src, dst
> * type：host, net, port, portrange

## 1、类型

* host 192.168.201.128
* net 128.3
* port 20
* portrange 6000-6008

## 2、目标

* src
* dst
* src or dst
* src and dst

## 3、协议

* tcp
* udp
* icmp

## 4、操作符

* and / &&
* or / || 
* not / !
* =
* ==
* !=

## 5、关键字接口

- if：表示网卡接口名、
- proc：表示进程名
- pid：表示进程 id
- svc：表示 service class
- dir：表示方向，in 和 out
- eproc：表示 effective process name
- epid：表示 effective process ID

```
tcpdump "( if=en0 and proc =nc ) || (if != en0 and dir=in)"
```

## 6、输出内容结构

```
21:26:49.013621 IP 172.20.20.1.15605 > 172.20.20.2.5920: Flags [P.], seq 49:97, ack 106048, win 4723, length 48
```

1. 时分秒毫秒 21:26:49.013621
2. 网络协议 IP
3. 发送方的ip地址+端口号，其中172.20.20.1是 ip，而15605 是端口号
4. 箭头 >， 表示数据流向
5. 接收方的ip地址+端口号，其中 172.20.20.2 是 ip，而5920 是端口号
6. 冒号
7. 数据包内容，包括Flags 标识符，seq 号，ack 号，win 窗口，数据长度 length，其中 [P.] 表示 PUSH 标志位为 1

## 7、Flags标识符

- `[S]` : SYN（开始连接）
- `[P]` : PSH（推送数据）
- `[F]` : FIN （结束连接）
- `[R]` : RST（重置连接）
- `[.]` : 没有 Flag，由于除了 SYN 包外所有的数据包都有ACK，所以一般这个标志也可表示 ACK



## 8、可选参数

### 1、设置不解析域名提升速度

- `-n`：不把ip转化成域名，直接显示  ip，避免执行 DNS lookups 的过程，速度会快很多
- `-nn`：不把协议和端口号转化成名字，速度也会快很多。
- `-N`：不打印出host 的域名部分.。比如,，如果设置了此选现，tcpdump 将会打印'nic' 而不是 'nic.ddn.mil'.

### 2、过滤结果输出到文件

```
tcpdump icmp -w icmp.pcap
```

### 3、从文件中读取数据包

```
tcpdump icmp -r all.pcap
```

### 4、详细输出

- `-v`：产生详细的输出. 比如包的TTL，id标识，数据包长度，以及IP包的一些选项。同时它还会打开一些附加的包完整性检测，比如对IP或ICMP包头部的校验和。
- `-vv`：产生比-v更详细的输出. 比如NFS回应包中的附加域将会被打印, SMB数据包也会被完全解码。
- `-vvv`：产生比-vv更详细的输出。比如 telent 时所使用的SB, SE 选项将会被打印, 如果telnet同时使用的是图形界面，其相应的图形选项将会以16进制的方式打印出来。

### 5、时间显示

- `-t`：在每行的输出中不输出时间
- `-tt`：在每行的输出中会输出时间戳
- `-ttt`：输出每两行打印的时间间隔(以毫秒为单位)
- `-tttt`：在每行打印的时间戳之前添加日期的打印（此种选项，输出的时间最直观）

### 6、显示数据包的头部

- `-x`：以16进制的形式打印每个包的头部数据（但不包括数据链路层的头部）
- `-xx`：以16进制的形式打印每个包的头部数据（包括数据链路层的头部）
- `-X`：以16进制和 ASCII码形式打印出每个包的数据(但不包括连接层的头部)，这在分析一些新协议的数据包很方便。
- `-XX`：以16进制和 ASCII码形式打印出每个包的数据(包括连接层的头部)，这在分析一些新协议的数据包很方便。

### 7、过滤特定流向的数据包

- `-Q`：选择是入方向还是出方向的数据包，可选项有：in, out, inout，也可以使用  --direction=[direction] 这种写法

### 8、对输出内容进行控制

- `-D` : 显示所有可用网络接口的列表
- `-e` : 每行的打印输出中将包括数据包的数据链路层头部信息
- `-E` : 揭秘IPSEC数据
- `-L` ：列出指定网络接口所支持的数据链路层的类型后退出
- `-Z`：后接用户名，在抓包时会受到权限的限制。如果以root用户启动tcpdump，tcpdump将会有超级用户权限。
- `-d`：打印出易读的包匹配码
- `-dd`：以C语言的形式打印出包匹配码.
- `-ddd`：以十进制数的形式打印出包匹配码

### 9、其他

- `-A`：以ASCII码方式显示每一个数据包(不显示链路层头部信息). 在抓取包含网页数据的数据包时, 可方便查看数据
- `-l` : 基于行的输出，便于你保存查看，或者交给其它工具分析
- `-q` : 简洁地打印输出。即打印很少的协议相关信息, 从而输出行都比较简短.
- `-c` : 捕获 count 个包 tcpdump 就退出
- `-s` :  tcpdump 默认只会截取前 `96` 字节的内容，要想截取所有的报文内容，可以使用 `-s number`， `number` 就是你要截取的报文字节数，如果是 0 的话，表示截取报文全部内容。
- `-S` : 使用绝对序列号，而不是相对序列号
- `-C`：file-size，tcpdump 在把原始数据包直接保存到文件中之前, 检查此文件大小是否超过file-size. 如果超过了, 将关闭此文件,另创一个文件继续用于原始数据包的记录. 新创建的文件名与-w 选项指定的文件名一致, 但文件名后多了一个数字.该数字会从1开始随着新创建文件的增多而增加. file-size的单位是百万字节(nt: 这里指1,000,000个字节,并非1,048,576个字节, 后者是以1024字节为1k, 1024k字节为1M计算所得, 即1M=1024 ＊ 1024 ＝ 1,048,576)
- `-F`：使用file 文件作为过滤条件表达式的输入, 此时命令行上的输入将被忽略.

# 二、示例

## 1、基于网卡、源IP地址过滤

```
tcpdump -i eth2 src 192.168.1.127
```

> 所有网卡可用 `-i any`

## 2、基于源、网段过滤

```
tcpdump net 192.168.1.0/24
```

```
tcpdump src net 192.168
```

## 3、基于源、端口过滤

```
tcpdump src port 80 or 8088
```

```
tcpdump src portrange 8000-8080
```

## 4、基于协议过滤

> ip, ip6, arp, rarp, atalk, aarp, decnet, sca, lat, mopdl,  moprc,  iso,  stp, ipx, netbeui

```
tcpdump icmp
```

## 5、基于IP协议的版本进行过滤

> 6 表示tcp在ip报文中的编号

ipv4

```
tcpdump 'ip proto tcp'
tcpdump ip proto 6
tcpdump 'ip protochain tcp'
tcpdump ip protochain 6
```

ipv6

```
tcpdump 'ip6 proto tcp'
tcpdump ip6 proto 6
tcpdump 'ip6 protochain tcp'
tcpdump ip6 protochain 6
```

## 6、基于包大小过滤

```
tcpdump less 32 
tcpdump greater 64 
tcpdump <= 128
```

## 7、基于mac地址过滤

```
tcpdump ether host [ehost]
tcpdump ether dst    [ehost]
tcpdump ether src    [ehost]
```

> ehost为记录在/etc/ethers里的name

## 8、过滤通过指定网关的数据包

```
tcpdump gateway [host]
```

## 9、过滤广播/多播数据包

```
tcpdump ether broadcast
tcpdump ether multicast
tcpdump ip broadcast
tcpdump ip multicast
tcpdump ip6 multicast
```

## 10、其他

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