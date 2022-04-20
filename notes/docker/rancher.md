# rancher 相关

# 一、编译rancher dashborad

##### 1、依赖安装

```
wget https://nodejs.org/dist/v16.14.2/node-v16.14.2-linux-x64.tar.xz
tar -vxf node-v16.14.2-linux-x64.tar.xz
cp -r node-v16.14.2-linux-x64 /usr/local/
cd node-v16.14.2-linux-x64/
./bin/node -v
v16.14.2

cd /usr/local/node-v16.14.2-linux-x64/
ln -s /usr/local/node-v16.14.2-linux-x64/bin/npm /usr/bin/
ln -s /usr/local/node-v16.14.2-linux-x64/bin/node /usr/bin/

npm config set registry=https://registry.npm.taobao.org
npm install -g cnpm --registry=https://registry.npm.taobao.org
npm install --global yarn
yarn config set registry https://registry.npm.taobao.org
```

> 可能存在preset-env报错问题

```
npm uninstall @babel/preset-env
npm install @babel/preset-env@7.12.13
```

##### 2、下载运行

> 依赖rancher server的运行，需提前部署

```
git clone https://github.com/rancher/dashboard.git
cd dashboard/

yarn install
API=https://192.168.0.31:30000 yarn dev
```

##### 3、容器开发

```
docker run -it --net=host --name dev -v /git/dashboard:/src rancher/dashboard:dev bash
```

##### 4、代码修改

```
vim pkg/settings/setting.go
UIIndex                           = NewSetting("ui-index", "/usr/share/rancher/ui/index.html")
UIDashboardIndex                  = NewSetting("ui-dashboard-index", "https://releases.rancher.com/dashboard/latest/index.html")
```

##### 5、静态编译

> 编译完成将生成文件，拷贝到http://192.168.0.31:81/dashboard/v2.6.3-beta1.tar.gz中

```
./scripts/build-embedded
ls dist
```

##### 6、编译rancher

> clone rancher 源码进行相应容器镜像编译

```
vim package/Dockerfile
curl -sL http://192.168.0.31:81/dashboard/v2.6.3-beta1.tar.gz | tar xvzf - --strip-components=2 && \
```

##### 7、重置dashboard密码

> https://github.com/rancher/rancher/issues/34920

```
kubectl -n cattle-system exec $(kubectl -n cattle-system get pods -l app=rancher | grep '1/1' | head -1 | awk '{ print $1 }') -- reset-password
```

# 二、部署运行rancher

##### 1、通过k8s部署

```
kubectl create namespace cattle-system

kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.7.1/cert-manager.crds.yaml
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.7.1

helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set bootstrapPassword=admin

helm install rancher . --namespace cattle-system

helm uninstall rancher --namespace cattle-system
```

> 修改rancher为NodePort

```
vim rancher/templates/service.yaml

apiVersion: v1
kind: Service
metadata:
  name: {{ template "rancher.fullname" . }}
  labels:
{{ include "rancher.labels" . | indent 4 }}
spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30090
  - name: https-internal
    port: 443
    protocol: TCP
    targetPort: 443
    nodePort: 30493
  selector:
    app: {{ template "rancher.fullname" . }}
```

##### 2、通过容器部署

```
docker run -d --restart=unless-stopped \
  --net="host" \
  --restart always \
  -v /etc/docker/certs.d/local.com:/container/certs \
  -e SSL_CERT_DIR="/container/certs" \
  --name rancher \
  rancher/rancher:latest
```

# 三、编译rancher-ui

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

# 四、通过 helm 安装 rancher

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

# 五、FAQ

## 1、如果harbor中应用更新相同版本，rancher会有cache，版本不会更新

```
docker exec -itu0 rancher bash
rm -rf /var/lib/rancher/management-state/catalog-cache
```

