# ssr privoxy 相关

## 一、ssr代理服务
##### 1、下载
```
git clone https://github.com/SAMZONG/gfwlist2privoxy.git
cd gfwlist2privoxy/
cp ssr /usr/local/bin
chmod +x /usr/local/bin/ssr
```
##### 2、安装
```
ssr install
ssr config
```
##### 3、 配置文件路径 /usr/local/share/shadowsocksr/config.json
```
{
    "server": "0..0.0.0",	// ssr服务器ip
    "server_ipv6": "::",
    "server_port": 8080,	// ssr服务器端口
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "123456",		// 对应password
    "method": "none",			// 这里对应SSGlobal配置中的Encryption
    "protocol": "auth_chain_a",		//对应protocl
    "protocol_param": "",
    "obfs": "http_simple",		//对应obfs
    "obfs_param": "hello.world",	//对应obfs_param
    "speed_limit_per_con": 0,
    "speed_limit_per_user": 0,
    "additional_ports" : {}, // only works under multi-user mode
    "additional_ports_only" : false, // only works under multi-user mode
    "timeout": 120,
    "udp_timeout": 60,
    "dns_ipv6": false,
    "connect_verbose_info": 0,
    "redirect": "",
    "fast_open": false
}
```
##### 4、启动/关闭
```
ssr start
ssr stop
```
##### 5、卸载
```
ssr uninstall
```
##### 6、这里操作会删除/usr/local/share/shadowsocksr
以上，本地监听服务已经配置完成了，在填写的过程中，要注意你的本地监听地址和监听端口，默认是127.0.0.1:1080，如果你修改了设置，那么在后续配置中也要配合修改。

## 二、Privoxy 配置
##### 1、安装
```
CentOS
yum install -y epel-release privoxy

Ubuntu
apt install -y privoxy
```
##### 2、全局模式
```
代理模式同其他平台上方式，将所有http/https请求走代理服务，如果需要全局代理的话按照如下操作即可，如果要使用PAC模式，请跳过此部分。

# 添加本地ssr服务到配置文件
echo 'forward-socks5 / 127.0.0.1:1080 .' >> /etc/privoxy/config

# Privoxy 默认监听端口是是8118
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
export no_proxy=localhost

启动服务
systemctl start privoxy.service
```
##### 3、PAC模式
```
使用GFWList是由AutoProxy官方维护，由众多网民收集整理的一个中国大陆防火长城的屏蔽列表。
cd gfwlist2privoxy/
bash gfwlist2privoxy
proxy(socks5): 127.0.0.1:1080		# 注意，如果你修改了ssr本地监听端口是需要设置对应的
{+forward-override{forward-socks5 127.0.0.1:1080 .}}

cp -af gfw.action /etc/privoxy/
echo 'actionsfile gfw.action' >> /etc/privoxy/config

# Privoxy 默认监听端口是是8118
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
export no_proxy=localhost
```
##### 4、启动服务
```
systemctl start privoxy.service
```
##### 5、proxy 环境变量
```
# privoxy默认监听端口为8118
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
export no_proxy=localhost
# no_proxy是不经过privoxy代理的地址
# 只能填写具体的ip、域名后缀，多个条目之间使用','逗号隔开
# 比如: export no_proxy="localhost, 192.168.1.1, ip.cn, chinaz.com"
# 访问 localhost、192.168.1.1、ip.cn、*.ip.cn、chinaz.com、*.chinaz.com 将不使用代理
```
##### 6、代理测试
```
curl -sL www.google.com

# 获取当前 IP 地址
# 如果使用 privoxy 全局模式，则应该显示 ss 服务器的 IP
# 如果使用 privoxy gfwlist模式，则应该显示本地公网 IP
```
##### 7、管理脚本
```
在以上部署操作完成后，应该已经可以正常科学上网了，但是如果需要进行管理时，需要分别管理ssr和privoxy，为了方便管理，这里写了一个shell脚本方便管理: ssr_manager

vim ssr_manager

#!/bin/bash
case $1 in
	start)
		ssr start &> /var/log/ssr-local.log
		systemctl start privoxy.service
		export http_proxy=http://127.0.0.1:8118
		export https_proxy=http://127.0.0.1:8118
		export no_proxy="localhost, ip.cn, chinaz.com"
		;;
	stop)
		unset http_proxy https_proxy no_proxy
		systemctl stop privoxy.service
		ssr stop &> /var/log/ssr-log.log
		;;
	autostart)
		echo "ssr start" >> /etc/rc.local
		systemctl enable privoxy.service
		echo "http_proxy=http://127.0.0.1:8118" >> /etc/bashrc
		echo "https_proxy=http://127.0.0.1:8118" >> /etc/bashrc
		echo "no_proxy='localhost, ip.cn, chinaz.com'" >> /etc/bashrc
		;;
	*)
		echo "usage: source $0 start|stop|autostart"
		exit 1
		;;
esac
```
##### 8、使用
```
mv ssr_manager /usr/local/bin
chmod +x ssr_manager
```
##### 9、启动服务
```
ssr_manager start
```
##### 10、关闭服务
```
ssr_manager stop 
```
##### 11、 添加开机自启动
```
ssr_manager autostart
```
##### 12、参考链接
https://www.zfl9.com/ss-local.html