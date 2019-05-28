# devstack相关

##### 1、添加stack user，devstack需要运行在非root用户。

```
 sudo useradd -s /bin/bash -d /opt/stack -m stack
```

##### 2、给用户提供sudo权限。

```
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo su - stack
```

##### 3、下载 devstack

```
git clone https://git.openstack.org/openstack-dev/devstack
cd devstack
```

##### 4、修改pip源、软件源

```
略
```

##### 5、创建local.conf文件，以下是mitaka版本的配置文件。

```
[[local|localrc]]
# use TryStack git mirror
GIT_BASE=http://git.trystack.cn
NOVNC_REPO=http://git.trystack.cn/kanaka/noVNC.git
SPICE_REPO=http://git.trystack.cn/git/spice/spice-html5.git

# Define images to be automatically downloaded during the DevStack built process.
DOWNLOAD_DEFAULT_IMAGES=False
IMAGE_URLS="http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img"

# Credentials
DATABASE_PASSWORD=0127
ADMIN_PASSWORD=0127
SERVICE_PASSWORD=0127
SERVICE_TOKEN=0127
RABBIT_PASSWORD=0127
#FLAT_INTERFACE=eth0

HOST_IP=172.18.20.157
SERVICE_HOST=172.18.20.157
MYSQL_HOST=172.18.20.157
RABBIT_HOST=172.18.20.157
GLANCE_HOSTPORT=172.18.20.157:9292

# Database Backend MySQL
enable_service mysql

# RPC Backend RabbitMQ
enable_service rabbit

# Enable Keystone - OpenStack Identity Service
enable_service key

# Horizon - OpenStack Dashboard Service
enable_service horizon

# Enable Glance - OpenStack Image service
enable_service g-api g-reg

# Enable Cinder - Block Storage service for OpenStack
VOLUME_GROUP="cinder-volumes"
enable_service cinder c-api c-vol c-sch c-bak

# Enable Heat (orchestration) Service
enable_service heat h-api h-api-cfn h-api-cw h-eng

# Enable Tempest - The OpenStack Integration Test Suite
enable_service tempest

# Enable NoVNC
enable_service n-novnc

# Enabling Neutron (network) Service
disable_service n-net
enable_service q-svc
enable_service q-agt
enable_service q-dhcp
enable_service q-l3
enable_service q-meta
enable_service q-metering
enable_service neutron

## Neutron options
Q_USE_SECGROUP=True
FLOATING_RANGE="172.18.20.0/24"
FIXED_RANGE="10.0.1.0/24"
NETWORK_GATEWAY="10.0.1.254"
Q_FLOATING_ALLOCATION_POOL=start=172.18.20.180,end=172.18.20.186
PUBLIC_NETWORK_GATEWAY="172.18.20.254"
Q_L3_ENABLED=True
PUBLIC_INTERFACE=eth0
Q_USE_PROVIDERNET_FOR_PUBLIC=True
OVS_PHYSICAL_BRIDGE=br-ex
PUBLIC_BRIDGE=br-ex
OVS_BRIDGE_MAPPINGS=public:br-ex

# VLAN configuration.
Q_PLUGIN=ml2
ENABLE_TENANT_VLANS=True

# Branches
KEYSTONE_BRANCH=stable/mitaka
NOVA_BRANCH=stable/mitaka
NEUTRON_BRANCH=stable/mitaka
GLANCE_BRANCH=stable/mitaka
CINDER_BRANCH=stable/mitaka
HEAT_BRANCH=stable/mitaka
HORIZON_BRANCH=stable/mitaka

# Select Keystone's token format
# Choose from 'UUID', 'PKI', or 'PKIZ'
# INSERT THIS LINE...
KEYSTONE_TOKEN_FORMAT=${KEYSTONE_TOKEN_FORMAT:-UUID}
KEYSTONE_TOKEN_FORMAT=$(echo ${KEYSTONE_TOKEN_FORMAT} | tr '[:upper:]' '[:lower:]')

# Work offline
#OFFLINE=True
# Reclone each time
RECLONE=yes

# Logging
DEST=/home/stack.mitaka
LOGFILE=/home/stack.mitaka/logs/stack.sh.log
VERBOSE=True
LOG_COLOR=True
SCREEN_LOGDIR=/home/stack.mitaka/logs
```

##### 6、运行安装

```
./stack.sh
```

##### 7、官网地址

[devstack](https://docs.openstack.org/devstack/latest/)