# proxy 相关

# 一、curl 使用代理

```
curl -O XX --socks5 192.168.0.127:1080
```

# 二、wget 使用代理

##### 1、修改环境变量

```
export http_proxy=http://192.168.0.127:1080
```

##### 2、修改 /etc/wgetrc 或 ~/.wgetrc

```
#You can set the default proxies for Wget to use for http, https, and ftp.
# They will override the value in the environment.
https_proxy = http://192.168.0.127:1080/
http_proxy = http://192.168.0.127:1080/
ftp_proxy = http://192.168.0.127:1080/

# If you do not want to use proxy at all, set this to off.
use_proxy = on
```

##### 3、使用 -e

```
wget -e "http_proxy=http://192.168.0.127:1080"
```

##### 4、安装 tsocks

```
apt-get -y install tsocks
```

```
vim /etc/tsocks.conf
server = 192.168.0.127
server_type = 5
server_port = 1080
```

```
tsocks wget http://*
```

# 三、docker 使用代理

```
mkdir -p /etc/systemd/system/docker.service.d

echo '[Service]
Environment="HTTP_PROXY=http://192.168.0.127:1080"
Environment="HTTPS_PROXY=http://192.168.0.127:1080"
Environment="NO_PROXY=localhost,127.0.0.1"' | tee > /etc/systemd/system/docker.service.d/http-proxy.conf
```

```
systemctl daemon-reload
systemctl restart docker
```

# 四、containerd使用代理

```
mkdir -p /etc/systemd/system/containerd.service.d

echo '[Service]
Environment="HTTP_PROXY=http://192.168.0.127:1080"
Environment="HTTPS_PROXY=http://192.168.0.127:1080"
Environment="NO_PROXY=localhost,127.0.0.1"' | tee > /etc/systemd/system/containerd.service.d/http_proxy.conf
```

```
systemctl daemon-reload
systemctl restart containerd
```

# 五、git 代理

```
git config --global http.proxy socks5://192.168.0.127:1080
git config --global https.proxy socks5://192.168.0.127:1080
```

```
git config --global url."https://github.com/".insteadOf https://github.com.cnpmjs.org
git config --global url."https://github.com/".insteadOf https://git.sdut.me/
```

# 六、linux 代理

```
export http_proxy=socks5://192.168.0.127:1080
export https_proxy=socks5://192.168.0.127:1080
export ftp_proxy=http://192.168.0.127:1080
export no_proxy=localhost,127.0.0.1
```

```
# 强制终端中的 wget、curl 等都走 SOCKS5 代理
export ALL_PROXY=socks5://192.168.0.127:1080
```

# 七、apt 代理

```
vim /etc/apt/apt.conf
```

```
Acquire::http::proxy "http://192.168.0.127:1080";
Acquire::https::proxy "http://192.168.0.127:1080";
```

# 八、yum 代理

```
vim /etc/yum.conf
```

```
# proxy=http://192.168.0.127:1080
# proxy_username=username
# proxy_password=password

# proxy=http://username:password@proxy_ip:port/
proxy=http://192.168.0.127:1080/
```

