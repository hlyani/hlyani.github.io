

# buildkit & nerdctl & here-document

# 一、部署

## 1、下载

```
curl -LO https://github.com/moby/buildkit/releases/download/v0.11.6/buildkit-v0.11.6.linux-amd64.tar.gz
```

## 2、安装

```
tar -zxvf buildkit-v0.11.6.linux-amd64.tar.gz 
cp bin/* /usr/bin/
```

## 3、启动

##### a、用参数启动

```
buildkitd &

buildkitd --oci-worker=false --containerd-worker=true & 
```

> 使用 --oci-worker=false --containerd-worker=true 参数,可以让buildkitd服务使用containerd后端

##### b、使用配置文件启动

[buildkitd.toml](https://github.com/moby/buildkit/blob/master/docs/buildkitd.toml.md)

```
mkdir /etc/buildkit/
```

```
cat > /etc/buildkit/buildkitd.toml << EOF
[worker.oci]
  enabled = false

[worker.containerd]
  enabled = true
  # namespace should be "k8s.io" for Kubernetes (including Rancher Desktop)
  namespace = "default"
  platforms = [ "linux/amd64", "linux/arm64" ]
  gc = true
  # gckeepstorage sets storage limit for default gc profile, in MB.
  gckeepstorage = 9000
  
 [registry."192.168.0.127:3000"]
  http = true
  insecure = true
EOF
```

> 配置使用containerd后端，禁用oic后端
>
> 配置命名空间default
>
> 配置平台amd64
>
> 配置垃圾回收空间限制

```
buildkitd --config /etc/buildkit/buildkitd.toml & 
```

##### c、用 systemd 启动

```
cat > /usr/lib/systemd/system/buildkitd.service <<EOF
[Unit]
Description=/usr/bin/buildkitd
ConditionPathExists=/usr/bin/buildkitd
After=containerd.service

[Service]
Type=simple
ExecStart=/usr/bin/buildkitd
User=root
Restart=on-failure
RestartSec=1500ms

[Install]
WantedBy=multi-user.target
EOF
```

```
cat > /usr/lib/systemd/system/buildkit.socket <<EOF
[Unit]
Description=BuildKit
Documentation=https://github.com/moby/buildkit

[Socket]
ListenStream=%t/buildkit/buildkitd.sock
SocketMode=0660

[Install]
WantedBy=sockets.target
EOF
```

```
systemctl daemon-reload && systemctl restart buildkitd && systemctl enable buildkitd
```

# 二、使用

## 1、基本构建

```
buildctl build --frontend=dockerfile.v0 --local context=. --local dockerfile=. --output type=image,name=tmp:1.0.0
```

> buildctl build --frontend=dockerfile.v0 --local context=. --local dockerfile=. --output type=image,name=tmp:1.0.0

> frontend：使用dockerfile作为前端，也可以使用gateway.v0（未测试）。

> local context： 指向当前目录，这是Dockerfile执行构建时的路径上下文,比如在从目录中拷贝文件到镜像里

> local dockerfile：指向当前目录，表示Dockerfile在此目录

> output 的 name： 表示构建的镜像名称

## 2、构建并推送

```
buildctl build --frontend dockerfile.v0 --opt target=foo --opt build-arg:foo=bar --local context=. --local dockerfile=. --output type=image,name=docker.io/username/image,push=true
```

## 3、导出到本地目录

```
buildctl build --frontend=dockerfile.v0 --local context=. --local dockerfile=. -o type=local,dest=./a
```

## 4、导出到本地tar包

```
buildctl build --frontend=dockerfile.v0 --local context=. --local dockerfile=. -o type=docker,dest=./a.tar
```

## 5、导出到docker

```
buildctl build --frontend=dockerfile.v0 --local context=. --local dockerfile=. -o type=docker,name=tmp:1.0.0 | docker load
```

## 6、导出为oci tar 包

```
buildctl build --frontend=dockerfile.v0 --local context=. --local dockerfile=. -o type=oci,name=./a.tar
```

## 7、构建多平台镜像

```
buildctl build --frontend=dockerfile.v0 --opt platform=linux/amd64,linux/arm64 --local context=. --local dockerfile=. --output type=image,name=tmp:1.0.0
```

## 8、查看缓存

```
buildctl du -v
```

## 9、清理缓存

```
buildctl prune
```

# 三、结合nerdctl

## 1、下载

```
curl -LO https://github.com/containerd/nerdctl/releases/download/v1.5.0/nerdctl-1.5.0-linux-amd64.tar.gz
```

```
tar -zxvf nerdctl-1.5.0-linux-amd64.tar.gz
```

## 2、自动补全

```
echo "source <(nerdctl completion bash)" >/etc/profile
source /etc/profile
```

## 3、配置文件

```
mkdir /etc/nerdctl

cat > /etc/nerdctl/nerdctl.toml <<EOF 
debug             = false
debug_full        = false
insecure_registry = true
address           = "unix:///run/containerd/containerd.sock"
namespace         = "k8s.io"
snapshotter       = "overlayfs"
cgroup_manager    = "cgroupfs"
hosts_dir         = ["/etc/containerd/certs.d"]
experimental      = true
EOF
```

## 4、基础使用

```
nerdctl ns ls
```

```
nerdctl image ls
```

```
nerdctl pull nginx:alpine
```

```
nerdctl system info
```

```
nerdctl system prune -h
```

```
nerdctl build -t hl:1.0.0 .
```

```
nerdctl login -u admin -p 123456 --insecure-registry 192.168.0.127:5000
```

```
nerdctl -n k8s.io push --insecure-registry 192.168.0.127:5000/tmp/test:1.0.0
```

```
nerdctl -n k8s.io --address /run/containerd/containerd.sock commit 008dcf8b52b3 192.168.0.127:5000/test:1.0.0
```

```
nerdctl -n k8s.io inspect --format '{{.State.Status}}' ID
```

```
nerdctl tag 192.168.0.127:5000/test:1.0.0 192.168.0.127:5000/test:2.0.0
```

```
docker save 192.168.0.127:5000/test:1.0.0 | nerdctl load
```

```
nerdctl images --format "{{.Repository}}:{{.Tag}}"
nerdctl inspect --format '{{.RepoTags}}' image_name
nerdctl inspect --format '{{.RepoTags}}' ubuntu
```

# 四、here-document

[here-documents](https://docs.docker.com/engine/reference/builder/#here-documents)

```
# syntax=docker/dockerfile:1
  
FROM debian

RUN <<EOT
mkdir /hl
touch /hl/yani
echo hl > /hl/yani
EOT
```

```
# syntax=docker/dockerfile:1
FROM debian
RUN <<EOT bash
  set -ex
  apt-get update
  apt-get install -y vim
EOT

# syntax=docker/dockerfile:1
FROM python:3.6
RUN <<EOT
#!/usr/bin/env python
print("hello world")
EOT
```

```
cat > demo.txt <<EOF
> 123
> asdb
> EOF
```

```
# syntax=docker/dockerfile:1
FROM alpine
COPY <<-EOT /app/script.sh
	echo hello ${FOO}
EOT
RUN FOO=abc ash /app/script.sh
```

# 五、构建

## 1、拉取代码

```
git clone https://github.com/moby/buildkit.git
```

# 2、安装 buildx 插件

```
wget https://github.com/docker/buildx/releases/download/v0.11.2/buildx-v0.11.2.linux-amd64
chmod a+x buildx-v0.11.2.linux-amd64
mkdir -p ~/.docker/cli-plugins
mv buildx-v0.11.2.linux-amd64 ~/.docker/cli-plugins/docker-buildx
```

## 3、编译

```
make
```

## 4、多架构构建

```
cat > /etc/buildkitd.toml << EOF
debug = true
root = "/var/lib/buildkit"

[registry."192.168.0.10:3000"]
http = true
insecure = true

[registry."docker.io"]
mirrors = ["192.168.0.10:3000"]
http = true
insecure = true
EOF
```

```
docker buildx ls
```

```
docker buildx create \
  --driver docker-container \
  --platform linux/arm64,linux/amd64 \
  --use \
  --bootstrap \
  --config /etc/buildkitd.toml \
  --name multi
```

```
docker buildx ls
NAME/NODE DRIVER/ENDPOINT             STATUS  BUILDKIT             PLATFORMS
multi *   docker-container   multi0  unix:///var/run/docker.sock running v0.12.4  linux/arm64*, linux/amd64*, linux/amd64/v2, linux/amd64/v3, linux/amd64/v4, linux/386
default   docker            default default                     running v0.11.6+0a15675913b7 linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/amd64/v4, linux/386
```

> 切换

```
docker buildx use <builder>
```

> 启动

```
docker buildx inspect --bootstrap multi
```

> 删除

```
docker buildx rm multi
```

> 构建

```
docker buildx build --platform linux/arm64,linux/amd64 --build-arg="REPO" -f Dockerfile.local --output=./dist .
```

> 使用配置

```
Makefile

.PHONY: cross
cross:
	$(BUILDX_CMD) bake binaries-cross
```

```
docker-bake.hcl

target "binaries-cross" {
  inherits = ["binaries"]
  output = [bindir("cross")]
  platforms = [
    "linux/amd64",
    "linux/arm64"
  ]
}
```

