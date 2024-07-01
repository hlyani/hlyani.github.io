# Kubeasz

[https://github.com/easzlab/kubeasz](https://github.com/easzlab/kubeasz)

# 一、准备

## 1.下载工具ezdown

```
export release=3.6.4
wget https://github.com/easzlab/kubeasz/releases/download/${release}/ezdown
chmod +x ./ezdown
```

## 2.下载kubeasz代码、二进制、默认容器镜像

```
# inside
./ezdown -D

# outside
./ezdown -D -m standard
```

## 2.1下载额外容器镜像（可选）

```
./ezdown -X flannel
./ezdown -X prometheus
./ezdown -X cilium
```

## 2.2下载离线系统包（yum/apt）（可选）

```
./ezdown -P
```

## 3.目录结构

* /etc/kubeasz 包含kubeasz版本为${release}的代码
* /etc/kubeasz/bin 包含k8s/etcd/docker/cni等二进制文件
* /etc/kubeasz/down 包含集群安装时需要的离线容器镜像
* /etc/kubeasz/down/packages 包含集群安装时需要的系统基础软件

# 二、安装

> 容器化运行kubeasz

```
./ezdown -S
```

> 使用默认配置安装aio集群

```
docker exec -it kubeasz ezctl start-aio
```

>  如果安装失败，查看日志排查后，重装

```
docker exec -it kubeasz ezctl setup default all
```

# 三、验证

```
source ~/.bashrc
kubectl version
kubectl get node
kubectl get pod -A
kubectl get svc -A
```

# 四、安装dashboard

## 1.部署

```
# ezctl 集成部署组件，xxxx 代表集群部署名
# dashboard 部署文件位于 /etc/kubeasz/clusters/xxxx/yml/dashboard/ 目录
./ezctl setup xxxx 07
```

## 2.验证

```
# 查看pod 运行状态
kubectl get pod -n kube-system | grep dashboard
dashboard-metrics-scraper-856586f554-l6bf4   1/1     Running   0          35m
kubernetes-dashboard-698d4c759b-67gzg        1/1     Running   0          35m

# 查看dashboard service
kubectl get svc -n kube-system|grep dashboard
kubernetes-dashboard   NodePort    10.68.219.38   <none>        443:24108/TCP                   53s

# 查看pod 运行日志
kubectl logs -n kube-system kubernetes-dashboard-698d4c759b-67gzg
```

## 3.登录

> 因为dashboard 作为k8s 原生UI，能够展示各种资源信息，甚至可以有修改、增加、删除权限，所以有必要对访问进行认证和控制，为演示方便这里使用 `https://NodeIP:NodePort` 方式访问 dashboard，支持两种登录方式：Kubeconfig、令牌(Token)

## Token令牌登录（admin）

```
# 获取 Bearer Token，找到输出中 ‘token:’ 开头的后面部分
$ kubectl describe -n kube-system secrets admin-user 
```

## Token令牌登录（只读）

```
# 获取 Bearer Token，找到输出中 ‘token:’ 开头的后面部分
$ kubectl describe -n kube-system secrets dashboard-read-user 
```

## Kubeconfig登录（admin）

> Admin kubeconfig文件默认位置：`/root/.kube/config`，该文件中默认没有token字段，使用Kubeconfig方式登录，还需要将token追加到该文件中，完整的文件格式如下：

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdxxxxxxxxxxxxxx
    server: https://192.168.0.127:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: admin
  name: kubernetes
current-context: kubernetes
kind: Config
preferences: {}
users:
- name: admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRxxxxxxxxxxx
    client-key-data: LS0tLS1CRUdJTxxxxxxxxxxxxxx
    token: eyJhbGcixxxxxxxxxxxxxxxx
```

# 五、清理

```
docker exec -it kubeasz ezctl destroy default
```

