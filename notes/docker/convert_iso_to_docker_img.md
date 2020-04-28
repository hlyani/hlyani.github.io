# 将ISO镜像转换为docker镜像

### 一、转换

##### 1、准备 iso

> 例： ubuntu-16.04.6-desktop-amd64.iso

##### 2、安装工具 squashfs-tools

```
yum install -y squashfs-tools
```

##### 3、创建两个目录

```
mkdir rootfs unquashfs
```

##### 4、挂载 iso

```
mount -o loop ubuntu-16.04.6-desktop-amd64.iso rootfs
```

##### 5、找到 filesystem.squashfs.file 文件路径

```
find . -type f | grep filesystem.squashfs
```

```
find . -type f | grep filesystem.squashfs
./rootfs/casper/filesystem.squashfs
./rootfs/casper/filesystem.squashfs.gpg
```

##### 6、使用 unsquashfs 解压 filesystem.squashfs 到 unsquashfs 文件夹

```
unsquashfs -f -d unsquashfs/ rootfs/casper/filesystem.squashfs
```

##### 7、压缩并导入到 docker

```
tar -C unsquashfs -c . | docker import - ubuntu/myimg
```

##### 8、查看

```
docker images|grep "ubuntu/mying"
```

### 二、安装桌面

##### 1、clone 以下项目

[https://github.com/hlyani/docker-ubuntu-desktop.git](https://github.com/hlyani/docker-ubuntu-desktop.git)

```
git clone https://github.com/hlyani/docker-ubuntu-desktop.git
```

##### 2、修改 dockerfile 文件，将镜像改为刚才构建好的

```
FROM ubuntu/myimg

ENV DEBIAN_FRONTEND noninteractive
ENV USER root

COPY sources.list /etc/apt/

RUN apt-get update && \
    apt-get install -y --no-install-recommends ubuntu-desktop && \
    apt-get install -y gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal && \
    apt-get install -y tightvncserver && \
    mkdir /root/.vnc

ADD xstartup /root/.vnc/xstartup
ADD passwd /root/.vnc/passwd

RUN chmod 600 /root/.vnc/passwd

CMD /usr/bin/vncserver :1 -geometry 1280x800 -depth 24 && tail -f /root/.vnc/*:1.log

EXPOSE 5901
```

##### 3、测试、运行构建好的镜像

> vnc://<host>:5901
>
> password: password

```
docker run -dp 5901:5901 queeno/ubuntu-desktop
```

