# proxychains-ng 代理工具

##### 1、安装git、gcc

```
yum -y install git gcc
```

##### 2、下载源码

```
git clone https://github.com/rofl0r/proxychains-ng
```

##### 3、生成配置文件

```
cd /root/proxychains-ng
./configure --prefix=/usr --sysconfdir=/etc
```

##### 4、编译

```
make
make install
make install-config
```

##### 5、修改配置文件 /etc/proxychains.conf

```
修改
socks4  127.0.0.1 9050
为
socks5  192.168.21.81 1080
```

##### 6、使用、测试

```
proxychains4 curl www.google.com
```

