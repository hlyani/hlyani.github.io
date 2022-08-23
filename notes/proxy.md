# proxy 相关

# 一、linux 代理

```
HTTP_PROXY=192.168.0.127:1080 COMMAND
```

# 二、curl 使用代理

```
curl -O XX --socks5 192.168.0.127:1080
```

# 三、wget 使用代理

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

# 四、docker [使用代理]()

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

```
echo '{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}'  | tee /etc/docker/daemon.json

systemctl daemon-reload
systemctl restart docker
```

```
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/", "http://hub-mirror.c.163.com", "http://hub-mirror.c.163.com"],
  "insecure-registries": ["registry.docker-cn.com", "docker.mirrors.ustc.edu.cn"],
  "debug": true,
  "experimental": true
}
```

[dockerproxy.com](dockerproxy.com)

```
{ "registry-mirrors": [ "https://dockerproxy.com" ] }
```

```
Docker Hub 官方镜像代理
常规镜像代理
官方命令：docker pull stilleshan/frpc:latest
代理命令：docker pull dockerproxy.com/stilleshan/frpc:latest

根镜像代理
官方命令：docker pull nginx:latest
代理命令：docker pull dockerproxy.com/library/nginx:latest
```

```
GitHub Container Registry
常规镜像代理
官方命令：docker pull ghcr.io/username/image:tag
代理命令：docker pull ghcr.dockerproxy.com/username/image:tag
```

```
Google Container Registry
常规镜像代理
官方命令：docker pull gcr.io/username/image:tag
代理命令：docker pull gcr.dockerproxy.com/username/image:tag
```

```
Google Kubernetes
常规镜像代理
官方命令：docker pull k8s.gcr.io/username/image:tag
代理命令：docker pull k8s.dockerproxy.com/username/image:tag

根镜像代理
官方命令：docker pull k8s.gcr.io/coredns:1.6.5
代理命令：docker pull k8s.dockerproxy.com/coredns:1.6.5
```

```
Quay.io
常规镜像代理
官方命令：docker pull quay.io/username/image:tag
代理命令：docker pull quay.dockerproxy.com/username/image:tag
```

# 五、containerd使用代理

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

# 六、git 代理

```
git config --global http.proxy socks5://192.168.0.127:1080
git config --global https.proxy socks5://192.168.0.127:1080
```

```
git config --global url."https://github.com/".insteadOf https://github.com.cnpmjs.org
git config --global url."https://github.com/".insteadOf https://git.sdut.me/
```

[https://ghproxy.com/](https://ghproxy.com/)

```
git clone https://ghproxy.com/https://github.com/stilleshan/ServerStatus
```

```
wget https://ghproxy.com/https://github.com/stilleshan/ServerStatus/archive/master.zip
```

```
curl -O https://ghproxy.com/https://github.com/stilleshan/ServerStatus/archive/master.zip
```

# 七、linux 代理

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

# 八、apt 代理

```
vim /etc/apt/apt.conf
```

```
Acquire::http::proxy "http://192.168.0.127:1080";
Acquire::https::proxy "http://192.168.0.127:1080";
```

# 九、yum 代理

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

# 十、go 代理

```
# 启用 Go Modules 功能
go env -w GO111MODULE=on

# 配置 GOPROXY 环境变量
# go env -w GOPROXY=https://goproxy.io,direct
go env -w GOPROXY=https://goproxy.cn,direct
```

# 十一、GitHub

[提高访问 github](https://mp.weixin.qq.com/s?__biz=MzI0ODU0NDI1Mg==&mid=2247517171&idx=3&sn=e06c6ed3c104ea3d001883e8cb1dcd36&chksm=e99ded60deea6476ecb19a43a94b25e4a0d57e7ed9ac653ac8d9c33bae8e6836eba28251a0f1&scene=132#wechat_redirect)

> 以下是 GitHub 完全同步仓库

```
https://github.com.cnpmjs.org
https://hub.fastgit.org
```

