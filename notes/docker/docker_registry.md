# docker registry 相关

## 一、docker registry

##### 1、拉取registry镜像

```
docker pull registry
```

##### 2、启动registry镜像

```
docker run -d --name registry -p 4000:5000 --restart=always -v /opt/registry/:/var/lib/registry/docker/registry registry:latest
```

##### 3、测试，拉取busybox镜像，push到registy中

```
docker pull busybox
docker tag busybox 127.0.0.1:4000/busybox
docker push 127.0.0.1:4000/busybox
```

##### 4、查看registry仓库

[registry api](https://docs.docker.com/registry/spec/api/#detail)

```
curl http://127.0.0.1:4000/v2/_catalog

curl http://127.0.0.1:4000/v2/contrail-agent-u14.04/tags/list

curl http://127.0.0.1:4000/v2/kolla/centos-source-nova-api/tags/list

curl http://127.0.0.1:4000/v2/flannel/manifests/latest
```

##### 5、配置docker registy

```
tee /etc/docker/daemon.json <<-'EOF'
{
  "insecure-registries": ["127.0.0.1:4000"]
}
EOF

systemctl daemon-reload
systemctl restart docker
```

##### 6、打包registry镜像文件

```
tar -zcvf  registry-20190618.tar.gz /opt/registry/
```

##### 7、再次使用

```
tar -zxvf registry-20190618.tar.gz -C /opt/

docker run -d --name registry -p 4000:5000 --restart=always -v /opt/registry/:/var/lib/registry/docker/registry registry:latest
```

## 二、docker harbor

[doker harbor](https://github.com/vmware/harbor/blob/master/docs/installation_guide.md)

##### 1、安装docker、docker-compose

```
yum -y install yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum -y install docker-ce
systemctl start docker
systemctl enable docker
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

##### 2、下载harbor-offline-installer-v1.5.2.tgz

```
tar -xvf harbor-offline-installer-v1.5.2.tgz
cd harbor
```

##### 3、编辑配置文件 harbor.cfg

```
hostname = 192.168.21.232
customize_crt = off
harbor_admin_password = admin
```

##### 4、安装

```
./install.sh
```

##### 5、使用

[use harbor](https://github.com/vmware/harbor/blob/master/docs/user_guide.md)

```
浏览器登陆 192.168.21.232

创建项目公开项目 test

客户端配置
tee /etc/docker/daemon.json <<-'EOF'
{
  "insecure-registries": ["192.168.21.232"]
}
EOF

systemctl daemon-reload
systemctl restart docker

push image 先 login
```

