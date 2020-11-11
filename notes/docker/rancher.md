# rancher 相关

# 一、运行

```
docker run -d --restart=unless-stopped \
  --net="host" \
  --restart always \
  -v /etc/docker/certs.d/local.com:/container/certs \
  -e SSL_CERT_DIR="/container/certs" \
  --name rancher \
  rancher/rancher:latest
```

# 二、编译rancher-ui

## 1、基础环境

```
npm config set registry=https://registry.npm.taobao.org
yarn config set registry https://registry.npm.taobao.org
npm install -g cnpm --registry=https://registry.npm.taobao.org
yum -y install gcc+ gcc-c++
```

## 2、更新nodejs

```
npm install -g npm@latest
npm install -g node-gyp
```

## 3、编译rancher-ui

```
git clone 'https://github.com/rancher/ui'
cd 'ui'
npm install
yarn upgrade
./scripts/update-dependencies
```

## 4、运行

```
yarn start
RANCHER="https://rancher-server" yarn start
RANCHER="https://192.168.0.10" yarn start
```

# 三、通过 helm 安装 rancher

## 1、创建命名空间

```
kubectl create namespace cattle-system
```

## 2、创建秘钥

```
kubectl -n cattle-system create secret tls tls-rancher-ingress --cert=/etc/docker/certs.d/local.com/local.com.cert --key=/etc/docker/certs.d/local.com/local.com.key
```

## 3、查看秘钥信息

```
kubectl -n cattle-system get secret
```

## 4、添加 rancher 仓库

```
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
```

> latest：最新版，建议在尝试新功能时使用。
> stable：稳定版，建议生产环境中使用。
> alpha：预览版，未来版本的实验性预览。

## 5、拉取 rancher chart 包

```
helm pull rancher-latest/rancher --untar
```

## 6、修改配置

```
vim rancher/value.yaml

hostname: dashboard
ingress:
	tls:
		source: secret
```

## 7、安装

```
helm -n cattle-system install rancher .
```

## 8、查看安装状态

```
kubectl -n cattle-system rollout status deploy/rancher
```

# 四、FAQ

## 1、如果harbor中应用更新相同版本，rancher会有cache，版本不会更新

```
docker exec -itu0 rancher bash
rm -rf /var/lib/rancher/management-state/catalog-cache
```

