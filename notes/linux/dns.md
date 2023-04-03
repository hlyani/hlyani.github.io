# 搭建本地 dns 服务器

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