# K8S 使用

# 一、常用操作

## 1、探针

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

## 2、亲和性、标签

* In: label的值在某个列表中
* NotIn：label的值不在某个列表中
* Exists：某个label存在
* DoesNotExist：某个label不存在
* Gt：label的值大于某个值（字符串比较）
* Lt：label的值小于某个值（字符串比较）

> 如果nodeAffinity中nodeSelector有多个选项，节点满足任何一个条件即可；
>
> 如果matchExpressions有多个选项，则节点必须同时满足这些选项才能运行pod 。

### 1.节点亲和性/反亲和性

```
NodeAffinity:                                        节点亲和性
    requiredDuringSchedulingIgnoredDuringExecution:  硬亲和性，必须部署在指定的节点上，或必须不部署在指定节点上
    preferredDuringSchedulingIgnoredDuringExecution: 软亲和性，尽量部署在满足条件的节点上，或者尽量不部署在被匹配的节点上
```

### 2.Pod亲和性/反亲和性

```
podAffinity:                                      Pod 亲和性
podAntiAffinity:                                  Pod 反亲和性
    requiredDuringSchedulingIgnoredDuringExecution:   将a应用和b应用部署在一起，或不部署在一起
        labelSelector
        	matchExpressions
        	matchLabels
    	namespaceSelector
    	namespaces
    	topologyKey
    preferredDuringSchedulingIgnoredDuringExecution:  尽量将a应用和b应用部署在一起，或不部署在一起
		podAffinityTerm
		weight                                         1-100
```

```
affinity:
podAntiAffinity:
  preferredDuringSchedulingIgnoredDuringExecution:
  - weight: 1
    podAffinityTerm:
      labelSelector:
        matchExpressions:
          - key: aaa
            operator: In
            values:
              - aaa
      topologyKey: "kubernetes.io/hostname"
```

```
affinity:
podAntiAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
          - key: aaa
            operator: In
            values:
              - bbb
      topologyKey: kubernetes.io/hostname
```

### 3.亲和性

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:       # 必须满足
        labelSelector:
        - matchExpressions:
          - key: disktype                                   # 标签 disktype
            operator: In                                    # 等于
            values:
            - ssd                                           # ssd
      preferredDuringSchedulingIgnoredDuringExecution:      # 尽量满足
      - weight: 1
        preference:
          matchExpressions:
          - key: processor                                  # labels key  
            operator: In                                    # 等于
            values:
            - gpu                                           # gpu
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

```
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: meme
            operator: In
            values:
            - bus  
        namespaces:              #如果写了namespaces但是留空，匹配所有namespace下的指定label的pod
          - kube-system
        topologyKey: kubernetes.io/hostname
```

### 4.反亲和性

```
spec:
  affinity:
    podAntiAffinity:                                      # 就这里加 Anti
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: meme
            operator: In
            values:
            - bus  
        topologyKey: kubernetes.io/hostname
```

### 5.示例

```
spec:
    template:
        metadata:
          labels:
            ddp: worker
    spec:
      affinity:
          podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                - key: ddp
                  operator: In
                  values:
                  - worker
              topologyKey: kubernetes.io/hostname
```

查看标签

```
kubectl get no --show-labels
kubectl get no -l testtag=error
kubectl get pods -l time=2019 --show-labels
```

设置标签

```
kubectl label nodes node2 node-role.kubernetes.io/worker=
kubectl label no node3 testtag=error
kubectl label no node1 testtag=node1
kubectl label node node2 role=worker
```

```
kubectl label no node1 disktype=ssd

spec:
  nodeSelector:
    disktype: ssd1
```

```
spec:
    nodeSelector:
        kubernetes.io/hostname: master.cluster.k8s
```

```
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        labelSelector:
        - matchExpressions:
          - key: testtag
            operator: In
            values:
            - node1
            - node2 
```

删除标签

```
kubectl label no node3 testtag-
```

## 3、集群资源查询

```
kubectl top
```

```
kubectl describe node <NODEMAME>
```

> Capacity（容量）: 指的是节点上理论上的最大资源量，即节点上未做任何预留，全部可用于运行Pods的最大资源总量。这通常反映了硬件的极限或管理员设定的上限。

> Allocatable（可分配）: 则是指在考虑了系统预留（system reserve）、kubelet预留以及其他系统组件（如kube-proxy、runtime等）所需资源后，真正可用于运行用户Pods的资源量。它是Kubernetes在调度Pod时实际参考的可用资源量，确保系统组件能够正常运行，防止资源被完全耗尽而导致节点不稳定。

> requests关注的是保障容器的最低资源需求，而limits则用来限制容器资源使用的上限

> 内存使用超出限制时可能会被Kubernetes OOM Killer（Out Of Memory killer）终止

> m 千分之一 毫核

```
kubectl proxy
http://127.0.0.1:8001/apis/metrics.k8s.io/v1beta1/nodes
http://127.0.0.1:8001/apis/metrics.k8s.io/v1beta1/nodes/<node-name>
http://127.0.0.1:8001/apis/metrics.k8s.io/v1beta1/pods
http://127.0.0.1:8001/apis/metrics.k8s.io/v1beta1/namespaces/<namespace-name>/pods/<pod-name>
```

```
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes/<node-name>
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/<namespace-name>/pods/<pod-name>
```

## 4、命名空间相关

### 1.创建、切换命名空间

```
kubectl get ns
kubectl create namespace my-namespace
kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name-here>
kubectl config set-context kubernetes-admin@cluster.local --namespace=kube-system
kubectl config view | grep namespace:
```

### 2.namespace 一直 Teminating

```
kubectl delete ns test
kubectl edit ns test
删除 finalizers
```

### 3.删除命名空间所有内容

```
kubectl delete pods --all -n test
```

```
kubectl api-resources --verbs=list --namespaced -o name|xargs -I {} kubectl delete --all {} -n test
```

## 5、维护节点

### 1.设置节点不可调度

```
kuberctl cordon node2
```

### 2.驱逐节点上的pod

```
kubectl drain node2 --delete-local-data --ignore-daemonsets --force
```

### 3.维护结束

```
kubectl uncordon node2
```

### 4.删除节点

```
kubectl cordon node2
kubectl drain node2 --delete-local-data --ignore-daemonsets --forece
kubectl delete node node2
```

> --delete-local-data  即使pod使用了emptyDir也删除
> --ignore-daemonsets  忽略deamonset控制器的pod，如果不忽略，deamonset控制器控制的pod被删除后可能马上又在此节点上启动起来,会成为死循环；
> --force  不加force参数只会删除该NODE上由ReplicationController, ReplicaSet, DaemonSet,StatefulSet or Job创建的Pod，加了后还会删除'裸奔的pod'(没有绑定到任何replication controller)

### 5.重新加入

```
[root@master] kubeadm token create --print-join-command
[root@node2] kubeadm join 172.27.9.131:6443 --token svrip0.lajrfl4jgal0ul6i     --discovery-token-ca-cert-hash sha256:5f656ae26b5e7d4641a979cbfdffeb7845cc5962bbfcd1d5435f00a25c02ea50 
```

## 6、annotation

修改annotation

```
kubectl annotate --overwrite pod test action=StopContainer
```

## 7、污点与容忍度

> 污点（Taint），排斥一类特定的Pod。
>
> 容忍度（Toleration），允许调度器调度带有对应污点的Pod，应用于Pod上。

* NoExecute

  这会影响已在节点上运行的 Pod，具体影响如下：

  * 如果 Pod 不能容忍这类污点，会马上被驱逐。
  * 果 Pod 能够容忍这类污点，但是在容忍度定义中没有指定 tolerationSeconds， 则 Pod 还会一直在这个节点上运行。
  * 如果 Pod 能够容忍这类污点，而且指定了 `tolerationSeconds`， 则 Pod 还能在这个节点上继续运行这个指定的时间长度。 这段时间过去后，节点生命周期控制器从节点驱除这些 Pod。

* NoSchedule

  除非具有匹配的容忍度规约，否则新的 Pod 不会被调度到带有污点的节点上。 当前正在节点上运行的 Pod 不会被驱逐。

* PreferNoScheduler

​	控制平面将尝试避免将不能容忍污点的 Pod 调度到的节点上，但不能保证完全避免。

```
kubectl taint nodes 192.168.0.127 key1=value1:NoExecute --overwrite
```

```
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
  tolerationSeconds: 6000
```

> operator的默认值是Equal
>
> 其他值：Exists
>
> tolerationSeconds是当pod需要被驱逐时，可以继续在node上运行的时间，可选参数。

删除

```
kubectl taint nodes 192.168.0.127 key1=value1:NoExecute-
```

> 允许在污点上调度

```
tolerations:
- key: "CriticalAddonsOnly"
  operator: "Exists"
- operator: "Exists"
  effect: "NoSchedule"
- operator: "Exists"
  effect: "NoExecute"
```

## 8、HostPath

| **挂载模式**          | **描述**                                                     |
| --------------------- | ------------------------------------------------------------ |
| **DirectoryOrCreate** | 如果在给定路径上什么都不存在，将根据需要创建空目录，权限设置为0755，与Kubelet具有相同的组和属主信息。 |
| **Directory**         | 在给定路径上必须存在目录。                                   |
| **FileOrCreate**      | 如果在给定路径上什么都不存在，那么将在给定路径根据需要创建空文件，权限设置为0644，具有与Kubelet相同的组和所有权。 |
| **File**              | 在给定路径上必须存在文件。                                   |

```
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - image: nginx:1.7.9
    name: test
    volumeMounts:
    - mountPath: /test
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      path: /data
      type: DirectoryOrCreate
```

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hostpath
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

## 9、获取 kubeconfig 并在 pod 中使用

```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: hpa-kubeconfig
  namespace: hpa
type: Opaque
data:
  config: $(kubectl config view --raw|base64|tr -d '\n')
EOF
```

```
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hpa-cronjob
  namespace: hpa
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hpa-cronjob
            image: kubectl:v1.26.8
            command:
            - "/bin/sh"
            - "-c"
            - "kubectl get no"
            volumeMounts:
            - name: kubeconfig
              mountPath: /root/.kube/config
              subPath: config
          volumes:
          - name: kubeconfig
            secret:
              secretName: hpa-kubeconfig
          restartPolicy: OnFailure
EOF
```

## 10、日志轮转

```
kubelet
--container-log-max-size=50Mi
--container-log-max-files=3
--container-log-dir=/path/to/custom/logs
```

```
/var/lib/kubelet/config.yaml
containerLogMaxFiles: 3 
containerLogMaxSize: 10Mi
```

```
containerd
[plugins."io.containerd.grpc.v1.cri".containerd]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    log-driver = "json-file"
    log-opts = ["max-size=50m", "max-file=3"]
```

## 11、配置变化重启pod

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "app.fullname" . }}
  labels:
    {{- include "app.labels" . | nindent 4 }}
    app.kubernetes.io/component: app
spec:
  replicas: 1
  selector:
    matchLabels:
      {{- include "app.selectorLabels" . | nindent 6 }}
      app.kubernetes.io/component: app
  template:
    metadata:
      labels:
        {{- include "app.labels" . | nindent 8 }}
        app.kubernetes.io/component: app
      annotations:
        configHash: {{ include (print $.Template.BasePath "/config.yaml") . | sha256sum | quote }}
        # configHash: {{ .Values.sagflow.config | toYaml | sha256sum | quote }}
```

> $.Template.BasePath "/config.yaml" -> templates/configmap.yaml

## 12、会话粘滞

```
spec:
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600  # 粘性时间为 1 小时
```

# 二、常用helm源

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

# 三、常用命令

## 1.jsonpath

```
kubectl get deploy -l app.kubernetes.io/name=controller -o jsonpath={.items[0].status.availableReplicas}

kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}"
```

## 2.强制删除容器

```
kubectl delete pod PODNAME --force --grace-period=0
```

```
helm uninstall --timeout 1s test & kubectl get po -o=name|grep test|xargs kubectl delete --grace-period=0 --force
```

## 3.从集群中导出配置

```
kubectl get daemonset -n kube-system kube-flannel-ds -o yaml > kube-system-kube-flannel-ds.yaml
```

## 4.测试 api sever

```
curl -kv https://10.96.0.1:443/version
```

## 5.集群组件状态

```
kubectl get --raw='/readyz?verbose'
```

## 6.events

```
kubectl get events --sort-by=.metadata.creationTimestamp --field-selector=involvedObject.kind=Pod,involvedObject.name=<pod-name>
```

```
kubectl get events --sort-by=.metadata.creationTimestamp --field-selector=involvedObject.kind=Pod,involvedObject.name=nginx-deployment-7bc4686759-m7h4l
```

```
kubectl get events
kubectl get events --field-selector type=Warning
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.kind!=Pod
kubectl get events --field-selector involvedObject.kind=Node, involvedObject.name=<node_name>
kubectl get events --field-selector type!=Normal
```

## 7.http 访问apiserver

```
kubectl proxy --address='0.0.0.0' --accept-hosts='^*$' --port=8080
```

## 8.logs

```
kubectl logs <pod_name>
kubectl logs --since=6h <pod_name>
kubectl logs --tail=50 <pod_name>
kubectl logs -f <service_name> [-c <$container>]
kubectl logs -f <pod_name>
kubectl logs -c <container_name> <pod_name>
kubectl logs <pod_name> pod.log
kubectl logs --previous <pod_name>
```

## 9.port-forward

```
kubectl port-forward $POD_NAME 8080:8080 -n default
kubectl port-forward svc/test 8080:31000
```

## 10.其他命令

```
docker inspect 5c3af3101afb -f "{{.HostConfig.Memory}}"

docker stats --format 'table {{.CPUPerc}}\t{{.MemPerc}}' --no-stream d17 2>/dev/null | tail -1 | sed 's/ //g'| sed 's/%/,/g'
```

```
kubectl run limit-test --image=busybox --limits "memory=100Mi" --command -- /bin/sh -c "while true; do sleep 2; done"

kubectl run limit-test --image=busybox --requests "cpu=50m" --command -- /bin/sh -c "while true; do sleep 2; done"
```

```
kubectl explain Pod.spec.volumes.hostPath
```

```
kubectl get pod test-pod -o jsonpath='{.spec.containers[*].resources.limits}'
```

```
kubectl expose pod httpbin --port 80
```

## 11.命令补全

```
kubectl completion bash
```

/root/.bashrc

```
source <(crictl completion bash)
source <(nerdctl completion bash)
source <(kubectl completion bash)
```

## 12.查看 gpu 使用情况

```
kubectl describe nodes | grep -E "Name:|nvidia.com/gpu"
```

```
kubectl get nodes -o custom-columns="NAME:.metadata.name,GPU_ALLOCATED:.status.allocatable.nvidia\.com/gpu,GPU_USED:.status.capacity.nvidia\.com/gpu"
```

## 13.pod 访问 svc

服务的规则如下

```
<service_name>.<namespace>.svc.cluster.local
```

```
<service_name>.<namespace>.svc.<后缀>
```

```
vim /var/lib/kubelet/config.yaml
...
clusterDomain: cluster.local
...
```

# 四、证书过期

```
kubeadm alpha certs renew all --config=/etc/kubernetes/kubeadm-config.yaml

kubeadm alpha certs check-expiration --config=/etc/kubernetes/kubeadm-config.yaml

cp -r /etc/kubernetes /etc/kubernetes_bak

cd /etc/kubernetes/pki/

rm -rf {apiserver.crt,apiserver-kubelet-client.crt,front-proxy-ca.crt,front-proxy-client.crt,front-proxy-client.key,front-proxy-ca.key,apiserver-kubelet-client.key,apiserver.key}

kubeadm init phase certs all --apiserver-advertise-address <IP>

cd /etc/kubernetes/

rm -rf {admin.conf,controller-manager.conf,kubelet.conf,scheduler.conf}

kubeadm init phase kubeconfig all

cp /etc/kubernetes/admin.conf $HOME/.kube/config
```

[https://github.com/hlyani/kubernetes1.17.3](https://github.com/hlyani/kubernetes1.17.3)

```
wget https://raw.githubusercontent.com/hlyani/kubernetes1.17.3/master/update-kubeadm-cert.sh
chmod +x update-kubeadm-cert.sh
./update-kubeadm-cert.sh all
./update-kubeadm-cert.sh master
```

# 五、修改 service 默认端口范围

> 默认范围 30000-32767

## 1、k8s

```
cat /etc/systemd/system/kube-apiserver.service | grep node-port

--service-node-port-range=80-32767 \
```

## 2、k3s

```
k3s server --kube-apiserver-arg --service-node-port-range=80-32767
```

```
cat /etc/systemd/system/k3s.service

ExecStart=/usr/local/bin/k3s \
    server --kube-apiserver-arg service-node-port-range=80-32767

systemctl daemon-reload
systemctl restart docker
```

```
cat /etc/systemd/system/k3s.service

ExecStart=/usr/local/bin/k3s \
    server --kube-apiserver-arg="service-node-port-range=80-32767"

systemctl daemon-reload
systemctl restart docker
```

# 六、Pod Pid限制

```
vim /var/lib/kubelet/config.yaml
podPidsLimit: 1024
```

```
/proc/sys/kernel/pid_max # 定义了可以分配给进程的最大进程 ID（PID） 

sysctl -w kernel.pid_max=65535

/etc/sysctl.conf
kernel.pid_max = 65535

sysctl -p
```

# 七、FAQ

## 1、flannel网络已存在

1. NetworkPlugin cni failed to set up pod "xxxxx" network: failed to set bridge addr: "cni0" already has an IP address different from10.x.x.x - Error

```
ip link set cni0 down
ip link set flannel.1 down 
ip link delete cni0
ip link delete flannel.1
systemctl restart containerd
systemctl restart kubelet
```

2. k8s cluster ping 10.96.0.1 no route

```
kubectl edit cm kube-proxy -n kube-system
...
    kind: KubeProxyConfiguration
    metricsBindAddress: ""
    mode: "ipvs"
...
```

3. kube-proxy Failed to retrieve node info: Unauthorized

> 报错日志来看是证书验证失败，github上看到了有此问题的解决方法 ，需要删除kube-proxy 依赖的secret
>
> 可能是多次运行kubeadm，导致集群里保存的证书和新生成的证书不一致

```
kubectl delete secret -n kube-system kube-proxy-token-kgrw7
```

4. k8s 修改 pod-network-cidr 地址范围

> --pod-network-cidr=10.244.0.0/16 -> --pod-network-cidr=192.168.0.0/16

```
kubectl -n kube-system edit cm kubeadm-config
vim /etc/kubernetes/manifests/kube-scheduler.yaml
```

```
kubectl cluster-info dump | grep -m 1 cluster-cidr
```

## 2、k8s证书过期，重新部署出问题，恢复数据和集群

1. 查看源pvc关系

```
[root@node1]# kubectl get pvc
NAME                                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-mariadb-master-0                    Bound    pvc-16840bfc-4a3b-45de-8250-3c71044c00ce   1000Gi     RWO            ceph-rbd       359d
data-mariadb-slave-0                     Bound    pvc-3a4b00ba-7fd9-4eb3-93ef-b4ea96648761   160Gi      RWO            ceph-rbd       359d
data-mariadb-slave-1                     Bound    pvc-12e8300a-3ef3-4ebe-a728-ec325666e675   160Gi      RWO            ceph-rbd       359d
data-mariadb-slave-2                     Bound    pvc-be01ef0f-51f9-4654-964b-288f74f5d43f   160Gi      RWO            ceph-rbd       359d
```

2. 查看每个节点的rbd挂载关系

```
[root@node1]# mount|grep rbd
/dev/rbd0 on /var/lib/kubelet/plugins/kubernetes.io/rbd/mounts/k8s-image-kubernetes-dynamic-pvc-d83535a2-774d-11ea-96f9-4a11faeddc0e type ext4 (rw,relatime,seclabel,stripe=1024,data=ordered)
/dev/rbd0 on /var/lib/kubelet/pods/6f364828-81bf-4bdc-9145-3d105f140932/volumes/kubernetes.io~rbd/pvc-c9b4b571-4c3c-46bf-8c67-1f1fc204dd6b type ext4 (rw,relatime,seclabel,stripe=1024,data=ordered)
/dev/rbd1 on /var/lib/kubelet/plugins/kubernetes.io/rbd/mounts/k8s-image-kubernetes-dynamic-pvc-d816cdfb-774d-11ea-96f9-4a11faeddc0e type ext4 (rw,relatime,seclabel,stripe=1024,data=ordered)
/dev/rbd1 on /var/lib/kubelet/pods/f1baca67-5ae2-4916-8e67-c9a37a51b94b/volumes/kubernetes.io~rbd/pvc-292d52f7-22b3-4af9-9633-94e09a0f8bef type ext4 (rw,relatime,seclabel,stripe=1024,data=ordered)
/dev/rbd8 on /var/lib/kubelet/plugins/kubernetes.io/rbd/mounts/k8s-image-kubernetes-dynamic-pvc-fc5a0ab3-220b-11ea-8d64-8e19e7bc2052 type ext4 (rw,relatime,seclabel,stripe=1024,data=ordered)
/dev/rbd8 on /var/lib/kubelet/pods/45de1bb4-5863-4dac-93f1-fd801a5a2f4d/volumes/kubernetes.io~rbd/pvc-16840bfc-4a3b-45de-8250-3c71044c00ce type ext4 (rw,relatime,seclabel,stripe=1024,data=ordered)
```

```
kubectl get pv pvc-XXXXXXXXXXXXXXXXXXXXXXXXX -o yaml
imageName: csi-vol-YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY
```

3. 查看pv pool 里面的rbd

```
rbd ls k8s
kubernetes-dynamic-pvc-045ec64a-2092-11ea-8d64-8e19e7bc2052
kubernetes-dynamic-pvc-0eef530b-206b-11ea-8d64-8e19e7bc2052
```

4. 重新创建应用，删除应用，查看当前应用的pvc，将pvc对应的rbd删除，并将旧的rbd重命名为当前pvc，最后再重新创建应用

- remove (rm)
- rename (mv)
- copy (cp)

```
rbd rm k8s/kubernetes-dynamic-pvc-d95e5e92-3c7e-11eb-9d5a-6a0e4650a17b
rbd mv k8s/kubernetes-dynamic-pvc-fc5a0ab3-220b-11ea-8d64-8e19e7bc2052 k8s/kubernetes-dynamic-pvc-d95e5e92-3c7e-11eb-9d5a-6a0e4650a17b
```

5. 其他

```
ceph osd dump | grep full_ratio
ceph osd set-full-ratio 0.98
ceph osd set-backfillfull-ratio 0.95
```

## 3、记一次 kube-flannel pod启动异常

> 第一现象，kube-flannel一直启不起来，不断重启

```
kubectl get po -A
```

> 查看pod日志，显示连接不上10.96.0.1:443，即kube-api 服务地址

```
kubectl logs -n kube-flannel kube-flannel-ds-7bcbn
```

```
kubectl get svc -A

NAMESPACE     NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP                  5d7h
```

> 尝试去各个节点 连接10.96.0.1:443，均不通

```
 curl -kv https://10.96.0.1:443/version
```

> 考虑查看kube-proxy日志

```
kubectl logs -n kube-system kube-proxy-p8fdr

...
Failed to retrieve node info: Unauthorized
...
```

> 报错日志来看是证书验证失败 ，需要删除kube-proxy 依赖的secret，让集群重新生成最新的secret

```
kubectl delete secret -n kube-system kube-proxy-token-kgrw7
```

> 等待secret重创，删除kube-proxy pod，重启kube-proxy

> 各个节点请求api-server，正常

```
 curl -kv https://10.96.0.1:443/version 
```

> 重新部署kube-flannel，查看flannel日志

```
NetworkPlugin cni failed to set up pod "xxxxx" network: failed to set bridge addr: "cni0" already has an IP address different from10.x.x.x - Error
```

> 删除之前已有网卡，重新服务，等待自动重新创建

```
ip link set cni0 down
ip link set flannel.1 down  
ip link delete cni0
ip link delete flannel.1
systemctl restart containerd / systemctl restart docker
systemctl restart kubelet
```
