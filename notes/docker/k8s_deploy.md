# k8s 安装部署


## 一、使用kubespray安装(kubespray: v2.11.0, k8s:v1.15.3)
##### 1、查看系统是否支持虚拟化

```
egrep --color 'vmx|svm' /proc/cpuinfo
```

##### 2、关闭和禁用防火墙

```
systemctl stop firewalld
systemctl disable firewalld

setenforce 0
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

##### 3、禁用交换分区

```
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
```

##### 4、每个节点分别安装docker

[docker安装](https://hlyani.github.io/notes/docker/docker.html)

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

systemctl start docker
systemctl enable docker
```

##### 5、免密

```
ssh-keygen
ssh-copy-id HOSTNAME
```

##### 6、安装python3

```
yum install -y epel-release
yum makecache fast
yum install -y python36
```

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
```

##### 7、每个节点分别下载kubespray源码

```
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
git tag
git checkout v2.11.0
```

##### 8、每个节点分别安装相关依赖

```
pip3 install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

##### 9、复制配置文件

```
cp -rfp inventory/sample inventory/mycluster
```

##### 10、更新配置文件信息

>  申明数组，输入节点ip

```
declare -a IPS=(10.10.1.2 10.10.1.3 10.10.1.4)
```

> 将ip写入脚本

```
CONFIG_FILE=inventory/mycluster/hosts.yml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

##### 11、检查配置文件是否有问题

```
cat inventory/mycluster/group_vars/all/all.yml
cat inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml

kube_version: v1.15.3
```

##### 12、修改镜像源为国内源

##### azk8s.cn 支持镜像转换列表（可选）

| global                                                       | proxy in China                                               | format                  | example                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------- | ------------------------------------------------------------ |
| [dockerhub](https://www.cnblogs.com/xuxinkun/p/hub.docker.com) (docker.io) | [dockerhub.azk8s.cn](http://mirror.azk8s.cn/help/docker-registry-proxy-cache.html) | `dockerhub.azk8s.cn//:` | `dockerhub.azk8s.cn/microsoft/azure-cli:2.0.61` `dockerhub.azk8s.cn/library/nginx:1.15` |
| gcr.io                                                       | [gcr.azk8s.cn](http://mirror.azk8s.cn/help/gcr-proxy-cache.html) | `gcr.azk8s.cn//:`       | `gcr.azk8s.cn/google_containers/hyperkube-amd64:v1.13.5`     |
| quay.io                                                      | [quay.azk8s.cn](http://mirror.azk8s.cn/help/quay-proxy-cache.html) | `quay.azk8s.cn//:`      | `quay.azk8s.cn/deis/go-dev:v1.10.0`                          |

```
grc_image_files=(
./roles/download/defaults/main.yml
./inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
)

for file in ${grc_image_files[@]};do
    sed -i 's#gcr.io/google_containers#registry.cn-hangzhou.aliyuncs.com/kbspray#g' $file
    sed -i 's#k8s.gcr.io#registry.cn-hangzhou.aliyuncs.com/kbspray#g' $file
    sed -i 's#gcr.io/google-containers#registry.cn-hangzhou.aliyuncs.com/kbspray#g' $file
    sed -i 's#quay.io/coreos#registry.cn-hangzhou.aliyuncs.com/kbspray#g' $file
done

```

> 以下文件看情况需要翻墙下载（可以略过）

```
cd /tmp/releases

https://storage.googleapis.com/kubernetes-release/release/v1.14.3/bin/linux/amd64/kubeadm

https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz

https://storage.googleapis.com/kubernetes-release/release/v1.14.3/bin/linux/amd64/hyperkube

https://github.com/projectcalico/calicoctl/releases/download/v3.4.4/calicoctl-linux-amd64
```

##### 13、如果需要可以配置 docker 代理（可使用 privoxy），需要下载的容器镜像

> 配置docker代理

```
vim /etc/systemd/system/docker.service.d/http-proxy.conf
```

```
[Service]
Environment="HTTP_PROXY=http://proxy.server:port"
Environment="HTTPS_PROXY=http://proxy.server:port"
Environment="NO_PROXY=localhost,127.0.0.1"
```

```
systemctl daemon-reload
systemctl restart docker
```

> 需要下载的镜像

```
gcr.io/google-containers/kube-proxy
gcr.io/google-containers/kube-apiserver
gcr.io/google-containers/kube-controller-manager
gcr.io/google-containers/kube-scheduler
coredns/coredns
calico/node
calico/cni
calico/kube-controllers
k8s.gcr.io/k8s-dns-node-cache
nginx
k8s.gcr.io/cluster-proportional-autoscaler-amd64
gcr.io/google_containers/kubernetes-dashboard-amd64
quay.io/coreos/etcd
gcr.io/google-containers/pause
gcr.io/google_containers/pause-amd64
```

##### 14、开始部署

[https://github.com/kubernetes-sigs/kubespray/blob/master/docs/ansible.md](https://github.com/kubernetes-sigs/kubespray/blob/master/docs/ansible.md)

```
ansible-playbook -i inventory/mycluster/hosts.yml --become --become-user=root cluster.yml
```

##### 15、获取集群信息

```
kubectl cluster-info

Kubernetes master is running at https://192.168.21.88:6443
coredns is running at https://192.168.21.88:6443/api/v1/namespaces/kube-system/services/coredns:dns/proxy
kubernetes-dashboard is running at https://192.168.21.88:6443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

##### 16、创建 dashboard 用户

[https://github.com/kubernetes/dashboard/wiki/Creating-sample-user](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user)

```
# cat dashboard-admin-user.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

```
kubectl apply -f dashboard-admin-user.yaml
```

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

##### 17、安装nginx测试

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dm
spec:
  replicas: 3
  selector:
    matchLabels:
      name: nginx
  template:
    metadata:
      labels:
        name: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
              name: http
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  ports:
    - port: 80
      name: http
      targetPort: 80
      protocol: TCP
  type: NodePort
  selector:
    name: nginx
```

```
kubectl apply -f nginx.yaml 
```

```
kubectl get service | grep nginx
```

##### 18、清理环境

```
ansible-playbook -i inventory/mycluster/hosts.yml reset.yml

rm -rf /etc/kubernetes/
rm -rf /var/lib/kubelet
rm -rf /var/lib/etcd
rm -rf /usr/local/bin/kubectl
rm -rf /etc/systemd/system/kubelet.service
systemctl stop etcd.service
systemctl disable etcd.service
docker stop $(docker ps -q)
docker rm $(docker ps -qa)
systemctl restart docker
```

##### 19、升级

[https://github.com/kubernetes-sigs/kubespray/blob/master/docs/upgrades.md](https://github.com/kubernetes-sigs/kubespray/blob/master/docs/upgrades.md)

* 升级kube

  ```
  git fetch origin
  git checkout origin/master
  ansible-playbook upgrade-cluster.yml -b -i inventory/sample/hosts.ini -e kube_version=v1.6.0
  ```

* 升级docker

  ```
  ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=docker
  ```

* 升级etcd

  ```
  ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=etcd
  ```

* 升级kubelet

  ```
  ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=node --skip-tags=k8s-gen-certs,k8s-gen-tokens
  ```

* 升级网络插件

  ```
  ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=network
  ```

* 升级所有的add-ones

  ```
  ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=apps
  ```

* 只升级helm(假定helm_enabled配置为true)

  ```
  ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=helm
  ```

* 升级master组件

  ```
  ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=master
  ```

##### 20、添加、减少节点

```
ansible-playbook -i inventory/mycluster/hosts.yml scale.yml -b -v \
  --private-key=~/.ssh/private_key  
```

```
ansible-playbook -i inventory/mycluster/hosts.yml remove-node.yml -b -v \
  --private-key=~/.ssh/private_key \
  --extra-vars "node=nodename,nodename2"
```

##### 21、其他

```
journalctl -u -f
```
```
kubectl edit service -n kube-system kubernetes-dashboard
```

```
google_containers
```

```
#查看部署组件
kubectl get all --namespace=kube-system
```

```
kubectl logs --namespace=kube-system kubernetes-dashboard-7bf56bf786-4pwpj --follow
```

```
1 node(s) had taints that the pod didn't tolerate.
有时候一个pod创建之后一直是pending，没有日志，也没有pull镜像，describe的时候发现里面有一句话： 1 node(s) had taints that the pod didn't tolerate.

直译意思是节点有了污点无法容忍，执行 kubectl get no -o yaml | grep taint -A 5 之后发现该节点是不可调度的。这是因为kubernetes出于安全考虑默认情况下无法在master节点上部署pod，于是用下面方法解决：

kubectl taint nodes --all node-role.kubernetes.io/master-
```

```
kubectl api-resources\
kubectl explain XXX
```

## 二、使用Minikube安装（适合单机部署开发）

> 安装kubenets版本v1.15

##### 1、查看系统是否支持虚拟化

```
egrep --color 'vmx|svm' /proc/cpuinfo
```

##### 2、关闭和禁用防火墙

```
systemctl stop firewalld
systemctl disable firewalld

setenforce 0
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

##### 3、禁用交换分区

```
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
```

##### 4、安装minikube

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && mv minikube /usr/local/bin/
```

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube

install minikube /usr/local/bin
```

##### 5、安装docker

[docker安装](https://hlyani.github.io/notes/docker/docker.html)

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

systemctl start docker
systemctl enable docker
```

##### 6、安装kubelet

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum -y  makecache fast

yum install -y kubectl

systemctl start kubelet
systemctl enable kubelet
```

##### 7、不依赖虚拟机模拟启动

> 执行下面命令会自动安装kubeadm、kubelet两个软件
>
> 在拉取镜像时会访问https://k8s.gcr.io/v2/失败，需要翻墙
>
> --image-repository 使用我同步到aliyun仓库的k8s镜像

```
minikube start --vm-driver=none --kubernetes-version v1.15.0 -v 0 --image-repository registry.cn-hangzhou.aliyuncs.com/hlyani
```

```
--registry-mirror=https://registry.docker-cn.com
```

```
* minikube v1.2.0 on linux (amd64)
* using image repository registry.cn-hangzhou.aliyuncs.com/ates-k8s
* Creating none VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
* Configuring environment for Kubernetes v1.15.0 on Docker 18.09.6
* Downloading kubeadm v1.15.0
* Downloading kubelet v1.15.0
* Pulling images ...
* Launching Kubernetes ... 
* Configuring local host environment ...

! The 'none' driver provides limited isolation and may reduce system security and reliability.
! For more information, see:
  - https://github.com/kubernetes/minikube/blob/master/docs/vmdriver-none.md
! kubectl and minikube configuration will be stored in /root
! To use kubectl or minikube commands as your own user, you may
! need to relocate them. For example, to overwrite your own settings:

  - sudo mv /root/.kube /root/.minikube $HOME
  - sudo chown -R $USER $HOME/.kube $HOME/.minikube

* This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
* Verifying: apiserver proxy etcd scheduler controller dns
* Done! kubectl is now configured to use "minikube"
* For best results, install kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl/
```

##### 8、执行下面命令先拉去镜像（可选）

```
kubeadm config images list
kubeadm config images pull --kubernetes-version v1.15.0 --image-repository registry.cn-hangzhou.aliyuncs.com/hlyani
```

```
docker images
REPOSITORY                                                         TAG                 IMAGE ID            CREATED             SIZE
registry.cn-hangzhou.aliyuncs.com/hlyani/kube-proxy                v1.15.0             d235b23c3570        8 days ago          82.4MB
registry.cn-hangzhou.aliyuncs.com/hlyani/kube-apiserver            v1.15.0             201c7a840312        8 days ago          207MB
registry.cn-hangzhou.aliyuncs.com/hlyani/kube-scheduler            v1.15.0             2d3813851e87        8 days ago          81.1MB
registry.cn-hangzhou.aliyuncs.com/hlyani/kube-controller-manager   v1.15.0             8328bb49b652        8 days ago          159MB
registry.cn-hangzhou.aliyuncs.com/hlyani/coredns                   1.3.1               eb516548c180        5 months ago        40.3MB
registry.cn-hangzhou.aliyuncs.com/hlyani/etcd                      3.3.10              2c4adeb21b4f        6 months ago        258MB
registry.cn-hangzhou.aliyuncs.com/hlyani/pause                     3.1                 da86e6ba6ca1        18 months ago       742kB
```

##### 9、查看init-defaults

```
kubeadm config print init-defaults 
```

##### 10、清除部署（可选）

```
minikube delete
```

```
[preflight] Running pre-flight checks
[preflight] Running pre-flight checks
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
[reset] Deleting contents of stateful directories: [/var/lib/kubelet /etc/cni/net.d /var/lib/dockershim /var/run/kubernetes]

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually.
For example:
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
```

##### 11、让kubectl使用minikube的配置文件

```
kubectl config use-context minikube 
```

##### 12、dashboard

```
minikube dashboard --url
```

##### 13、测试，部署nginx

```
 kubectl run hello --image=nginx --port=80
 kubectl expose deployment hello --type=NodePort
 #kubectl expose deployment hello --port=80 --type=LoadBalancer
 
 kubectl get pod
 curl $(minikube service hello --url)
```

##### 14、其他

```
minikube有一个强大的子命令叫做addons，它可以在kubernetes中安装插件（addon）。这里所说的插件，就是部署在Kubernetes中的Deploymen或者Daemonset。

Kubernetes在设计上有一个特别有意思的地方，就是它的很多扩展功能，甚至于基础功能也可以部署在Kubernetes中，比如网络插件、DNS插件等。安装这些插件的时候， 就是用kubectl命令，直接在kubernetes中部署。
```

```
minikube addons list
minikube addons disable XXX
minikube addons enable  XXX
```

## 三、使用kubeadm安装

##### 1、安装kubeadm, kubelet 和 kubect

- `kubeadm`: the command to bootstrap the cluster.
- `kubelet`: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
- `kubectl`: the command line util to talk to your cluster.

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum -y  makecache fast

yum install -y kubectl kubeadm kubelet

systemctl start kubelet
systemctl enable kubelet
```

##### 2、由于绕过了iptables而导致路由错误的问题。

```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
##### 3、禁用交换分区和禁用防火墙

```
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab

setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

systemctl stop firewalld
systemctl disable firewalld
```
##### 4、安装docker

[docker安装](https://hlyani.github.io/notes/docker/docker.html)

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

systemctl start docker
systemctl enable docker
```

##### 5、使用kubeadm开始安装

```kubeadm.yaml
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
api:
  advertiseAddress: "192.168.1.10"
networking:
  podSubnet: "10.0.0.0/24"
kubernetesVersion: "v1.15.0"
imageRepository: "registry.cn-hangzhou.aliyuncs.com/hlyani"
```

```
kubeadm init --config kubeam.yaml
```
或
```
kubeadm init --image-repository registry.cn-hangzhou.aliyuncs.com/hlyani --apiserver-advertise-address=192.168.1.10 --kubernetes-version=v1.15.0 --pod-network-cidr=10.0.0.0/24 -v 0
```

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.10:6443 --token 4mgxai.ou2w2ff7zwz5ykym \
    --discovery-token-ca-cert-hash sha256:16bf000d7f9ce90f932f13747727e9795eac8babe901f7fae5842347a661d67a 

```

##### 6、安装网络插件

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml
```
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/62e44c867a2846fefb68bd5f178daf4da3095ccb/Documentation/kube-flannel.yml
```

> Error registering network: failed to acquire lease: node "node1" pod cidr not assigned

[https://github.com/coreos/flannel/blob/fd8c28917f338a30b27534512292cd5037696634/Documentation/troubleshooting.md#kubernetes-specific](https://github.com/coreos/flannel/blob/fd8c28917f338a30b27534512292cd5037696634/Documentation/troubleshooting.md#kubernetes-specific)

```
 kubectl patch node node1 -p '{"spec":{"podCIDR":"10.0.0.0/24"}}'
```
```
kubectl apply -f https://docs.projectcalico.org/v3.7/manifests/calico.yaml
```

##### 7、添加node节点

```
kubeadm join 192.168.1.10:6443 --token 4mgxai.ou2w2ff7zwz5ykym \
    --discovery-token-ca-cert-hash sha256:16bf000d7f9ce90f932f13747727e9795eac8babe901f7fae5842347a661d67a 
```

```
kubeadm token list

openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'

kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

##### 8、安装dashboard

[https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard)

[https://github.com/kubernetes-retired/heapster](https://github.com/kubernetes-retired/heapster)

```
git clone https://github.com/kubernetes-retired/heapster.git
git tag
git checkout v1.5.4

ls d/
grafana.yaml  heapster-rbac.yaml  heapster.yaml  influxdb.yaml

kubectl apply -f d/
```

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml

#image: registry.cn-hangzhou.aliyuncs.com/hlyani/kubernetes-dashboard-amd64:v1.10.1
spec:
  ports:
    - port: 443
      targetPort: 8443
  type: NodePort
```

###### 修改

```
kubectl edit service -n kube-system kubernetes-dashboard
```

[https://github.com/kubernetes/dashboard/wiki/Creating-sample-user](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user)

```
# cat dashboard-admin-user.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system


kubectl apply -f dashboard-admin-user.yaml
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

##### 9、失败清理环境

```
rm -rf /var/lib/etcd/*
rm -rf /var/lib/kubelet/*
rm -rf /etc/kubernetes/*
docker rm -f `docker ps -qa`
kill -9 `netstat -ntlp|grep 102|awk '{print $7}'|cut -d'/' -f1`
echo "1" >/proc/sys/net/bridge/bridge-nf-call-iptables
kubeadm reset -f
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

## 四、FAQ

#### 1、Warning  VolumeResizeFailed  13m                 volume_expand  error expanding volume "default/data-mariadb-master-0" of plugin "kubernetes.io/rbd": rbd info failed, error: exit status 127

> 扩容pvc时失败问题

```
kubcectl get po -A
```

```
kubectl get pvc
```

```
kubectl describe pvc data-mariadb-master-0
```

```
echo "deb http://mirrors.aliyun.com/debian/ stretch main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ stretch main non-free contrib
deb http://mirrors.aliyun.com/debian-security stretch/updates main
deb-src http://mirrors.aliyun.com/debian-security stretch/updates main
deb http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib
deb http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib" | tee > sources.list
```

```
kubectl cp sources.list --namespace kube-system kube-controller-manager-node1:/etc/apt/
```

```
kubectl exec -it --namespace kube-system kube-controller-manager-node1 sh
apt update && apt install -y ceph-common
```

```
kubectl cp /etc/ceph/ceph.client.admin.keyring --namespace kube-system kube-controller-manager-node1:/etc/ceph/

kubectl cp /etc/ceph/ceph.conf --namespace kube-system kube-controller-manager-node1:/etc/ceph/
```

```
rbd ls k8s
```

