# 容器交叉编译

[https://github.com/multiarch/qemu-user-static](https://github.com/multiarch/qemu-user-static)

# 一、运行环境

```
#docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

docker run --rm --privileged multiarch/qemu-user-static --reset
```

# 二、多阶段构建

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

# 三、其他

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

