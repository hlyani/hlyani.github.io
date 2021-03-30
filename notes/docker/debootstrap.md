# 容器基础镜像构建

> 构建基础的 rootfs —> 配置基础系统参数 —> 部署用户自定义软件 —> 清理系统 —> 打包为容器镜像 —> 测试镜像 —> 发布仓库

[https://docs.docker.com/develop/develop-images/baseimages/](https://docs.docker.com/develop/develop-images/baseimages/)

# 一、Ubuntu

## 1、安装 debootstrap

```
apt install -y debootstrap
```

## 2、创建 rootfs 存放位置

```
mkdir -p /opt/diros
cd /opt/diros
```

## 3、构建基础 Ubuntu 20.10 groovy 的 rootfs

```
debootstrap --verbose --arch=amd64 groovy /opt/diros http://mirrors.aliyun.com/ubuntu
#debootstrap --verbose --arch=amd64 bionic /opt/diros https://mirrors.tuna.tsinghua.edu.cn/ubuntu
```

## 4、配置基础系统参数

```
chroot /opt/diros/ /bin/bash
```

```
apt update
apt upgrade
apt -y install vim locales
```

```
dpkg-reconfigure locales
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

tee /etc/lsb-release <<-'EOF'
ID=Intewell
NAME="Intewell-Linux"
VERSION=""
VERSION_ID=
EOF
```

```
rm -rf /tmp/*
apt clean

exit
```

## 5、打包并创建 docker 镜像

```
tar -C /opt/diros/ -cv . | docker import - diros
```

```
#tar --numeric-owner --exclude=/proc --exclude=/sys --exclude=/diros.tar -cvf diros.tar /
tar --numeric-owner --exclude=/proc --exclude=/sys -cvf diros.tar /
cat diros.tar | docker import - diros

tar --numeric-owner --exclude=/proc --exclude=/sys -C / -cv . | docker import - diros
```

## 6、测试

```
docker run diros cat /etc/lsb-release
```

# 二、Centos

## 1、下载 moby

```
git clone https://github.com/moby/moby
```

```
#git checkout 20.10
cd moby/contrib/
```

## 2、构建

```
./mkimage-yum.sh centos
```

```
docker images
```

# 三、FROM scratch

```
mkdir /opt/tmp
cd /opt/tmp
```

```
vim Dockerfile

FROM scratch
ADD hello /
CMD ["/hello"]
```

```
wget https://raw.githubusercontent.com/docker-library/hello-world/master/hello.c
```

```
docker run --rm -it -v $PWD:/build ubuntu:16.04
container# apt-get update && apt-get install -y build-essential
container# cd /build
container# gcc -o hello -static -nostartfiles hello.c
```

```
docker build --tag hello .
```

```
docker run --rm hello
```

# 四、多阶段编译

```
vim Dockerfile

FROM ubuntu:16.04 AS buildstage
RUN apt update && apt install -y build-essential wget && \
    wget https://raw.githubusercontent.com/docker-library/hello-world/master/hello.c && \
    gcc -o hello -static -nostartfiles hello.c

FROM scratch
COPY --from=buildstage hello /
CMD ["/hello"]
```

```
docker build --tag hello .
```

# 五、buildah

[https://github.com/containers/buildah/blob/master/docs/buildah-push.md](https://github.com/containers/buildah/blob/master/docs/buildah-push.md)

```
apt install -y buildah
yum install -y buildah
```

```
buildah images
```

```
vim Dockerfile

FROM ubuntu:16.04 AS buildstage
RUN apt update && apt install -y build-essential wget && \
    wget https://raw.githubusercontent.com/docker-library/hello-world/master/hello.c && \
    gcc -o hello -static -nostartfiles hello.c

FROM scratch
COPY --from=buildstage hello /
CMD ["/hello"]
```

```
buildah bud -t test:latest ./
```

```
buildah push cbd1566b3bbb dir:./b
```

```
buildah push cbd1566b3bbb docker-archive:./b.tar
```

