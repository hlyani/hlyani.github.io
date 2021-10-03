# kolla部署相关

## 一、环境要求

> 两张网卡

> 除系统盘外，至少1块磁盘

## 二、安装

[kolla deploy](https://docs.openstack.org/project-deploy-guide/kolla-ansible/ocata/quickstart.html#deploy-kolla)

##### 1、安装相关依赖

```
#CentOS
dnf install -y python3-devel libffi-devel gcc openssl-devel python3-libselinux python3-venv

#Ubuntu
apt install -y python3-dev libffi-dev gcc libssl-dev python3-venv
```

##### 2、安装docker

[aliyun 安装docker-ce](https://yq.aliyun.com/articles/110806)

[tsinghua 安装docker-ce](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)

```
curl -sSL https://get.docker.io | bash

or

curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

##### 3、python 虚拟环境

```
mkdir /kolla
python3 -m venv /kolla/venv3
source /kolla/venv3/bin/activate
```

##### 4、禁用防火墙

```
setenforce 0
systemctl stop firewalld
systemctl disable firewalld

iptables -L
```

##### 5、安装 kolla 和 kolla-ansible

```
cd /kolla

git clone -b stable/wallaby --depth=1 https://github.com/openstack/kolla
git clone -b stable/wallaby --depth=1 https://github.com/openstack/kolla-ansible

pip install ./kolla
pip install ./kolla-ansible

pip install docker python-openstackclient
```

##### 6、复制配置文件globals.yml、password.yml到/etc中

```
cp -r kolla-ansible/etc/kolla /etc/
cp -r kolla-ansible/ansible/inventory/multinode .
```

##### 7、修改multinode

```
[control]
node1 ansible_python_interpreter=/kolla/venv3/bin/python3

[network]
node1 ansible_python_interpreter=/kolla/venv3/bin/python3

[compute]
node1 ansible_python_interpreter=/kolla/venv3/bin/python3

[monitoring]
node1 ansible_python_interpreter=/kolla/venv3/bin/python3

[storage]
node1 ansible_python_interpreter=/kolla/venv3/bin/python3

[deployment]
node1 ansible_connection=local ansible_python_interpreter=/kolla/venv3/bin/python3
```

##### 8、ceph 准备

> 部署参考，[ceph部署](https://hlyani.github.io/notes/ceph/ceph_deploy.html)

```
ceph osd pool create volumes 128
ceph osd pool create images 128
ceph osd pool create backups 128
ceph osd pool create vms 128

rbd pool init volumes
rbd pool init images
rbd pool init backups
rbd pool init vms

ceph auth get-or-create client.glance mon 'profile rbd' osd 'profile rbd pool=images'
ceph auth get-or-create client.cinder mon 'profile rbd' osd 'profile rbd pool=volumes, profile rbd pool=vms, profile rbd pool=images'
ceph auth get-or-create client.cinder-backup mon 'profile rbd' osd 'profile rbd pool=backups'

ceph auth get-or-create client.glance|tee ceph.client.glance.keyring
ceph auth get-or-create client.cinder|tee ceph.client.cinder.keyring
ceph auth get-or-create client.cinder-backup|tee ceph.client.cinder-backup.keyring
```

##### 9、部署前配置

```
kolla-genpwd
```

```
tree /etc/kolla/config/
├── cinder
│   ├── ceph.client.cinder.keyring
│   ├── ceph.conf
│   ├── cinder-backup
│   │   ├── ceph.client.cinder-backup.keyring
│   │   ├── ceph.client.cinder.keyring
│   │   └── ceph.conf
│   └── cinder-volume
│       ├── ceph.client.cinder.keyring
│       └── ceph.conf
├── glance
│   ├── ceph.client.glance.keyring
│   └── ceph.conf
├── glance.conf
└── nova
    ├── ceph.client.cinder.keyring
    └── ceph.conf
```

```
vim /etc/kolla/globals.yaml

config_strategy: "COPY_ALWAYS"
kolla_base_distro: "ubuntu"
kolla_install_type: "source"
openstack_release: "wallaby"
kolla_internal_vip_address: "192.168.0.243"
kolla_external_vip_address: "{{ kolla_internal_vip_address }}"
docker_registry: 192.168.0.90:3000
docker_namespace: "kolla"
network_interface: "eno1"
neutron_external_interface: "eno2"
keepalived_virtual_router_id: "242"
enable_chrony: "yes"
enable_cinder: "yes"
enable_cinder_backup: "no"
enable_cyborg: "no"
enable_fluentd: "yes"
enable_magnum: "yes"
external_ceph_cephx_enabled: "yes"
ceph_glance_keyring: "ceph.client.glance.keyring"
ceph_glance_user: "glance"
ceph_glance_pool_name: "images"
ceph_cinder_keyring: "ceph.client.cinder.keyring"
ceph_cinder_user: "cinder"
ceph_cinder_pool_name: "volumes"
ceph_nova_keyring: "{{ ceph_cinder_keyring }}"
#ceph_nova_user: "nova"
ceph_nova_user: "{{ ceph_cinder_user }}"
ceph_nova_pool_name: "vms"
glance_backend_ceph: "yes"
glance_backend_file: "no"
cinder_backend_ceph: "yes"
nova_backend_ceph: "yes"
```

##### 10、部署前检查（可选）

```
kolla-ansible -i /kolla/multinode prechecks
```

##### 11、拉取镜像（可选）

```
kolla-ansible -i /kolla/multinode pull

kolla-ansible pull -e 'ansible_python_interpreter=/kolla/venv3/bin/python3'
```

##### 12、部署

```
kolla-ansible -i /kolla/multinode deploy

# 只部署某些组件
kolla-ansible -i /kolla/multinode deploy --tags="haproxy"

# 过滤部署某些组件
kolla-ansible -i /kolla/multinode deploy --skip-tags="haproxy"
```

##### 13、生成 admin-openrc.sh

```
kolla-ansible -i /kolla/multinode post-deploy
```

##### 14、运行 init-runonce

[init-runonce参考](https://github.com/hlyani/kolla/blob/master/init-runonce)

```
cd /usr/share/kolla-ansible
vim init-runonce
./init-runonce
```

## 二、其他问题

##### 1、ceph pg size问题

```
pool_pg_num 默认是128，用户需要根据需要进行修改,对于此平台我们提供以下计算公式供用户设定pool_pg_num。
40*osd_size >= replic_size *(8 + pool_pg_num)
注: osd_size为集群中osd的总数，此处即为所有存储服务器上用作存储的硬盘数目之和。
replic_size为每个pool的副本数，默认为3.
pool_pg_num是我们需要计算的pg数，pool_pg_num必须为2的N次方。
```

##### 2、kolla-ansible自带工具

```
# 可用于从系统中移除部署的容器
/usr/local/share/kolla-ansible/tools/cleanup-containers

#可用于移除由于残余网络变化引发的docker启动的neutron-agents主机
/usr/local/share/kolla-ansible/tools/cleanup-host

#可用于从本地缓存中移除所有的docker image
/usr/local/share/kolla-ansible/tools/cleanup-images
```

##### 3、mariadb集群出现故障

```
#!/bin/bash
ansible -i multinode all  -m shell -a 'docker stop mariadb'

ansible -i multinode all -m shell -a "sed -i 's/safe_to_bootstrap: 0/safe_to_bootstrap: 1/g' /var/lib/docker/volumes/mariadb/_data/grastate.dat"

kolla-ansible mariadb_recovery -i multinode
```

##### 4、Some qemu processes were detected.\nDocker will not be able to stop the nova_libvirt container with those running.

```
pgrep qemu  ##查找qemu的进程ID
kill -9 <QEMU_ID>
```

##### 5、清除 iptables 规则

```
iptables -F; iptables -X; iptables -Z
```

##### 6、清除上次部署

```
kolla-ansible destroy -i multinode --yes-i-really-really-mean-it
```

##### 7、rabbitmq异常

```
先重启所有节点rabbitmq（多适用于关机导致的异常）
ansible -i multinode all -m shell -a 'docker restart rabbitmq'
#Multiple different configurations with equal version numbers detected. Shutting down.

如果重启节点没用，再删除并重新部署所有节点rabbitmq（多适用于部署时出现的异常）
ansible -i multinode all -m shell -a 'docker rm -f rabbitmq'
ansible -i multinode all -m shell -a 'docker volume rm rabbitmq'
ansible -i multinode all -m shell -a 'rm -rf /etc/kolla/rabbitmq'
kolla-ansible deploy -i multinode
```

##### 8、nova_libvirt异常

```
ansible -i multinode all -m shell -a 'rm -rf /var/run/libvirtd.pid;docker restart nova_libvirt nova_compute'
```

##### 9、扩展

```
修改globals.yaml
若只是修改一些不涉及组件和镜像的配置（不增删容器），只需修改完globals后，upgrade、post-deploy即可
kolla-ansible upgrade -i multinode
kolla-ansible post-deploy -i multinode
若新开组件和关闭组件(包括增加新的容器，如新起osd容器)，以及更新openstack版本，不清理之前配置直接deploy即可
# Valid option is Docker repository tag
openstack_release: “4.0.0” #修改为最新的tag

kolla-ansible deploy -i multinode
kolla-ansible post-deploy -i multinode
如果修改了/etc/config/[server].conf
kolla-ansible reconfigure --tags [server] -i multinode
```

##### 10、缩减

```
ceph
mon(例删除mon 10.0.0.1)
docker exec ceph_mon ceph mon dump
docker exec ceph_mon ceph mon rm 10.0.0.1
docker exec ceph_mon ceph osd crush remove 10.0.0.1
删除各个节点/var/lib/docker/volumes/ceph_mon_config/_data/ceph.conf对应条目
删除multinode对应条目
osd(例删除osd.0)
docker exec ceph_mon ceph osd out osd.0
docker exec ceph_mon ceph osd crush rm osd.0
docker exec ceph_mon ceph auth del osd.0
docker exec ceph_mon ceph osd down osd.0
docker exec ceph_mon ceph osd rm osd.0

openstack
减少controller（控制）节点
vim multinode 去掉相关控制节点
kolla-ansible deploy -i multinode
减少compute（计算）节点
openstack compute service list
openstack compute service delete ID
vim multinode 去掉相关计算节点
```

##### 11、修改openstack容器内的源码

```
例nova-api：

docker exec -itu0 nova_api bash
cd /var/lib/kolla/venv/lib/python2.7/site-packages/nova/
```

##### 12、指定python环境

```
kolla-ansible pull -e 'ansible_python_interpreter=/root/venv3/bin/python3'

kolla-ansible -e 'ansible_python_interpreter=/root/venv3/bin/python3' -i multinode prechecks

kolla-ansible -e 'ansible_python_interpreter=/root/venv3/bin/python3' -i multinode deploy
```

```
[control]
kolla1 ansible_python_interpreter=/root/venv3/bin/python3
kolla2 ansible_python_interpreter=/root/venv3/bin/python3
kolla3 ansible_python_interpreter=/root/venv3/bin/python3
```

##### 13、防止部署neutron-dhcp-agent时失败，配置kolla.conf

```
# Create the drop-in unit directory for docker.service
mkdir -p /etc/systemd/system/docker.service.d

# Create the drop-in unit file
tee /etc/systemd/system/docker.service.d/kolla.conf <<-'EOF'
[Service]
MountFlags=shared
EOF

systemctl daemon-reload
systemctl restart docker
```

##### 14、禁用宿主机的libvirt

```
# CentOS 7
systemctl stop libvirtd.service
systemctl disable libvirtd.service

# Ubuntu
service libvirt-bin stop
update-rc.d libvirt-bin disable
/usr/sbin/libvirtd: error while loading shared libraries:
libvirt-admin.so.0: cannot open shared object file: Permission denied
sudo apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
```

##### 15、旧版本，配置磁盘（新版本默认使用 bluestore）

```
[storage]
storage_node1_hostname ceph_osd_store_type=bluestore
storage_node2_hostname ceph_osd_store_type=bluestore
storage_node3_hostname ceph_osd_store_type=filestore
storage_node4_hostname ceph_osd_store_type=filestore

parted /dev/sdb -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_BS 1 -1

# 日志和数据在同一个磁盘
parted /dev/xvdb -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP 1 -1
parted /dev/xvdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP 1 -1

ansible -i multinode all -m shell -a 'parted /dev/vdb -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP 1 -1'

# 使用日志盘
parted /dev/vdb -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDB 1 -1
parted /dev/vdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC 1 -1

parted /dev/vdd -s mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDB_J 0% 5GB \
-s mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC_J 5GB 100%

parted /dev/vdb print
parted /dev/vdc print
```

##### 16、配置网络

```
bond0 宿主机网卡，管理网网卡（internel、存储。。。）
bond1：管理网外部网卡（public,vip）
bond2：外网网卡（浮动ip等，up网卡，不用配置ip）
bond3:存储网络、tunnel网络

ip addr show
ip link set bond2 up
```

##### 17、添加config文件（可选）

```
mkdir /etc/kolla/config
cd /etc/kolla/config
touch ceph.conf
touch cinder.conf
touch nova.conf
touch keystone.conf
touch polling.yaml
```

```ceph.conf
[global]
osd pool default size = 3
osd pool default min size = 2
mon_max_pg_per_osd = 200
osd crush update on start = false
```

```cinder.conf
[DEFAULT]
enable_force_upload = true
```

```nova.conf
[vnc]
novncproxy_base_url = http://111.111.111.111:6080/vnc_auto.html

[libvirt]
cpu_mode = host-passthrough
```

```keystone.conf
[token]
driver = keystone.token.backends.sql.Token
expiration = 86400
```

```polling.yaml
sources:
- interval: 180
  meters:
  - cpu
  - cpu_util
  - cpu_l3_cache
  - memory.usage
  - network.incoming.bytes
  - network.incoming.packets
  - network.outgoing.bytes
  - network.outgoing.packets
  - disk.device.read.bytes
  - disk.device.read.requests
  - disk.device.write.bytes
  - disk.device.write.requests
  - disk.device.usage
  - disk.device.iops
  name: some_pollsters
```

##### 18、调整日志

```
ln -sf /var/lib/docker/volumes/kolla_logs/_data/ /var/log/kolla
```

##### 19、格式转换与上次镜像

```
qemu-img convert -f qcow2 -O raw cirros-0.5.2-x86_64-disk.img cirros-0.5.2-x86_64-disk.raw
```

```
openstack image create --disk-format qcow2 --container-format bare --public --property os_type=linux --file cirros-0.5.2-x86_64-disk.img cirros
```

##### 20、镜像

```
kolla/ubuntu-source-nova-compute:wallaby
kolla/ubuntu-source-cinder-volume:wallaby
kolla/ubuntu-source-nova-novncproxy:wallaby
kolla/ubuntu-source-cinder-api:wallaby
kolla/ubuntu-source-cinder-scheduler:wallaby
kolla/ubuntu-source-nova-conductor:wallaby
kolla/ubuntu-source-nova-ssh:wallaby
kolla/ubuntu-source-nova-api:wallaby
kolla/ubuntu-source-nova-scheduler:wallaby
kolla/ubuntu-source-keystone-ssh:wallaby
kolla/ubuntu-source-keystone:wallaby
kolla/ubuntu-source-keystone-fernet:wallaby
kolla/ubuntu-source-magnum-api:wallaby
kolla/ubuntu-source-magnum-conductor:wallaby
kolla/ubuntu-source-neutron-server:wallaby
kolla/ubuntu-source-placement-api:wallaby
kolla/ubuntu-source-horizon:wallaby
kolla/ubuntu-source-neutron-dhcp-agent:wallaby
kolla/ubuntu-source-neutron-metadata-agent:wallaby
kolla/ubuntu-source-neutron-l3-agent:wallaby
kolla/ubuntu-source-neutron-openvswitch-agent:wallaby
kolla/ubuntu-source-glance-api:wallaby
kolla/ubuntu-source-cyborg-agent:wallaby
kolla/ubuntu-source-cyborg-conductor:wallaby
kolla/ubuntu-source-cyborg-api:wallaby
kolla/ubuntu-source-heat-engine:wallaby
kolla/ubuntu-source-heat-api-cfn:wallaby
kolla/ubuntu-source-heat-api:wallaby
kolla/ubuntu-source-kolla-toolbox:wallaby
kolla/ubuntu-source-mariadb-server:wallaby
kolla/ubuntu-source-mariadb-clustercheck:wallaby
kolla/ubuntu-source-openvswitch-db-server:wallaby
kolla/ubuntu-source-nova-libvirt:wallaby
kolla/ubuntu-source-openvswitch-vswitchd:wallaby
kolla/ubuntu-source-rabbitmq:wallaby
kolla/ubuntu-source-fluentd:wallaby
kolla/ubuntu-source-memcached:wallaby
kolla/ubuntu-source-chrony:wallaby
kolla/ubuntu-source-cron:wallaby
kolla/ubuntu-source-keepalived:wallaby
kolla/ubuntu-source-haproxy:wallaby
```

## 三、添加节点

[adding-and-removing-hosts](https://docs.openstack.org/kolla-ansible/latest/user/adding-and-removing-hosts.html)

```
kolla-ansible -i <inventory> bootstrap-servers
kolla-ansible -i <inventory> pull
kolla-ansible -i <inventory> deploy
```

## 四、卸载节点

```
l3_id=$(openstack network agent list --host <host> --agent-type l3 -f value -c ID)
#target_l3_id=$(openstack network agent list --host <target host> --agent-type l3 -f value -c ID)
openstack router list --agent $l3_id -f value -c ID | while read router; do
  openstack network agent remove router $l3_id $router --l3
#  openstack network agent add router $target_l3_id $router --l3
done
openstack network agent set $l3_id --disable
```

```
dhcp_id=$(openstack network agent list --host <host> --agent-type dhcp -f value -c ID)
#target_dhcp_id=$(openstack network agent list --host <target host> --agent-type dhcp -f value -c ID)
openstack network list --agent $dhcp_id -f value -c ID | while read network; do
  openstack network agent remove network $dhcp_id $network --dhcp
#  openstack network agent add network $target_dhcp_id $network --dhcp
done
```

```
kolla-ansible -i <inventory> stop --yes-i-really-really-mean-it
```

```
openstack network agent list --host <host> -f value -c ID | while read id; do
  openstack network agent delete $id
done
```

```
openstack compute service list --os-compute-api-version 2.53 --host <host> -f value -c ID | while read id; do
  openstack compute service delete --os-compute-api-version 2.53 $id
done
```

```
openstack compute service set <host> nova-compute --disable
```

```
openstack server list --all-projects --host <host> -f value -c ID | while read server; do
  openstack server migrate --live-migration $server
done
```

