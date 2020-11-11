# harbor 相关

## 1、安装 docker

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

## 2、安装 docker-compose

```
curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

```
chmod +x /usr/local/bin/docker-compose
```

## 3、下载 harbor

[https://github.com/goharbor/harbor](https://github.com/goharbor/harbor)

```
wget https://github.com/goharbor/harbor/releases/download/v2.0.0-rc2/harbor-offline-installer-v2.0.0-rc2.tgz
```

## 4、生成证书

##### 1、生成私钥

```
openssl genrsa -out ca.key 4096
```

##### 2、生成证书

```
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Chengdu/L=Chengdu/O=example/OU=Personal/CN=local.com" \
 -key ca.key \
 -out ca.crt
```

## 5、生成 server 端证书

##### 1、生私钥

```
openssl genrsa -out local.com.key 4096
```

##### 2、生成证书请求文件

```
openssl req -sha512 -new \
    -subj "/C=CN/ST=Chengdu/L=Chengdu/O=example/OU=Personal/CN=local.com" \
    -key local.com.key \
    -out local.com.csr
```

##### 3、生成 x509 v3

```
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=local.com
DNS.2=local
DNS.3=hostname
EOF
```

##### 4、使用 v3.ext 文件生成 harbor 主机证书

```
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in local.com.csr \
    -out local.com.crt
```

##### 5、提供证书给harbor或docker

```
cp local.com.crt /data/cert/
cp local.com.key /data/cert/
```

##### 6、转换 *.crt 为 *.cert， 以给 docker 使用

```
openssl x509 -inform PEM -in local.com.crt -out local.com.cert
```

##### 7、创建文件和复制 server 端证书、key 、Ca文件到 harbor 主机上的 docker 的认证文件夹

```
mkdir -p /etc/docker/certs.d/local.com/
```

```
cp local.com.cert /etc/docker/certs.d/local.com/
cp local.com.key /etc/docker/certs.d/local.com/
cp ca.crt /etc/docker/certs.d/local.com/
```

##### 8、重启 docker

```
systemctl restart docker
```

## 6、准备和配置 harbor配置文件

```
tar -zxvf harbor-offline-installer-v2.0.0-rc2.tgz
```

```
cd harbor/
cp harbor.yml.tmpl harbor.yml
vim harbor.yml
```

```
hostname: local.com
https:
    certificate: /etc/docker/certs.d/local.com/local.com.cert
    private_key: /etc/docker/certs.d/local.com/local.com.key
data_volume: /data
```
>将prepare中的镜像goharbor/prepare:v2.0.0 改为 goharbor/prepare:v2.0.0-dev

```
./prepare
```

## 7、部署harbor

```
./install.sh --with-chartmuseum
```

## 8、重装（如需执行）

```
docker-compose down -v
docker-compose up -d
```

```
./install.sh --with-chartmuseum
```

## 9、使用

### 一、docker image

##### 1、将证书拷贝到需要使用harbor的主机

```
scp -r 127.0.0.1:/etc/docker/certs.d/local.com/ *.*.*.*:/etc/docker/certs.d/
```

##### 2、编辑 /etc/docker/daemon.json

```
{
  "insecure-registries": ["local.com"],
}
```

##### 3、重启 docker

```
systemctl restart docker
```

##### 4、配置 hosts 文件

```
127.0.0.1 local.com
```

##### 5、登录

```
docker login local.com
```

##### 6、使用

```
for i in `docker images|egrep -v '9001|none|REPOSITORY'|awk '{print $1":"$2}'`;do docker tag $i "local.com/stx/"$i;done
```

```
for i in `docker images|grep local.com|awk '{print $1":"$2}'`;do docker push $i;done
```

### 二、chart

##### 1、添加仓库

```
helm repo add --ca-file /etc/docker/certs.d/local.com/ca.crt --username=admin --password=qwe local https://local.com/chartrepo/k8s
```

##### 2、push

```
helm push --ca-file /etc/docker/certs.d/local.com/ca.crt nginx-6.0.1.tgz local
```

```
helm push grafana-0.0.2.tgz test --username hl --password xxx
```

##### 3、pull

```
helm pull local/nginx
```

##### 4、安装

```
helm install --ca-file=ca.crt --username=admin --password=Passw0rd --version 6.0.1 local/nginx
```

## 10、忘记密码

##### 1、运行harbor-db容器

```
docker exec -it harbor-db bash
```

##### 2、进入数据库并选择表

```
psql -U postgres -d registry
```

##### 3、查看用户信息

```
select * from harbor_user;
```

```
admin     |       | ce058f765a36af71a7c43d95710d6074 | system admin   | admin user     | f       |            | rdk5buqqqi975je9hcv6hv0040agvf3b | t             | 2020-07-01 09:48:17.048602 | 2020-07-01 09:48:20.272419 | sha256
```

##### 4、更新密码（当前harbor版本v2.0.0）

> 密码修改后为admin/qwe

```
update harbor_user set salt='rdk5buqqqi975je9hcv6hv0040agvf3b', password='ce058f765a36af71a7c43d95710d6074' where user_id = 1;
```

> 可删除data/database内所有内容，重新部署，获取harbor_user表的相关字段