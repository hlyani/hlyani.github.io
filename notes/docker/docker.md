# docker 相关

# 一、常用

## 1、安装docker

[aliyun 安装docker-ce](https://yq.aliyun.com/articles/110806)

[tsinghua 安装docker-ce](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)

```
curl -sSL https://get.docker.io | bash

or

curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

docker --version
docker info
```

## 2、导入、导出镜像

```
docker save registry:latest > registry.tar.gz
docker save -o registry.tar.gz registry:latest

docker load < registry.tar.gz
docker load -i registry.tar.gz
```

## 3、启动容器

```
docker run -itd --name mariadb --restart=always -v /opt/mysql:/etc/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=qwe mariadb

docker run --name mariadb -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -v /tmp/my.cnf:/etc/mysql/my.cnf -d mariadb
```

## 4、使用dockerpy

```
>>> import docker
>>> client = docker.DockerClient(base_url='unix://var/run/docker.sock')
```

## 5、docker update

```
docker update --restart=always wiki
docker update --cpu-shares 512 -m 300M abebf7571666 hopeful_morse
docker update --kernel-memory 80M test
```

## 6、查看容器ip地址、id

```
docker inspect -f '\{\{.NetworkSettings.IPAddress\}\}' wiki
docker inspect -f '\{\{.Id\}\}' registry
docker inspect --format '\{\{.Id\}\}' registry
```

## 7、查看全部容器id、占用空间

```
docker ps -qa
docker ps -as
```

## 8、保存镜像

```
docker commit
docker commit -a "user" -m "commit info" [CONTAINER] [imageName]:[imageTag]
docker login --username=[userName] --password=[pwd] [registryURL]
docker tag [imageID] [remoteURL]:[imageTag]
docker push [remoteURL]:[imageTag]
docker pull [remoteURL]:[imageTag]
docker diff
```

## 9、--restart

```
no – 默认值，如果容器挂掉不自动重启

on-failure – 当容器以非 0 码退出时重启容器,同时可接受一个可选的最大重启次数参数 (e.g. on-failure:10)

always – 不管退出码是多少都要重启
```

## 10、资源限制

```
# 限制内存最大使用
-m 1024m --memory-swap=1024m
# 限制容器使用CPU
--cpuset-cpus="0,1"
```

## 11、一个容器连接到另一个容器

```
docker run -i -t --name sonar -d -link mmysql:db  tpires/sonar-server sonar
```

## 12、构建自己的镜像

```
docker build -t <镜像名> <Dockerfile路径>
docker build -t xx/gitlab .
```

## 13、查看容器端口

```
docker port registry
```

## 14、查看容器进程

```
docker top registry
```

## 15、监控容器资源使用情况

```
docker stats
docker stats --no-stream
```

## 16、批量删除名字包含"none"的镜像

```
docker rmi $(docker images | grep "none" | awk '{print $3}')
```

## 17、查看可用命令

```
docker help
```

## 18、login

```
docker login --username=yourhubusername --email=youremail@company.com
```

## 19、删除已安装docker

```
yum list installed | grep docker
yum remove -y docker.x86_64
yum remove -y docker-client.x86_64
yum remove -y docker-common.x86_64
```

## 20、配置国内docker源

```
vim /etc/docker/daemon.json
{"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]  }
systemctl restart docker
```

## 21、使用 --volumes-from 备份

```
docker run --rm --volumes-from gitlab -v /backup1:/backup2 ubuntu tar cvf /backup2/gitlab-etc.tar /etc/gitlab
```

## 22、清理

```
$ cat /usr/bin/prune_docker.sh
#!/bin/bash
docker container prune -f # 删除所有退出状态的容器
docker volume prune -f # 删除未被使用的数据卷
docker image prune -f # 删除 dangling 或所有未被使用的镜像

$ crontab -l
0 0 * * * /usr/bin/prune_docker.sh >> /var/log/prune_docker.log 2>&1
```

## 23、docker 代理

```
cat /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=http://proxy.server:port"
Environment="HTTPS_PROXY=http://proxy.server:port"
Environment="NO_PROXY=localhost,127.0.0.1"

systemctl daemon-reload
systemctl restart docker
```

## 24、使用本地仓库

```
vim /etc/docker/daemon.json

{
    "insecure-registries": ["192.168.0.11:30002"]
}

systemctl daemon-reload
systemctl restart docker
```

## 25、azk8s.cn 支持镜像转换列表

| global                                                       | proxy in China                                               | format                  | example                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------- | ------------------------------------------------------------ |
| [dockerhub](https://www.cnblogs.com/xuxinkun/p/hub.docker.com) (docker.io) | [dockerhub.azk8s.cn](http://mirror.azk8s.cn/help/docker-registry-proxy-cache.html) | `dockerhub.azk8s.cn//:` | `dockerhub.azk8s.cn/microsoft/azure-cli:2.0.61` `dockerhub.azk8s.cn/library/nginx:1.15` |
| gcr.io                                                       | [gcr.azk8s.cn](http://mirror.azk8s.cn/help/gcr-proxy-cache.html) | `gcr.azk8s.cn//:`       | `gcr.azk8s.cn/google_containers/hyperkube-amd64:v1.13.5`     |
| quay.io                                                      | [quay.azk8s.cn](http://mirror.azk8s.cn/help/quay-proxy-cache.html) | `quay.azk8s.cn//:`      | `quay.azk8s.cn/deis/go-dev:v1.10.0`                          |

## 26、docker in docker

```
docker run -itd --privileged=true -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/bin/docker --name centos centos
```

## 27、不裁剪输出

```
docker ps -a --no-trunc
```

## 28、创建docker网络并使用

> 备注：subnet指定一个网段， -o选项可以解决使用ifconfig命令看不到自己创建的网桥名字的问题

```
docker network create docker01 --subnet=10.10.10.0/24 -o com.docker.network.bridge.name=docker01
```

```
docker run -itd --net docker01 --ip 10.10.10.51  镜像名
```

# 二、linux实现docker资源隔离

Linux 提供的主要的 NameSpace

- Mount NameSpace- 用于隔离文件系统的挂载点
- UTS NameSpace- 用于隔离 HostName 和 DomianName
- IPC NameSpace- 用于隔离进程间通信
- PID NameSpace- 用于隔离进程 ID
- Network NameSpace- 用于隔离网络
- User NameSpace- 用于隔离用户和用户组 UID/GID

# 三、理解Docker容器和镜像

![理解Docker容器和镜像](../../imgs/docker_layer.jpg)

##### Image Definition

> 镜像（Image）就是一堆只读层（read-only layer）的统一视角，也许这个定义有些难以理解，下面的这张图能够帮助理解镜像的定义。

![深入理解Docker容器和镜像](../../imgs/image_definition.jpg)

> 从左边看到了多个只读层，它们重叠在一起。除了最下面一层，其它层都会有一个指针指向下一层。这些层是Docker内部的实现细节，并且能够在主机（运行Docker的机器）的文件系统上访问到。统一文件系统（union file system）技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统。就可以在图片的右边看到这个视角的形式。可以在主机文件系统上找到有关这些层的文件。需要注意的是，在一个运行中的容器内部，这些层是不可见的。在我的主机上，我发现它们存在于/var/lib/docker/aufs目录下。

```
sudo tree -L 1 /var/lib/docker//var/lib/docker/├── aufs├── containers├── graph├── init├── linkgraph.db├── repositories-aufs├── tmp├── trust└── volumes7 directories, 2 files
```

##### Container Definition

> 容器（container）的定义和镜像（image）几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的。

![深入理解Docker容器和镜像](../../imgs/container_definition.jpg)

> 可能会发现，容器的定义并没有提及容器是否在运行，没错，这是故意的。要点：容器 = 镜像 + 读写层。并且容器的定义并没有提及是否要运行容器。

##### Running Container Definition

> 一个运行态容器（running container）被定义为一个可读写的统一文件系统加上隔离的进程空间和包含其中的进程。下面这张图片展示了一个运行中的容器。

![深入理解Docker容器和镜像](../../imgs/runing_container.jpg)

> 正是文件系统隔离技术使得Docker成为了一个前途无量的技术。一个容器中的进程可能会对文件进行修改、删除、创建，这些改变都将作用于可读写层（read-write layer）。下面这张图展示了这个行为。

![深入理解Docker容器和镜像](../../imgs/change_container_layer.jpg)

> 可以通过运行以下命令来验证上面所说的：

```
docker run ubuntu touch happiness.txt
```

> 即便是这个ubuntu容器不再运行，依旧能够在主机的文件系统上找到这个新文件。

```
find / -name happiness.txt/var/lib/docker/aufs/diff/860a7b...889/happiness.txt
```

##### Image Layer Definition

> 为了将零星的数据整合起来，提出了镜像层（image layer）这个概念。下面的这张图描述了一个镜像层，通过图片能够发现一个层并不仅仅包含文件系统的改变，它还能包含了其他重要信息。

![深入理解Docker容器和镜像](../../imgs/image_layer_definition.jpg)

> 元数据（metadata）就是关于这个层的额外信息，它不仅能够让Docker获取运行和构建时的信息，还包括父层的层次信息。需要注意，只读层和读写层都包含元数据。

![深入理解Docker容器和镜像](../../imgs/rw_md.jpg)

> 除此之外，每一层都包括了一个指向父层的指针。如果一个层没有这个指针，说明它处于最底层。

![深入理解Docker容器和镜像](../../imgs/layer_point.jpg)

##### Metadata Location

> 我发现在我自己的主机上，镜像层（image layer）的元数据被保存在名为”json”的文件中，比如说：

```
/var/lib/docker/graph/e809f156dc985.../json
```

> e809f156dc985...就是这层的id

> 一个容器的元数据好像是被分成了很多文件，但或多或少能够在/var/lib/docker/containers/<id>目录下找到，<id>就是一个可读层的id。这个目录下的文件大多是运行时的数据，比如说网络，日志等等。

#### 全局理解（Tying It All Together)

> 现在，让结合上面提到的实现细节来理解Docker的命令。

```
docker create <image-id>
```

![深入理解Docker容器和镜像](../../imgs/create_image.jpg)

> docker create 命令为指定的镜像（image）添加了一个可读写层，构成了一个新的容器。注意，这个容器并没有运行。

![深入理解Docker容器和镜像](../../imgs/new_rw_layer.jpg)

```
docker start <container-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_start.jpg)

> Docker start命令为容器文件系统创建了一个进程隔离空间。注意，每一个容器只能够有一个进程隔离空间。

```
docker run <image-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_run.jpg)

> 看到这个命令，通常会有一个疑问：docker start 和 docker run命令有什么区别。

![深入理解Docker容器和镜像](../../imgs/diff_run_and_create.jpg)

> 从图片可以看出，docker run 命令先是利用镜像创建了一个容器，然后运行这个容器。这个命令非常的方便，并且隐藏了两个命令的细节，但从另一方面来看，这容易让用户产生误解。

```
docker ps
```

![深入理解Docker容器和镜像](../../imgs/docker_ps.jpg)

> docker ps 命令会列出所有运行中的容器。这隐藏了非运行态容器的存在，如果想要找出这些容器，我们需要使用下面这个命令。

```
docker ps –a
```

![深入理解Docker容器和镜像](../../imgs/docker_ps_a.jpg)

> docker ps –a命令会列出所有的容器，不管是运行的，还是停止的。

```
docker images
```

![深入理解Docker容器和镜像](../../imgs/docker_image.jpg)

> docker images命令会列出了所有顶层（top-level）镜像。实际上，在这里我们没有办法区分一个镜像和一个只读层，所以我们提出了top-level镜像。只有创建容器时使用的镜像或者是直接pull下来的镜像能被称为顶层（top-level）镜像，并且每一个顶层镜像下面都隐藏了多个镜像层。

```
docker images –a
```

![深入理解Docker容器和镜像](../../imgs/docker_image_a.jpg)

> docker images –a命令列出了所有的镜像，也可以说是列出了所有的可读层。如果想要查看某一个image-id下的所有层，可以使用docker history来查看。

```
docker stop <container-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_stop.jpg)

> docker stop命令会向运行中的容器发送一个SIGTERM的信号，然后停止所有的进程。

```
docker kill <container-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_kill.jpg)

> docker kill 命令向所有运行在容器中的进程发送了一个不友好的SIGKILL信号。

```
docker pause <container-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_pause.jpg)

> docker stop和docker kill命令会发送UNIX的信号给运行中的进程，docker pause命令则不一样，它利用了cgroups的特性将运行中的进程空间暂停。具体的内部原理可以在这里找到：https://www.kernel.org/doc/Doc ... m.txt，但是这种方式的不足之处在于发送一个SIGTSTP信号对于进程来说不够简单易懂，以至于不能够让所有进程暂停。

```
docker rm <container-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_rm.jpg)

> docker rm命令会移除构成容器的可读写层。注意，这个命令只能对非运行态容器执行。

```
docker rmi <image-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_rmi.jpg)

> docker rmi 命令会移除构成镜像的一个只读层。只能够使用docker rmi来移除最顶层（top level layer）（也可以说是镜像），也可以使用-f参数来强制删除中间的只读层。

```
docker commit <container-id>
```

![commit](../../imgs/docker_commit1.jpg)

![深入理解Docker容器和镜像](../../imgs/docker_commit.jpg)

> docker commit命令将容器的可读写层转换为一个只读层，这样就把一个容器转换成了不可变的镜像。

```
docker build
```

![深入理解Docker容器和镜像](../../imgs/docker_build.jpg)

> docker build命令非常有趣，它会反复的执行多个命令。

![深入理解Docker容器和镜像](../../imgs/docker_build1.jpg)

> 从上图可以看到，build命令根据Dockerfile文件中的FROM指令获取到镜像，然后重复地1）run（create和start）、2）修改、3）commit。在循环中的每一步都会生成一个新的层，因此许多新的层会被创建。

```
docker exec <running-container-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_exec.jpg)

> docker exec 命令会在运行中的容器执行一个新进程。

```
docker inspect <container-id> or <image-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_inspect.jpg)

> docker inspect命令会提取出容器或者镜像最顶层的元数据。

```
docker save <image-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_save.jpg)

> docker save命令会创建一个镜像的压缩文件，这个文件能够在另外一个主机的Docker上使用。和export命令不同，这个命令为每一个层都保存了它们的元数据。这个命令只能对镜像生效。

```
docker export <container-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_export.jpg)

> docker export命令创建一个tar文件，并且移除了元数据和不必要的层，将多个层整合成了一个层，只保存了当前统一视角看到的内容（译者注：expoxt后的容器再import到Docker中，通过docker images –tree命令只能看到一个镜像；而save后的镜像则不同，它能够看到这个镜像的历史镜像）。

```
docker history <image-id>
```

![深入理解Docker容器和镜像](../../imgs/docker_history.jpg)

> docker history命令递归地输出指定镜像的历史镜像。

# 四、编写dockerfile的最佳实践

##### 1、创建短暂的容器

> 你定义的图像Dockerfile应该生成尽可能短暂的容器。通过“短暂”，我们的意思是容器可以被停止和销毁，然后重建并用绝对最小的设置和配置替换。

##### 2、了解构建上下文

> 发出docker build命令时，当前工作目录称为*构建上下文*。默认情况下，假定Dockerfile位于此处，但您可以使用文件flag（-f）指定其他位置。无论Dockerfile实际存在的位置如何，当前目录中的所有文件和目录的递归内容都将作为构建上下文发送到Docker守护程序。

##### 3、通过stdin管道Dockerfile

> Docker能够通过stdin与本地或远程构建上下文管道Dockerfile来构建映像。 通过stdin管道Dockerfile对于执行一次性构建非常有用，无需将Dockerfile写入磁盘，或者在生成Dockerfile的情况下，并且之后不应该持久化。

##### 4、排除.dockerignore

> 要排除与构建无关的文件（不重构源存储库），请使用.dockerignore文件。此文件支持与.gitignore文件类似的排除模式。

##### 5、使用多阶段构建

> 多阶段构建允许您大幅减小最终image的大小，而无需减少中间层和文件的数量。

> 由于image是在构建过程的最后阶段构建的，因此可以通过利用构建缓存来最小化image层。

##### 6、避免安装不必要的包

> 为了降低复杂性，依赖性，文件大小和构建时间，请避免安装额外的或不必要的软件包，只是因为它们可能“很好”。

##### 7、解耦应用程序

> 每个容器应该只关注一个问题。将应用程序分离到多个容器中可以更容易地水平扩展和重用容器。

##### 8、最小化层数

> 只有说明RUN，COPY，ADD创建图层。其他指令创建临时中间图像，并不增加构建的大小。

> 在可能的情况下，使用多阶段构建，并仅将所需的工件复制到最终图像中。这允许您在中间构建阶段中包含工具和调试信息，而不会增加最终图像的大小。

##### 9、对多行参数进行排序

> 只要有可能，通过按字母顺序排序多行参数来缓解以后的更改。这有助于避免重复包并使列表更容易更新。这也使PR更容易阅读和审查。

##### 10、利用构建缓存

> 构建映像时，Docker会逐步Dockerfile执行您的指令， 按指定的顺序执行每个指令。在检查每条指令时，Docker会在其缓存中查找可以重用的现有image，而不是创建新的（重复）image。