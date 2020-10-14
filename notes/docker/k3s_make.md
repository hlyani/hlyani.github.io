# k3s 源码编译

# 一、编译

## 1、安装  golang

```
wget https://golang.google.cn/dl/go1.15.2.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.15.2.linux-amd64.tar.gz
```

```
vim /etc/profile
export PATH=$PATH:/usr/local/go/bin
```

## 2、获取源码（v1.19.2+k3s1）

```
git clone -b v1.19.2+k3s1 --depth=1 https://github.com/rancher/k3s.git
```

> 注：
>
> --depth=1 用于指定克隆深度，为1即表示只克隆最近一次commit.
>
> -b ${branch} clone 某个分支

## 3、修改代理

```
cd ./k3s/
```

```
vim ./Dockerfile.dapper
ARG http_proxy=http://192.168.0.110:1080
ARG https_proxy=http://192.168.0.110:1080
ENV http_proxy=http://192.168.0.110:1080
ENV https_proxy=http://192.168.0.110:1080

curl -x 192.168.0.110:1080 ...
```

```
vim ./scripts/download
curl -x 192.168.0.110:1080
```

## 4、下载生成依赖

```
mkdir -p build/data && ./scripts/download && go generate
```

## 5、编译

```
SKIP_VALIDATE=true make
```

> 1、SKIP_VALIDATE=true，因为修改了源文件，不跳过检查会报 dirty 错误
>
> 2、如果修改了go.mod 文件，需要在make前先执行 go mod vendor && go mod tidy 

## 6、查看编译结果

> 编译成功会在当前目录生成 dist 文件夹
>
> k3s 为编译后的二进制文件
>
> k3s-airgap-images-amd64.tar为k3s相关镜像
>
> k3s-images.txt 为k3s相关镜像列表

```
ls ./dist/artifacts/
k3s  k3s-airgap-images-amd64.tar  k3s-images.txt
```

```
cat k3s-images.txt
docker.io/rancher/coredns-coredns:1.6.9
docker.io/rancher/klipper-helm:v0.3.0
docker.io/rancher/klipper-lb:v0.1.2
docker.io/rancher/library-busybox:1.31.1
docker.io/rancher/library-traefik:1.7.19
docker.io/rancher/local-path-provisioner:v0.0.14
docker.io/rancher/metrics-server:v0.3.6
docker.io/rancher/pause:3.1
```

# 二、运行验证

> 将编译成功后的 dist/artifacts/ 文件拷贝至相应节点，控制节点和计算节点

## 1、运行 k3s 控制节点（控制节点执行）

```
cd dist/artifacts/
```

```
./k3s server &
```

## 2、查看集群状态（控制节点执行）

```
./k3s kubectl get nodes
```

## 3、运行 k3s 计算节点（计算节点执行）

```
k3s agent --server https://myserver:6443 --token ${NODE_TOKEN}
```

> myserver 为控制节点地址
>
> token 为控制节点，cat /var/lib/rancher/k3s/server/node-token 中的内容

## 4、再次查看集群状态（控制节点执行）

```
./k3s kubectl get nodes
```

## 5、将 k3s 二进制执行文件放入环境变量中

```
cp dist/artifacts/k3s /usr/bin/
```

## 6、参考链接

[k3s build](https://github.com/rancher/k3s/blob/master/BUILDING.md)