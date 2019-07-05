# openshift 相关

```
yum -y install epel-release git ansible pyOpenSSL python-cryptography python-lxml

yum -y install origin-node-3.11* origin-clients-3.11* origin-3.11* conntrack-tools
```

```
git clone https://github.com/openshift/openshift-ansible.git
cd openshift-ansible
git checkout v3.11.0
```



```
imgs=(docker.io/cockpit/kubernetes:latest docker.io/openshift/origin-control-plane:v3.11 docker.io/openshift/origin-deployer:v3.11 docker.io/openshift/origin-docker-registry:v3.11 docker.io/openshift/origin-pod:v3.11 quay.io/coreos/etcd:v3.2.22 docker.io/openshift/origin-haproxy-router:v3.11 docker.io/openshift/origin-control-plane:v3.11.0 docker.io/openshift/origin-deployer:v3.11.0 docker.io/openshift/origin-haproxy-router:v3.11.0 docker.io/openshift/origin-pod:v3.11.0  docker.io/openshift/origin-docker-registry:v3.11.0)

for img in ${imgs[@]};do docker pull $img;done
```



```
ansible-playbook -i inventory/hosts.localhost playbooks/prerequisites.yml
ansible-playbook -i inventory/hosts.localhost playbooks/deploy_cluster.yml
```





```
ansible-playbook playbooks/adhoc/uninstall_openshift.yml
```



部署
----

```
yum -y install centos-release-openshift-origin311 epel-release docker git pyOpenSSL
systemctl start docker
systemctl enable docker
yum install -y install origin-clients-3.11* origin-node-3.11*
# 免密
yum -y install openshift-ansible

```

`/etc/ansible/hosts`

```
[OSEv3:children]
masters
nodes
etcd
[OSEv3:vars]
ansible_ssh_user=root
ansible_become=true
openshift_deployment_type=origin
openshift_use_openshift_sdn=false
os_sdn_network_plugin_name=cni
openshift_use_calico=true
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]
openshift_master_default_subdomain=apps.srv.world
openshift_docker_insecure_registries=172.30.0.0/16
#openshift_disable_check=disk_availability,docker_storage,memory_availability,docker_image_availability
openshift_disable_check=docker_storage
[masters]
ctrl.srv.world openshift_schedulable=true containerized=false
[etcd]
ctrl.srv.world
[nodes]
ctrl.srv.world openshift_node_group_name='node-config-master-infra'
node01.srv.world openshift_node_group_name='node-config-compute'

```

```
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/prerequisites.yml
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/deploy_cluster.yml
htpasswd /etc/origin/master/htpasswd admin

```

```
curl -O -L https://github.com/projectcalico/calicoctl/releases/download/v3.1.3/calicoctl
chmod a+x calicoctl
mv calicoctl /bin
calicoctl node status

```


### nfs
```
[root@ctrl ~]# cat /etc/exports
/var/nfsshare *(rw,sync,no_subtree_check,no_root_squash)

```

```
git clone https://github.com/kubernetes-incubator/external-storage.git
cd external-storage/nfs-client
# https://medium.com/faun/openshift-dynamic-nfs-persistent-volume-using-nfs-client-provisioner-fcbb8c9344e
# https://github.com/kubernetes-incubator/external-storage/blob/master/nfs-client/README.md
kubectl patch storageclass managed-nfs-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

```


### 卸载
```
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/adhoc/uninstall_openshift.yml

```

若要更改网络相关，还需要删除cni相关的配置文件
```
rm -rf /etc/cni/net.d/*
```