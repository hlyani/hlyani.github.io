# 容器交叉编译

# 一、gcc-linaro

[gcc-linaro-7.5.0](https://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/aarch64-linux-gnu/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz)

## 1、dockerfile

```
From ubuntu:20.04

ENV PATH=$PATH:/root/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/root/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/lib

WORKDIR /root

RUN sed -i s@/archive.ubuntu.com/@/mirrors.ustc.edu.cn/@g /etc/apt/sources.list && \
    apt update && \
    DEBIAN_FRONTEND=noninteractive apt install -y dracut git make libncurses-dev libelf-dev bison flex libssl-dev bc && \
    git clone http://192.168.0.90/kdpf/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.git && \
    alias make='make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-'
```

## 2、make_kernel

```
#!/bin/bash

rm -rf out_put/*

docker run -it --rm -v $PWD/out:/out gcc-linaro:1.0.0 /bin/sh -c "sh << EOF
alias make='make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j20'
git clone -b build --depth=1 http://192.168.0.90/kdpf/kernel.git
cd kernel/linux-4.19.37-rt/
make && make modules_install
cd hacl && make && make install && cd ..
sync
cp -f arch/arm64/boot/Image.gz /out/vmlinuz-4.19.37-rt19.arm64
cp -f .config /out/config-4.19.37-rt19.arm64
cp -f System.map /out/System.map-4.19.37-rt19.arm64
rm -rf /out/4.19.37-rt19 && cp -r /lib/modules/4.19.37-rt19 /out/
rm -rf /out/kernel && cp -rf /root/kernel /out/
EOF"
mv out/vmlinuz-4.19.37-rt19.arm64 out_put/vmlinuz-4.19.37-rt19.arm64
mv out/config-4.19.37-rt19.arm64 out_put/config-4.19.37-rt19.arm64
mv out/System.map-4.19.37-rt19.arm64 out_put/System.map-4.19.37-rt19.arm64
cp -r out/4.19.37-rt19 out_put/4.19.37-rt19
mv out/kernel/linux-4.19.37-rt out/linux-4.19.37-rt_build
cd out && chmod +x make_mod_compile_built_for_lfs.sh && ./make_mod_compile_built_for_lfs.sh linux-4.19.37-rt_build && sync
mv linux-4.19.37-rt_build ../out_put/linux-4.19.37-rt_build
chmod +x make_initramfs.sh && ./make_initramfs.sh && rm -rf 4.19.37-rt19 && sync
mv initramfs_unpatch/initramfs-4.19.37-rt19.img.arm64 ../out_put/initramfs-4.19.37-rt19.img.arm64
cd ../out_put/
tar -zcf 4.19.37-rt19.arm64.gz 4.19.37-rt19 && rm -rf 4.19.37-rt19
#dracut --kver 4.19.37-rt19 --add-drivers 'nvme nvme_core'
#cp -r /boot/* /out
```

```
docker build -t gcc-linaro:1.0.0 .
```

# 二、multiarch

[https://github.com/multiarch/qemu-user-static](https://github.com/multiarch/qemu-user-static)

## 一、运行环境

```
#docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

docker run --rm --privileged multiarch/qemu-user-static --reset

docker cp $(docker create --rm multiarch/qemu-user-static):/usr/bin/qemu-aarch64-static ./
```

```
docker run -it --rm -v /root/qemu-aarch64-static:/usr/bin/qemu-aarch64-static ubuntu bash
```

## 二、多阶段构建

```
vim Dockerfile

FROM multiarch/qemu-user-static as qemu

FROM arm64v8/ubuntu:20.04

COPY --from=qemu /usr/bin/qemu-aarch64-static /usr/bin

RUN echo "while true;do echo 'hello';done" > /a.sh

CMD [ "bash", "/a.sh" ]
```

```
docker build . -t test:1.0.0 -f Dockerfile
```

## 三、其他

## 1、构建参数

> --squash，默认false。设置该选项，将新构建出的多个层压缩为一个新层，但是将无法在多个镜像之间共享新层；设置该选项，实际上是创建了新image，同时保留原有image。

> --force-rm，默认false。设置该选项，总是删除掉中间环节的容器

> --rm，默认--rm=true，即整个构建过程成功后删除中间环节的容器

> --compress，默认false。设置该选项，将使用 gzip 压缩构建的上下文

> --no-cache，默认false。设置该选项，将不使用Build Cache构建镜像

### 1)、去除没有用的层（实验性）

```
FROM multiarch/qemu-user-static as qemu

FROM arm64v8/ubuntu:20.04

COPY --from=qemu /usr/bin/qemu-aarch64-static /usr/bin

RUN echo "while true;do echo 'hello';done" > /a.sh && rm -rf /usr/bin/qemu-aarch64-static

CMD [ "bash", "/a.sh" ]
```

### 2)、运行 docker 实验功能

```
vim /etc/docker/daemon.json
{
    "experimental": true
}

systemctl daemon-reload
systemctl restart docker
```

### 3)、构建

```
docker build --squash . -t test:1.0.0 -f Dockerfile
```

## 2、.dockerignore

> 文件中指定在传递给 docker 引擎 时需要忽略掉的文件或文件夹

```
comment

#代表根目录（上下文环境目录中）中以abc开头的任意直接子目录或者直接子文件将被忽略
#如/abc  abc.txt
/abc*

#代表根目录（上下文环境目录中）中任意直接子目录中以abc开头的任意直接子目录或者直接子文件将被忽略
#如 /file/abc  /file/abc.txt
*/abc*

#代表根目录（上下文环境目录中）中任意两级目录下以abc开头的任意直接子目录或者直接子文件将被忽略
#如 /file1/file2/abc  /file1/file2/abc.txt
*/*/abc*

#排除根目录中的文件和目录，其名称是单字符扩展名temp。例如，/tempa与/tempb被排除在外。
temp?	

#Docker还支持一个**匹配任意数量目录（包括零）的特殊通配符字符串
**/abc*

#以!（感叹号）开头的行可用于对排除项进行例外处理,比如原本包含了README.md这个文件的过滤，但是加了如下一行后
#就不会再过滤README.md，依然会将其提交到守护进程。
!README.md

#异常规则的放置位置会影响行为
*.md
!README*.md
README-secret.md
#README-secret.md 仍然会被忽略
	
*.md
README-secret.md
!README*.md
#README-secret.md 不会被忽略

您甚至可以使用该.dockerignore文件来排除Dockerfile和.dockerignore文件。这些文件仍然发送到守护程序，因为它需要它们来完成它的工作。但是ADD和COPY命令不会将它们复制到图像中。
```

## 3、从 docker image 拷贝文件

```
docker run -v $PWD:/opt/mount --rm --entrypoint cp multiarch/qemu-user-static /usr/bin/qemu-aarch64-static /opt/mount/
```

```
docker cp $(docker create --rm multiarch/qemu-user-static):/usr/bin/qemu-aarch64-static ./
```

## 4、在 x86 运行 arm 容器

```
apt install qemu-user-static
docker run -it --rm -v /usr/bin/qemu-aarch64-static:/usr/bin/qemu-aarch64-static arm64v8/ubuntu:20.04 uname -m
```

