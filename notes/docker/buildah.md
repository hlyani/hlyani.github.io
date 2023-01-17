# Buildah

# 一、编译部署

##### 1、拉取代码

```
git clone https://gitlabwh.uniontech.com/wuhan/container/buildah
```

##### 2、安装依赖

```
apt install -y pkg-config libgpgme-dev libseccomp-dev libdevmapper-dev
```

```
go get github.com/sbinet/go-python
```

##### 3、构建

```
 cd buildah
 git checkout v1.18.0
 make
```

##### 4、复制配置文件

```
mkdir /etc/containers/
cp docs/samples/registries.conf /etc/containers/
cp tests/policy.json /etc/containers/

cp ./vendor/github.com/containers/storage/storage.conf /etc/containers/
```

##### 5、部署网络插件

```
git clone https://ghproxy.com/https://github.com/containernetworking/plugins
cd ./plugins; ./build_linux.sh 
mkdir -p /opt/cni/bin
install -v ./bin/* /opt/cni/bin
```

##### 6、安装buildah

```
make install
```

或

```
cp bin/buildah /usr/local/bin/buildah
chmod 755 /usr/local/bin/buildah
```

# 二、使用

##### 1、根据Dockerfile构建镜像

```
buildah bud -t test
```

##### 2、推送镜像

```
buildah push --tls-verify=false --creds admin:123 192.168.0.127:3000/test/test:latest
```

##### 3、挂载已有镜像并修改、重新生成镜像

```
buildah from localhost/test
buildah mount $mycontainer
  
buildah commit $mycontainer containers-storage:myecho2
```

##### 4、查看容器

> 1）在构建容器镜像是的零时容器
>
> 2）mount后的容器

```
buildah containers
```

##### 5、修改镜像

```
buildah copy myecho-working-container-2 newecho /usr/local/bin
buildah config --entrypoint "/bin/sh -c /usr/local/bin/newecho" myecho-working-container-2
buildah run myecho-working-container-2 -- sh -c '/usr/local/bin/newecho'
buildah commit myecho-working-container-2 containers-storage:mynewecho
```

##### 6、基于scratch构建

```
buildah from scratch
```

# 三、其他

```
install -D -m0755 bin/buildah $(DESTDIR)/$(BINDIR)/buildah
$(MAKE) -C docs install

#install -D -m0755 bin/buildah //usr/local/bin/buildah
#make -C docs install
```

