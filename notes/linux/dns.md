# 搭建本地 dns 服务器

# 一、简要配置

# 1.dns 节点(如192.168.0.127)

> 确保53端口不被占用

```
vim /etc/dnsmasq.conf
no-hosts
no-resolv
no-poll
neg-ttl=3600
min-cache-ttl=3600
dns-forward-max=10000
cache-size=100000
edns-packet-max=1232
log-facility=/var/log/dnsmasq.log
all-servers
server=114.114.114.114
server=8.8.8.8
address=/hl.cn/10.0.0.127
```

```
systemctl restart dnsmasq
```

> cache-size                  # 设置 DNS 缓存大小，增加缓存可以减少频繁的外部 DNS 查询，提升查询速度。
>
> min-cache-ttl             # 设置缓存条目的最小 TTL（生存时间），单位为秒。此项可减少频繁的 DNS 查询，提高缓存命中率。
>
> dns-forward-max     # 设置允许的最大 DNS 并发查询数量。默认值可能较低，增加该值可以提高 DNS 并发查询性能。
>
> edns-packet-max     # 限制 EDNS0 扩展 DNS 响应包的大小，避免某些网络环境中较大的 DNS 包导致丢包问题。优化 EDNS 数据包大小，减少因 MTU 问题导致的查询失败。
>
> neg-ttl=3600             # 对未能解析的请求结果进行缓存，减少重复解析开销。
>
> all-servers                 # 同时向所有配置的上游 DNS 服务器发出查询请求，哪个服务器先返回结果就使用哪个，减少查询延迟。
>
> strict-order               # 按顺序使用 resolv-file 中的上游 DNS 服务器
>
> no-poll                       # 禁止定期轮询 resolv-file 变化
>
> no-resolv                   # 禁止自动读取 /etc/resolv.conf（确保只使用 resolv-file）
>
> no-hosts                    # 禁用系统的 hosts 文件，避免不必要的加载
>
> log-async=10            # 异步写日志，10 行合并写入一次，减少 I/O 压力
>
> log-facility=/var/log/dnsmasq.log  # 日志输出路径，可按需调整

```
# 配置测试
dnsmasq --test
```

# 2.使用节点

> 永久修改，需修改网卡 dns

```
vim /etc/resolv.conf
nameserver 192.168.0.127
# 每次查询的超时时间为 1 秒
# 每个服务器的重试次数为 3 次
# 启用 DNS 轮询，按顺序使用多个 nameserver
options timeout:1 attempts:3 rotate
```

# 二、测试

```
yum install -y dnsperf
```

```
cat domains.txt 
hl.cn A
xxx.yyy A
```

```
dnsperf -s 192.168.0.127 -p 53 -d domains.txt -c 256 -l 10
```

# 三、其他

##### 1、安装

```
yum方式安装，如下：
yum -y install dnsmasq
dnsmasq -v

apt-get方式安装，如下：
apt-get -y install dnsmasq
dnsmasq -v
```
##### 2、编辑配置文件
```
vim /etc/dnsmasq.conf
resolv-file=/etc/resolv.dnsmasq.conf
strict-order

echo 'resolv-file=/etc/dnsmasq.d/resolv.dnsmasq.conf' >> /etc/dnsmasq.conf
echo 'addn-hosts=/etc/dnsmasq.d/dnsmasq.hosts' >> /etc/dnsmasq.conf
```
##### 3、添加开机启动
```
/etc/init.d/dnsmasq restart
```
##### 4、检查是否启动
```
netstat -tunlp|grep 53
```
##### 5、使用DNS加快解析速度。打开/etc/dnsmasq.conf文件，server=后面可以添加指定的DNS，例如不同的网站使用不同的DNS。
* 国内指定DNS
```
server=/cn/114.114.114.114
server=/taobao.com/114.114.114.114
server=/taobaocdn.com/114.114.114.114
```
*  指定DNS
```
server=/google.com/223.5.5.5
```
* 屏蔽网站/广告
```
vim /etc/dnsmasq.conf
address=/ad.youku.com/127.0.0.1
address=/ad.iqiyi.com/127.0.0.1
```
> 指定域名解析到特定的IP上。这个功能可以让你控制一些网站的访问，非法的DNS就经常把一些正规的网站解析到不正确IP上。
address=/olinux.org.cn/123.123.123.123

> 内网DNS(DNS劫持)。首先将局域网中的所有的设备的本地DNS设置为已经安装Dnsmasq的服务器IP地址。然后修改已经安装Dnsmasq的服务器Hosts文件：/etc/hosts，指定域名到特定的IP中。

##### 6、其他电脑配置dns，检查测试缓存
```
dig www.baidu.com
```

##### 7、Dnsmasq小结
		 1、Dnsmasq作为本地DNS服务器安装方便，操作简单，改动的地方也不是很多，如果用VPS来搭建本地DNS，响应的速度会更快，也更稳定。
		 2、Dnsmasq的功能强大，反DNS劫持、加快解析速度、屏蔽广告、控制内网DNS、强制域名跳转到特定IP上等这些功能在我们的实际的生活中都是很有用的。
