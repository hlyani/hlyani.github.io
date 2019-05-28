# kolla部署相关

## 一、环境要求

* 两张网卡
* 除系统盘外，至少1块磁盘

## 二、安装

[kolla deploy](https://docs.openstack.org/project-deploy-guide/kolla-ansible/ocata/quickstart.html#deploy-kolla)

##### 1、安装相关依赖

```
#CentOS
yum -y install epel-release
yum -y install python-pip
pip install -U pip
yum install python-devel libffi-devel gcc openssl-devel

#Ubuntu
apt-get update
apt-get -y install python-pip
pip install -U pip
apt-get install python-dev libffi-dev gcc libssl-dev
```

##### 2、安装ansible

```
#CentOS
yum -y install ansible

#Ubuntu
apt-get install ansible

pip install -U ansible
```

##### 3、安装docker

[aliyun 安装docker-ce](https://yq.aliyun.com/articles/110806)

[tsinghua 安装docker-ce](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)

```
curl -sSL https://get.docker.io | bash

or

curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

docker --version
docker info
```

##### 4、防止部署neutron-dhcp-agent时失败，配置kolla.conf

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

##### 5、安装python-docker-py

```
yum -y install python-docker-py
pip install -U docker-p
```

##### 6、安装ntp

```
# CentOS 7
yum install ntp
systemctl enable ntpd.service
systemctl start ntpd.service

apt-get -y install ntp
```

##### 7、禁用宿主机的libvirt

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

##### 8、禁用防火墙

```
setenforce 0
systemctl stop firewalld
systemctl disable firewalld
```

##### 9、安装kolla-ansible

```
pip install kolla-ansible
```

##### 10、复制配置文件globals.yml、password.yml到/etc中

```
#CentOS
cp -r /usr/share/kolla/etc_examples/kolla /etc/kolla/

#Ubuntu
cp -r /usr/local/share/kolla/etc_examples/kolla /etc/kolla/

#CentOS
cp /usr/share/kolla-ansible/ansible/inventory/* ~/

#Ubuntu
cp /usr/local/share/kolla-ansible/ansible/inventory/* ~/
```

##### 11、配置网络

```
bond0 宿主机网卡，管理网网卡（internel、存储。。。）
bond1：管理网外部网卡（public,vip）
bond2：外网网卡（浮动ip等，up网卡，不用配置ip）
bond3:存储网络、tunnel网络

ip addr show
ip link set bond2 up
```

##### 12、配置磁盘

```
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

##### 13、修改globals.yml

```
kolla_base_distro: "centos"
kolla_install_type: "source"
openstack_release: "queens"
kolla_internal_vip_address: "10.10.32.10"
kolla_external_vip_address: "10.10.31.10"
docker_registry: "10.10.32.11:4000"
network_interface: "bond0"
kolla_external_vip_interface: "bond1"
storage_interface: "bond3"
tunnel_interface: "bond3"
neutron_external_interface: "bond2"
keepalived_virtual_router_id: "61"
enable_ceilometer: "yes"
enable_central_logging: "yes"
enable_ceph: "yes"
enable_ceph_rgw: "yes"
enable_cinder: "yes"
enable_collectd: "yes"
enable_gnocchi: "yes
enable_heat: "yes"
enable_horizon: "no"
enable_neutron_lbaas: "yes"
enable_neutron_fwaas: "yes"
enable_neutron_qos: "yes"
enable_neutron_agent_ha: "yes"
enable_panko: "yes"
ceph_enable_cache: "yes"
enable_ceph_rgw_keystone: "yes"
ceph_pool_pg_num: 256
ceph_pool_pgp_num: 256
glance_backend_file: "no"
glance_backend_ceph: "yes"
panko_database_type: "mysql"
cinder_backend_ceph: "{{ enable_ceph }}"
cinder_volume_group: "cinder-volumes"
cinder_backup_driver: "ceph"
nova_backend_ceph: "{{ enable_ceph }}"
nova_compute_virt_type: "kvm"
ceph_enable_journal: "yes"
```

##### 14、添加config文件（可选）

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

##### 15、生成密码

```
kolla-genpwd
```

##### 16、修改multinode文件

```
[control]
node1
node2
node3
[network]
node1
node2
node3
[inner-compute]

# external-compute is the groups of compute nodes which can reach
# outside
[external-compute]
node1
node2
node3
[compute:children]
inner-compute
external-compute

[monitoring]
node1
node2
node3
[storage]
node1
node2
node3
```

##### 17、初始化（可选）

```
kolla-ansible -i multinode bootstrap-servers
```

##### 18、部署前检查（可选）

```
kolla-ansible -i multinode prechecks
```

##### 19、拉去镜像（可选）

```
kolla-ansible -i multinode pull
```

##### 20、部署

```
kolla-ansible -i multinode deploy

# 只部署某些组件
kolla-ansible -i multinode deploy --tags="haproxy"

# 过滤部署某些组件
kolla-ansible -i multinode deploy --skip-tags="haproxy"
```

##### 21、生成admin-openrc.sh

```
kolla-ansible post-deploy
```

##### 22、调整日志

```
ln -sf /var/lib/docker/volumes/kolla_logs/_data/ /var/log/kolla
```

##### 23、运行init-runonce

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

