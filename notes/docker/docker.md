# docker 相关

## 一、常用

##### 1、安装docker

[aliyun 安装docker-ce](https://yq.aliyun.com/articles/110806)

[tsinghua 安装docker-ce](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)

```
curl -sSL https://get.docker.io | bash

or

curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

docker --version
docker info
```

##### 2、导入、导出镜像

```
docker save registry:latest > registry.tar.gz
docker save -o registry.tar.gz registry:latest

docker load < registry.tar.gz
docker load -i registry.tar.gz
```

##### 3、启动容器

```
docker run -itd --name mariadb --restart=always -v /opt/mysql:/etc/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=qwe mariadb

docker run --name mariadb -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -v /tmp/my.cnf:/etc/mysql/my.cnf -d mariadb
```

##### 4、使用dockerpy

```
>>> import docker
>>> client = docker.DockerClient(base_url='unix://var/run/docker.sock')
```

##### 5、docker update

```
docker update --restart=always wiki
docker update --cpu-shares 512 -m 300M abebf7571666 hopeful_morse
docker update --kernel-memory 80M test
```

##### 6、查看容器ip地址、id

```
docker inspect -f '\{\{.NetworkSettings.IPAddress\}\}' wiki
docker inspect -f '\{\{.Id\}\}' registry
docker inspect --format '\{\{.Id\}\}' registry
```

##### 7、查看全部容器id、占用空间

```
docker ps -qa
docker ps -as
```

##### 8、保存镜像

```
docker commit
docker commit -a "user" -m "commit info" [CONTAINER] [imageName]:[imageTag]
docker login --username=[userName] --password=[pwd] [registryURL]
docker tag [imageID] [remoteURL]:[imageTag]
docker push [remoteURL]:[imageTag]
docker pull [remoteURL]:[imageTag]
docker diff
```

##### 9、--restart

```
no – 默认值，如果容器挂掉不自动重启

on-failure – 当容器以非 0 码退出时重启容器,同时可接受一个可选的最大重启次数参数 (e.g. on-failure:10)

always – 不管退出码是多少都要重启
```

##### 10、资源限制

```
# 限制内存最大使用
-m 1024m --memory-swap=1024m
# 限制容器使用CPU
--cpuset-cpus="0,1"
```

##### 11、一个容器连接到另一个容器

```
docker run -i -t --name sonar -d -link mmysql:db  tpires/sonar-server sonar
```

##### 12、构建自己的镜像

```
docker build -t <镜像名> <Dockerfile路径>
docker build -t xx/gitlab .
```

##### 13、查看容器端口

```
docker port registry
```

##### 14、查看容器进程

```
docker top registry
```

##### 15、监控容器资源使用情况

```
docker stats
docker stats --no-stream
```

##### 16、批量删除名字包含"none"的镜像

```
docker rmi $(docker images | grep "none" | awk '{print $3}')
```

##### 17、查看可用命令

```
docker help
```

##### 18、login

```
docker login --username=yourhubusername --email=youremail@company.com
```

##### 19、删除已安装docker

```
yum list installed | grep docker
yum remove -y docker.x86_64
yum remove -y docker-client.x86_64
yum remove -y docker-common.x86_64
```

##### 20、配置国内docker源

```
vim /etc/docker/daemon.json
{"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]  }
systemctl restart docker
```

## 二、编写dockerfile的最佳实践

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