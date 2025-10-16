# proxy 相关

# 一、linux 代理

```
HTTP_PROXY=192.168.0.127:1080 COMMAND
```

```
export {all_proxy,https_proxy,http_proxy,ftp_proxy,HTTP_PROXY,HTTPS_PROXY,FTP_RPOXY,ALL_PROXY}=http://127.0.0.1:1080
export no_proxy="localhost, 127.0.0.1, ::1"
```

```
unset proxy all_proxy https_proxy http_proxy ftp_proxy HTTP_PROXY HTTPS_PROXY FTP_RPOXY ALL_PROXY no_proxy NO_PROXY
```

```
export proxy=127.0.0.1:33210
export http_proxy=http://$proxy
export HTTP_PROXY=$http_proxy
export https_proxy=$http_proxy
export HTTPS_PROXY=$http_proxy
export ftp_proxy=$http_proxy
export FTP_RPOXY=$http_proxy
export all_proxy=socks5://$proxy
export ALL_PROXY=socks5://$proxy
export no_proxy="localhost, 127.0.0.1, ::1"
export NO_PROXY="localhost, 127.0.0.1, ::1"
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

# 四、docker 使用代理

##### proxy

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

##### noohub.ru

```
mkdir -p /etc/dockersudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://docker.m.daocloud.io","https://huecker.io","https://dockerhub.timeweb.cloud","https://noohub.ru"]
}
EOF
```

##### 中科大

```
echo '{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}'  | tee /etc/docker/daemon.json

systemctl daemon-reload
systemctl restart docker
```

##### 其他镜像源

```
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/", "http://hub-mirror.c.163.com", "http://hub-mirror.c.163.com"],
  "insecure-registries": ["registry.docker-cn.com", "docker.mirrors.ustc.edu.cn"],
  "debug": true,
  "experimental": true
}
```

```
# 使用中科大镜像源 
docker pull docker.mirrors.ustc.edu.cn/library/mysql:5.7

# 使用 Azure 中国镜像源
docker pull dockerhub.azk8s.cn/library/mysql:5.7
```

##### dockerproxy

[dockerproxy.com](dockerproxy.com)

```
{ "registry-mirrors": [ "https://dockerproxy.com" ] }
```

##### ustc

[ustc](http://mirrors.ustc.edu.cn/help/dockerhub.html)

```
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}
```

##### Docker Hub

```
docker pull stilleshan/frpc:latest
docker pull nginx:latest
```

```
docker pull dockerproxy.com/stilleshan/frpc:latest
docker pull dockerproxy.com/library/nginx:latest
```

##### ghcr.io - GitHub Container Registry

```
docker pull ghcr.io/username/image:tag
```

```
docker pull ghcr.dockerproxy.com/username/image:tag
```

##### gcr.io - Google Container Registry

```
docker pull gcr.io/username/image:tag
```

```
docker pull gcr.dockerproxy.com/username/image:tag
docker pull gcr.mirrors.ustc.edu.cn/xxx/yyy:zzz
docker pull gcr.azk8s.cn/xxx/yyy:zzz
```

##### k8s.gcr.io/registry.k8s.io - Google Kubernetes

```
docker pull k8s.gcr.io/username/image:tag
docker pull registry.k8s.io/username/image:tag
```

```
docker pull k8s.dockerproxy.com/username/image:tag
```

##### quay.io

```
docker pull quay.io/username/image:tag
```

```
docker pull quay.dockerproxy.com/username/image:tag
docker pull quay.mirrors.ustc.edu.cn/xxx/yyy:zzz
docker pull quay.azk8s.cn/xxx/yyy:zzz
```

##### docker-wrapper

[docker_wrapper](https://github.com/silenceshell/docker_wrapper)

> 一个 Python 编写的工具脚本，可以替代系统的 Docker 命令，自动从 Azure 中国拉取镜像并自动 Tag 为目标镜像和删除 Azure 镜像。

```
git clone https://github.com/silenceshell/docker-wrapper.git
cp docker-wrapper/docker-wrapper.py /usr/local/bin/
```

```
docker-wrapper pull k8s.gcr.io/kube-apiserver:v1.14.1
```

##### azk8spull

[azk8spull](https://github.com/xuxinkun/littleTools#azk8spull)

> 一个 Shell 编写的脚本，这个脚本功能和 docker-wrapper 类似。同样可以自动从 Azure 中国拉取镜像并自动 Tag 为目标镜像和删除 Azure 镜像。

```
git clone https://github.com/xuxinkun/littleTools
cd littleTools
chmod +x install.sh
./install.sh
```

```
azk8spull quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.24.1
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



```
echo 'Acquire::http::Proxy "http://192.168.0.127:1080";' > /etc/apt/apt.conf.d/proxy.conf
echo 'Acquire::https::Proxy "http://192.168.0.127:1080";' >> /etc/apt/apt.conf.d/proxy.conf
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

export GOPROXY="https://athens.azurefd.net"
export GOPROXY="https://mirrors.tencent.com/go/"
export GOPROXY="https://mirrors.aliyun.com/goproxy/"
```

# 十一、nodejs/npm 代理

```
npm config set registry https://registry.npmmirror.com
npm config set cache /root/.npm
npm config set proxy http://192.168.0.127:1080
npm config set https-proxy http://192.168.0.127:1080
```

# 十二、GitHub

[提高访问 github](https://mp.weixin.qq.com/s?__biz=MzI0ODU0NDI1Mg==&mid=2247517171&idx=3&sn=e06c6ed3c104ea3d001883e8cb1dcd36&chksm=e99ded60deea6476ecb19a43a94b25e4a0d57e7ed9ac653ac8d9c33bae8e6836eba28251a0f1&scene=132#wechat_redirect)

> 以下是 GitHub 完全同步仓库

```
https://github.com.cnpmjs.org
https://hub.fastgit.org
```

# 十三、systemd

```
vim /etc/systemd/system.conf
[Manager]
DefaultEnvironment="http_proxy=http://proxy:port" "https_proxy=http://proxy:port"
```

