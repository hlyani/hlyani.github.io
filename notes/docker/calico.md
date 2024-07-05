# Calico

Calico是三层虚拟网络解决方案（BGP）。每一个节点都是一个vRouter，都需要通过BGP协议学习生成路由规则，从而实现各节点上Pod之间互联互通。

# 一、BGP通信模型

BGP模型要求所有节点在同一个二层网络中。不一定所有的底层网络都支持BGP。

## 1、BGP peer（小规模网络使用）

点对点BGP，如果一个网络中有10个BGP，即是1:9的通信模型，形成n*(n-1)个通信网络。

所以在此模型下如果网络规模较大BGP路由学习报文会占据很大的网络带宽。

BGP peer不存在单点问题，BGP peer宕机会有其他的进行替代。

## 2、BGP Reflector（大规模网络使用）

反射器模型，所有节点都将自己所有拥有的路由信息汇总给Reflector，由Reflector用1:n-1的方式向外进行反射。

BGP Reflector需要做冗余。

# 二、Overlay Network

## 1、IPIP

用IP报文来封装IP报文，因此其开销更小。

## 2、VXLAN

类似于Flannel的VXLAN启动DirectRouting的网络模型，Calico也支持混合使用路由和叠加网络模型。

如果节点在同一子网内使用BGP，如果跨子网则使用VXLAN。

# 三、架构

Flannel中host-gw模型使用veth-pair

Calico使用内核Iptables和Routes表来完成其中部分功能。

## 主要组件：

##### 1.每个节点都需要运行组件：

* BGP客户端（默认启用）：需要运行于每个节点，负责将Felix生成的路由信息载入内核并通告到整个网络中。
* BGP Reflector：专用反射各BGP客户端发来路由信息，将N->N-1转为N->1模型。
* Felix：需要运行于各节点之上的守护进程，主要负责完成接口管理、路由规划、acl规划（即网络策略，借助iptables实现）、状态报告。
* BIRD：是vRouter的关键实现，整个BGP的路由表是由BIRD生成的，而路由规划是Felix完成的。BIRD自身可以扮演两种角色。

##### 2.在节点之外需要运行组件：

* etcd：Calico也会Flannel一样需要依靠etcd来保存一些自身的状态数据，也可以像Flannel一样将API Server当为自身的存储后端。大规模集群中建议额外部署etcd专用于Calico集群，以免和K8S性能上冲突。
* Route Reflector：路由反射器。
* Calico编排系统插件：Calico不仅支持给K8S提供虚拟网络，也支持OpenShift、OpenStack。所以Calico是一个通用的虚拟网络。要让Calico能适用于K8S，需要一个Calico的编排系统插件让etcd和Calico插件之间能双向转换通信。

##### 3.K8S所需组件：

* calico-node：类似于Flanneld需要运行于每个节点之上。calico-node中封装了Felix和BIRD。
* calico-kube-controller：运行于K8S集群上的中央控制系统。负责Calico和整个K8S的协同，也包括其他核心功能实现。

# 四、部署

[https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises](https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises)

## 1、下载calico资源清单

```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
```

## 2、配置

确保Pod的CIDR为192.168.0.0/16，如果非此网点则需要修改calico.yaml。

## 3、部署

```
kubectl apply -f calico.yaml
```

## 4、验证

calico默认使用IPIP模型。

```
# 查看路由信息
ip r list

blackhole 10.244.0.0/24 proto bird         # blackhole 表示当前节点。
10.244.0.4 dev cali5411bb555f9 scope link  # 此处可以看到 pod 的数据流出是直接到达内核的
10.244.1.0/24 via 172.16.11.81 dev tunl0 proto bird onlink
# 出现 tunl0 接口，现在报文发送时会发送给 tunl0 接口
# tunl0 宿主机内核
```

查看calico的地址池

```
kubectl get ippools -o yaml
```

验证ipip工作逻辑

```
tcpdump -i eth0 -nn ip host k8s-node01 and host k8s-node03

02:33:55.805615 IP 172.16.11.81 > 172.16.11.83: IP 10.244.1.16.49490 > 10.244.3.16.10016: Flags [P.], seq 1133911456:1133911482, ack 3769557439, win 85, options [nop,nop,TS val 3535292140 ecr 2812652948], length 26 (ipip-proto-4)

# 从以上抓包结果中可以看出 ipip 在通信时，分为外部 ip 和内部 ip 2层。
# ipip-proto-4 表示此报文为 ipip 报文。
```

# 五、calicoctl

```
 curl -o /usr/bin/kubectl-calico -O -L  "https://github.com/projectcalico/calicoctl/releases/download/v3.21.5/calicoctl-linux-amd64"
```

```
kubectl-calico -h

kubectl calico -h
```

```
vim /etc/calico/calicoctl.cfg

apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: "kubernetes" # 此处申明后端存储为kubernetes
  kubeconfig: "/root/.kube/config"
```

获取节点

```
kubectl calico get nodes
```

查看节点状态

```
kubectl-calico node status
```

获取地址池

```
kubectl calico get ippool
kubectl calico get ippool -o yaml
```

查看地址分配信息

```
kubectl calico ipam show --allow-version-mismatch
```

查看每个节点上的地址分配信息

```
kubectl calico ipam show --show-blocks
```

查看ipam配置信息

```
kubectl calico ipam show --show-configuration

+--------------------+-------+
|      PROPERTY      | VALUE |
+--------------------+-------+
| StrictAffinity     | false |   # pod被重建后是否使用原有地址
| AutoAllocateBlocks | true  |   # 是否支持自动分配地址
| MaxBlocksPerHost   |     0 |
+--------------------+-------+
```

# 六、配置

```
kubectl calico get ippools -o yaml
```

```
apiVersion: projectcalico.org/v3
items:
- apiVersion: projectcalico.org/v3
  kind: IPPool
  metadata:
    creationTimestamp: "2024-05-06T06:00:24Z"
    name: default-ipv4-ippool
    resourceVersion: "6789"
    uid: 943b85b2-9759-49ce-8f73-78f1f3f8a111
  spec:
    blockSize: 24
    cidr: 192.168.0.0/16
    ipipMode: CrossSubnet      # 将ipipMode改为CrossSubnet或Never
    natOutgoing: true
    nodeSelector: all()
    vxlanMode: Never
kind: IPPoolList
metadata:
  resourceVersion: "9418"

# 改BGP模式需要修改ipipMode
# CrossSubnet表示混杂模式也就是混合模式，表示跨节点子网时才使用IPIP
# Never表示纯BGP模式

# vxlanMode: CrossSubnet ipipMode: Never 表示VxLan的混合模型
```

1. 将其重新应用到网络中

```
kubectl calico apply -f default-ipv4-ippool.yaml
```

BGP生效再次查看路由信息

```
ip route list


```

