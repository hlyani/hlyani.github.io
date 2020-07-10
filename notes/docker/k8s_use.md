# k8s 使用

## 一、探针（健康状态监测）

### 1、探针执行方式

* LivenessProbe： 判断容器是否存活 running状态，如果不健康kubelet就会杀掉pod，根据重启策略RestartPolicy进行相应的处理
* ReadinessProbe：判断容器是否处于可用Ready状态，达到ready状态表示pod可以接受请求，如果不健康，从service的后端endpoint列表中把pod隔离出去

> 两种探针探测失败的方式不同，一个是重启容器，一个是不提供服务

### 2、诊断的三种方式

* ExecAction: 在容器内执行指定命令。如果命令退出时返回码为0则认为诊断成功
* TCPSocketAction：对指定端口上的容器的IP地址进行TCP检查，如果端口打开，则诊断认为是成功的
* HTTPGetAction：对指定的端口和路径上的容器的IP地址执行HTTP Get请求，如果响应状态码大于等于200且小于400，则诊断被认为是成功的

### 3、Probe详细配置

* initialDelaySeconds: 容器启动后第一次执行探测需要等待多少秒
* periodSeconds：执行探测的频率。默认10秒，最小1秒
* timeoutSeconds：探测超时时间，默认1秒，最小1秒
* successThreshold：探测失败后，最少连续探测成功多少次才被认定为成功。默认1,对于liveness必须是1，最小值1
* failureThreshold：探测成功后，最少连续探测失败多少次才被认定为失败，默认3，最小1
* HTTP probe 中可以给httpGet设置其他配置项

### 4、httpget其他配置项

* host：连接的主机名，默认连接到pod的IP。你可能想在http header中设置"Host"而不是使用IP。
* scheme：连接使用的schema，默认HTTP。
* path: 访问的HTTP server的path。
* httpHeaders：自定义请求的header。HTTP运行重复的header。
* port：访问的容器的端口名字或者端口号。端口号必须介于1和65535之间。

### 5、示例

```
livenessProbe:
  exec:
    command: ["cat","/app/index.html"]


exec:
  command: ["cat","/app/index.html"]
```

```
livenessProbe:
  exec:
    command: ["cat", "/app/index.html"]
  # tcpSocket:
  #   port: http
  initialDelaySeconds: 30
  timeoutSeconds: 5
  failureThreshold: 6
readinessProbe:
  exec:
    command: ["cat", "/app/index.html"]
  # tcpSocket:
  #   port: http
  initialDelaySeconds: 5
  timeoutSeconds: 3
  periodSeconds: 5
```

## 二、亲和性

* In: label的值在某个列表中
* NotIn：label的值不在某个列表中
* Exists：某个label存在
* DoesNotExist：某个label不存在
* Gt：label的值大于某个值（字符串比较）
* Lt：label的值小于某个值（字符串比较）

> 如果nodeAffinity中nodeSelector有多个选项，节点满足任何一个条件即可；如果matchExpressions有多个选项，则节点必须同时满足这些选项才能运行pod 。

### 1、给节点打标签

```
kubectl label no node1 disktype=ssd

spec:
  nodeSelector:
    disktype: ssd1
```

```
kubectl label no node3 testtag-
kubectl label no node3 testtag=error
kubectl get no --show-labels
kubectl get no -l testtag=error
kubectl label no node1 testtag=node1
kubectl label no node2 testtag=node2
```

```
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: testtag
            operator: In
            values:
            - node1
            - node2 
```

```
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: testtag
            operator: NotIn
            values:
            - error
```

```
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchExpressions:
      - key: testtag
        operator: NotIn
        values:
        - error
```

## 三、命名空间相关

### 1、创建、切换命名空间

```
kubectl get ns
kubectl create namespace my-namespace
kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name-here>
kubectl config set-context kubernetes-admin@cluster.local --namespace=kube-system
kubectl config view | grep namespace:
```

### 2、namespace 一直 Teminating

```
kubectl delete ns test
kubectl edit ns test
删除 finalizers
```

## 四、维护节点

### 1、设置节点不可调度

```
kuberctl cordon node2
```

### 2、驱逐节点上的pod

```
kubectl drain node2 --delete-local-data --ignore-daemonsets --force
```

### 3、维护结束

```
kubectl uncordon node2
```

### 4、删除节点

```
kubectl cordon node2
kubectl drain node2 --delete-local-data --ignore-daemonsets --forece
kubectl delete node node2
```

> --delete-local-data  即使pod使用了emptyDir也删除
> --ignore-daemonsets  忽略deamonset控制器的pod，如果不忽略，deamonset控制器控制的pod被删除后可能马上又在此节点上启动起来,会成为死循环；
> --force  不加force参数只会删除该NODE上由ReplicationController, ReplicaSet, DaemonSet,StatefulSet or Job创建的Pod，加了后还会删除'裸奔的pod'(没有绑定到任何replication controller)

### 5、重新加入

```
[root@master] kubeadm token create --print-join-command
[root@node2] kubeadm join 172.27.9.131:6443 --token svrip0.lajrfl4jgal0ul6i     --discovery-token-ca-cert-hash sha256:5f656ae26b5e7d4641a979cbfdffeb7845cc5962bbfcd1d5435f00a25c02ea50 
```

## 五、常用helm源

```
helm repo add aliyuncs https://apphub.aliyuncs.com
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://kubernetes-charts.storage.googleapis.com
helm repo add local https://local.com/chartrepo/k8s
helm repo add az https://mirror.azure.cn/kubernetes/charts/
helm repo add goharbor https://helm.goharbor.io
helm repo add emqx https://repos.emqx.io/charts
```

```
helm repo add --ca-file /etc/docker/certs.d/local.com/ca.crt --username=admin --password=XXX local https://local.com/chartrepo/k8s
```

