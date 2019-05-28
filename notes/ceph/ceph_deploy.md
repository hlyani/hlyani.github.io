# Ceph 部署
## 一、ceph deploy 部署ceph
##### 1、添加release.key
```
apt-get update && apt-get -y upgrade

wget -q -O- 'https://download.ceph.com/keys/autobuild.asc' | apt-key add -
```
##### 2、查看当前系统的发行版本信息
```
lsb_release -sc
```
##### 3、添加Ceph软件包源，用Ceph稳定版（如cuttlefish、dumpling、emperor、firefly、hammer、jewel等）替换掉jewel
```
echo deb http://mirrors.aliyun.com/ceph/debian-jewel/ $(lsb_release -sc) main | tee /etc/apt/sources.list.d/ceph.list

apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E84AC2C0460F3994
```
##### 4、更新源仓库并安装Ceph
```
apt-get update && apt-get -y install ceph-deploy
```
##### 5、将ceph deploy源换为国内源
```
vim /usr/lib/python2.7/dist-packages/ceph_deploy/hosts/debian/install.py

sed -i 's/{protocol}:\/\/download.ceph.com/http:\/\/mirrors.aliyun.com\/ceph/g' /usr/lib/python2.7/dist-packages/ceph_deploy/hosts/debian/install.py
sed -i 's/{protocol}:\/\/download.ceph.com/http:\/\/mirrors.aliyun.com\/ceph/g' /usr/lib/python2.7/dist-packages/ceph_deploy/hosts/debian/install.py
sed -i 's/download.ceph.com/mirrors.tuna.tsinghua.edu.cn\/ceph/g' /usr/lib/python2.7/dist-packages/ceph_deploy/util/constants.py
```
##### 6、安装NTP（以免因时钟漂移导致故障）
```
apt-get -y install ntp
```
##### 7、安装 SSH 服务器
```
apt-get -y install openssh-server
```
##### 8、配置hosts
```
echo "
172.18.20.181 ceph-node1
172.18.20.182 ceph-node2
172.18.20.183 ceph-node3" | tee >> /etc/hosts
```
##### 9、创建部署 CEPH 的用户（可不用）
```
1、在各 Ceph 节点创建新用户。
useradd -d /home/myCeph -m myCeph
passwd myCeph
2、确保各 Ceph 节点上新创建的用户都有 sudo 权限。
echo "myCeph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/myCeph
chmod 0440 /etc/sudoers.d/myCeph
```
##### 10、允许无密码 SSH 登录
```
1、生成 SSH 密钥对，但不要用 sudo 或 root 用户。提示 “Enter passphrase” 时，直接回车，口令即为空：
ssh-keygen
2、把公钥拷贝到各 Ceph 节点，把下列命令中的 myCeph 替换成前面创建部署 Ceph 的用户里的用户名。
ssh-copy-id myCeph@node1
3、（推荐做法）修改 ceph-deploy 管理节点上的 ~/.ssh/config 文件，这样 ceph-deploy 就能用你所建的用户名登录 Ceph 节点了，而无需每次执行 ceph-deploy 都要指定 --username {username} 。这样做同时也简化了 ssh 和 scp 的用法。把 {username} 替换成你创建的用户名。

Host node1
   Hostname node1
   User myCeph
   
ssh-copy-id ceph-node1
```
##### 11、安装Ceph
```
ceph-deploy install controller compute{2,3,4,5,6,7,8}
```
##### 12、创建集群
```
ceph-deploy new controller compute{2,3,4,5}
```
##### 13、修改配置文件 ，把下面这行加入 [global] 段：
```
echo "osd crush chooseleaf type = 0" >> ceph.conf
        echo "osd pool default size = 1" >> ceph.conf
        echo "osd journal size = 100" >> ceph.conf
```
##### 14、配置初始monitor(s)、并收集所有秘钥：
```
#ceph-deploy mon create-initial

ceph-deploy mon create controller compute{2,3,4,5}

ceph-deploy gatherkeys controller compute{2,3,4,5}
```
##### 15、擦净磁盘
```
ceph-deploy disk zap controller:sd{b,c,d,e} compute2:sd{b,c,d} compute{3,4,5,6,7,8}:sd{b,c,d,e}
```
##### 16、准备磁盘
```
ceph-deploy osd prepare controller:sd{b,c,d,e} compute2:sd{b,c,d} compute{3,4,5,6,7,8}:sd{b,c,d,e}

ceph-deploy osd prepare ceph-node1:xvdb ceph-node1:xvdc ceph-node1:xvde ceph-node2:xvdb ceph-node2:xvdc ceph-node2:xvde ceph-node3:xvdb ceph-node3:xvdc ceph-node3:xvde
```
##### 17、激活磁盘
```
ceph-deploy osd activate controller:sd{b1,c1,d1,e1} compute2:sd{b1,c1,d1} compute{3,4,5,6,7,8}:sd{b1,c1,d1,e1}
```
##### 18、复制配置文件到所有节点，用ceph-deploy 把配置文件和 admin 密钥拷贝到管理节点和 Ceph 节点，这样每次执行 Ceph 命令行时就无需指定 monitor 地址和 ceph.client.admin.keyring 了。
```
ceph-deploy admin node1

ceph-deploy --overwrite-conf admin ceph-node1 ceph-node2 ceph-node3
```
##### 19、确保你对 ceph.client.admin.keyring 有正确的操作权限。
```
chmod +r /etc/ceph/ceph.client.admin.keyring
```
##### 20、检查集群的健康状况
```
ceph health
ceph -s
ceph osd tree 
```
##### 21、清除部署
```
ceph-deploy purge ceph-node1 ceph-node2 ceph-node3
ceph-deploy purgedata ceph-node1 ceph-node2 ceph-node3
ceph-deploy forgetkeys

ceph osd pool delete rbd rbd --yes-i-really-really-mean-it
```
##### 22、修改crushmap
```
1、获得 crush map，获得默认 crushmap (加密)
ceph osd getcrushmap -o crushmap.dump
2、转换 crushmap 格式 (加密 -> 明文格式)
crushtool -d crushmap.dump -o crushmap.txt
3、转换 crushmap 格式(明文 -> 加密格式)
crushtool -c crushmap.txt -o crushmap.done
4、重新使用新 crushmap
ceph osd setcrushmap -i crushmap.done
```
## 二、将ceph配置文openstack的后端配置
##### 1、创建pool，初始化pool。

```
ceph osd pool create volumes 128
ceph osd pool create images 128
ceph osd pool create backups 128
ceph osd pool create vms 128

rbd pool init volumes
rbd pool init images
rbd pool init backups
rbd pool init vms
```

##### 2、在glance-api节点安装以下包。

```
apt-get -y install python-rbd
yum -y install python-rbd
```

##### 3、在nova-compute, cinder-backup,cinder-volume安装以下包。

```
apt-get -y install ceph-common
yum -y install ceph-common
```

##### 4、如果开启了 cephx authentication, 需要为nova、cinder、glance创建新用户。

```
#ceph auth get-or-create client.glance mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=images, allow rwx pool=images.cache'

ceph auth get-or-create client.glance mon 'profile rbd' osd 'profile rbd pool=images'

ceph auth get-or-create client.cinder mon 'profile rbd' osd 'profile rbd pool=volumes, profile rbd pool=vms, profile rbd-read-only pool=images'

ceph auth get-or-create client.cinder-backup mon 'profile rbd' osd 'profile rbd pool=backups'
```

##### 5、为相应节点的client.cinder, client.glance, client.cinder-backup添加keyrings，改变它们的所有权。

```
ceph auth get-or-create client.glance | ssh {your-glance-api-server} tee /etc/ceph/ceph.client.glance.keyring
ssh {your-glance-api-server} chown glance:glance /etc/ceph/ceph.client.glance.keyring
ceph auth get-or-create client.cinder | ssh {your-volume-server} tee /etc/ceph/ceph.client.cinder.keyring
ssh {your-cinder-volume-server} chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring
ceph auth get-or-create client.cinder-backup | ssh {your-cinder-backup-server} tee /etc/ceph/ceph.client.cinder-backup.keyring
ssh {your-cinder-backup-server} chown cinder:cinder /etc/ceph/ceph.client.cinder-backup.keyring
```

##### 6、运行了nova-compute的节点需要keyring文件。

```
ceph auth get-or-create client.cinder | ssh {your-nova-compute-server} tee /etc/ceph/ceph.client.cinder.keyring
```

##### 7、需要将client.cinder用户的秘钥配置到libvirt中，libvirt进程从cinder挂载块设备时，需要有权限访问ceph集群。获取并拷贝一个临时的secret key到所有运行的nova-compute节点。

```
ceph auth get-key client.cinder | ssh {your-compute-node} tee client.cinder.key
```

##### 8、然后，在compute节点，添加secret key到libvirt，再删除临时的secret key。

```
# 保存uuid，后面会用到
uuidgen
457eb676-33da-42ec-9a8c-9293d545c337

cat > secret.xml <<EOF
<secret ephemeral='no' private='no'>
  <uuid>457eb676-33da-42ec-9a8c-9293d545c337</uuid>
  <usage type='ceph'>
    <name>client.cinder secret</name>
  </usage>
</secret>
EOF
virsh secret-define --file secret.xml
Secret 457eb676-33da-42ec-9a8c-9293d545c337 created

virsh secret-set-value --secret 457eb676-33da-42ec-9a8c-9293d545c337 --base64 $(cat client.cinder.key) && rm client.cinder.key secret.xml
```

##### 9、修改glance-api.conf。

```
[glance_store]
stores = rbd
default_store = rbd
rbd_store_pool = images
rbd_store_user = glance
rbd_store_ceph_conf = /etc/ceph/ceph.conf
rbd_store_chunk_size = 8
...
# 启用copy-on-write
[DEFAULT]
show_image_direct_url = True
...
[paste_deploy]
flavor = keystone
```

##### 10、修改cinder.conf。

```
[DEFAULT]
...
enabled_backends = ceph
glance_api_version = 2
...
# 配置cinder backup
backup_driver = cinder.backup.drivers.ceph
backup_ceph_conf = /etc/ceph/ceph.conf
backup_ceph_user = cinder-backup
backup_ceph_chunk_size = 134217728
backup_ceph_pool = backups
backup_ceph_stripe_unit = 0
backup_ceph_stripe_count = 0
restore_discard_excess_bytes = true
...
[ceph]
volume_driver = cinder.volume.drivers.rbd.RBDDriver
volume_backend_name = ceph
rbd_pool = volumes
rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_flatten_volume_from_snapshot = false
rbd_max_clone_depth = 5
rbd_store_chunk_size = 4
rados_connect_timeout = -1
rbd_user = cinder
rbd_secret_uuid = 457eb676-33da-42ec-9a8c-9293d545c337
```

##### 11、配置nova.conf将nova挂载到ceph rbd块设备。

```
[libvirt]
...
images_type = rbd
images_rbd_pool = vms
images_rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_user = cinder
rbd_secret_uuid = 457eb676-33da-42ec-9a8c-9293d545c337
disk_cachemodes="network=writeback"
```

##### 12、重启服务

```
glance-control api restart
service nova-compute restart
service cinder-volume restart
service cinder-backup restart

service openstack-glance-api restart
service openstack-nova-compute restart
service openstack-cinder-volume restart
service openstack-cinder-backup restart
```

##### 13、使用块设备

```
cinder create --image-id {id of image} --display-name {name of volume} {size of volume}

qemu-img convert -f {source-format} -O {output-format} {source-filename} {output-filename}
qemu-img convert -f qcow2 -O raw precise-cloudimg.img precise-cloudimg.raw
```

## 三、使用glance task-create上传镜像并将格式转为raw。

##### 1、编辑glance-api.conf。

```
[task]
task_executor = taskflow
work_dir=/tmp

[taskflow_executor]
engine_mode = serial
max_workers = 10
conversion_format=raw
```

##### 2、重启服务

```
glance-control api restart

service openstack-glance-api restart
```

##### 3、上传镜像

```
glance task-create --type import --input '{"import_from_format": "qcow2", "import_from": "http://172.18.20.160/cirros-0.3.4-x86_64-disk.img", "image_properties": {"name": "cirros-RAW", "disk_format": "qcow2", "container_format": "bare"}}'
```

## 四、移除osd。

```
ceph osd tree
ceph osd out osd.0
ceph osd crush rm osd.0
ceph auth del osd.0
ceph osd rm osd.0
```

## 五、添加mon，移除mon

```
ceph-deploy install ctrl
ceph-deploy mon create ctrl 
ceph-deploy mon add ctrl
ceph-deploy mon create-initial
ceph quorum_status --format json-pretty

ceph-deploy mon destroy comp-3
ceph quorum_status --format json-pretty

ceph-deploy gatherkeys comp-1
```

## 六、启动ceph自带监控界面

```
ceph mgr module enable dashboard
```

## 七、常用命令

```
#查询当前池方法
ceph osd lspools
ceph osd pool ls
ceph osd pool ls detail

#删除data,metadata池
ceph osd pool delete metadata metadata --yes-i-really-really-mean-it
ceph osd pool delete data data --yes-i-really-really-mean-it

#创建 volumes 存储池
ceph osd pool create volumes 4000 4000

#查询 volumes 池当前复制副本数量
ceph osd dump | grep 'replicated size' | grep volumes

#修改复制副本为 2 并验证
ceph osd pool set volumes size 2
ceph osd dump | grep 'replicated size' | grep volumes

#查看存储使用情况
ceph df
```

## 八、kolla 中ceph相关

```
parted /dev/xvdb -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP 1 -1

修改globalsl.yaml
enable_ceph: "yes"
ceph_enable_cache: "yes"
ceph_cache_mode: "writeback"

2个240 固态(一个做journal，一个做cache)

【cache】
parted /dev/xvdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_CACHE_BOOTSTRAP 1 -1

【journal】
parted /dev/xvdd \
-s mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDB_J 0% 5G \
-s mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC_J 5G 10G \
-s mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC_J 10G 15G \
-s mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC_J 15G 20G \
-s mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC_J 20G 25G \
-s mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC_J 25G 30G \
-s mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC_J 30G 35G \
-s mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC_J 35G 40G \
-s mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC_J 40G 45G

9个2T SATA

parted /dev/vdb -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDB 1 -1
parted /dev/vdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC 1 -1
parted /dev/vdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC 1 -1
parted /dev/vdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC 1 -1
parted /dev/vdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC 1 -1
parted /dev/vdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC 1 -1
parted /dev/vdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC 1 -1
parted /dev/vdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC 1 -1
parted /dev/vdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_VDC 1 -1
```

## 九、常见问题

##### 1、日志盘bug

```
ceph-disk list

vim /usr/lib/python2.7/dist-packages/ceph_disk/main.py

sgdisk --typecode=6:4fbd7e29-9d25-41b8-afd0-062c0ceff05d -- /dev/sda

sgdisk --typecode=1:45b0969e-9b03-4f30-b4c6-b4b80ceff106 -- /dev/ssd

chown -R ceph:ceph /dev/nvme0n1p2

parted /dev/sdg
mklabel        GPT

mkpart cache xfs 0G 380G
mkpart sdb_journal xfs 380G 385G
```

##### 2、NO_PUBKEY E84AC2C0460F3994

```
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E84AC2C0460F3994
```

##### 3、** ERROR: error creating empty object store in /var/lib/ceph/tmp/mnt.blV9DA: (13) Permission denied

```
将ceph集群需要使用的所有磁盘权限，所属用户、用户组改给ceph
 
chown ceph:ceph /dev/xvdb
```

##### 4、clock skew detected on mon.ceph-node2, mon.ceph-node3  Monitor clock skew detected 

```
/etc/init.d/ntpd stop

service chrony stop
ntpdate time.nist.gov
service chrony start
```

##### 5、 OSD_SCRUB_ERRORS: 1 scrub errors；PG_DAMAGED: Possible data damage: 1 pg inconsistent

```
docker exec ceph_mon sh -c 'for i in "`ceph health detail|grep -vE \"(PG_DAMAGED|HEALTH_ERR|OSD_SCRUB_ERRORS)\"|awk \"{print $2}\"`";do ceph pg repair $i ;done'

for i in `ceph health  detail|grep 'active+clean+inconsistent'|awk '{print $2}'`;do ceph pg repair $i ;done
ceph -s
```

##### 6、往下刷缓存

```
rados -p images-cache cache-flush-evict-all
rados -p volumes-cache cache-flush-evict-all
rados -p backups-cache cache-flush-evict-all
rados -p vms-cache cache-flush-evict-all
rados -p gnocchi-cache cache-flush-evict-all
```


















