# Ceph 相关
> PG = Placement Group
> PGP = Placement Group for Placement purpose
>
> pg_num = number of placement groups mapped to an OSD
>
> When pg_num is increased for any pool, every PG of this pool splits into half, but they all remain mapped to their parent OSD.
>
> Until this time, Ceph does not start rebalancing. Now, when you increase the pgp_num value for the same pool, PGs start to migrate from the parent to some other OSD, and cluster rebalancing starts. This is how PGP plays an important role.
> By Karan Singh
>
>
> PG是指定存储池存储对象的目录有多少个，PGP是存储池PG的OSD分布组合个数
> PG的增加会引起PG内的数据进行分裂，分裂到相同的OSD上新生成的PG当中
> PGP的增加会引起部分PG的分布进行变化，但是不会引起PG内对象的变动
>
> pg_num的增加会使原来PG中的对象均匀地分布到新建的PG中，原来的副本分布方式不变
> pgp_num的增加会使PG的分布方式发生变化，但是PG内的对象并不会变动
> pgp决定pg分布时的组合方式的变化



## 一、常用命令

##### 1、查询集群状态

```
# 健康状态
ceph -s
ceph health
ceph health detail
ceph pg stat
ceph pg dump_stuck unclean|grep unknown
ceph crash ls


# pool
ceph osd lspools
ceph osd pool ls
ceph osd pool ls detail

# osd
ceph osd tree

# 资源使用状态
ceph df
ceph df detail
ceph osd df
ceph osd df rbd
ceph osd df tree

# 其他
 ceph quorum_status --format json-pretty
 ceph osd pool get-quota volumes
 ceph osd pool get rbd crush_rule
```

##### 2、删除data，metadata池

```
ceph osd pool delete metadata metadata --yes-i-really-really-mean-it
ceph osd pool delete data data --yes-i-really-really-mean-it
```

##### 3、创建volumes存储池

```
ceph osd pool create volumes 4000 4000
```

##### 4、查询 volumes 池当前复制副本数量

```
ceph osd dump | grep 'replicated size' | grep volumes
```

##### 5、修改复制副本为 2 并验证

```
ceph osd pool set volumes size 2
ceph osd dump | grep 'replicated size' | grep volumes
```

##### 6、查看存储使用情况

```
ceph df
```

##### 7、查看集群得分

> 分数越低越好

```
ceph balancer eval

current cluster score 0.032004 (lower is better)
```

##### 8、修复

```
 ceph pg deep-scrub 4.b6
ceph pg repair 4.b6
```

##### 9、暂停与恢复osd对外访问

```
ceph osd pause
ceph osd unpause
```

##### 10、数据均衡调整

> 按利用率调整OSD的权重

```
ceph osd reweight-by-utilization
```

> 按归置组分布情况调整OSD的权重

```
ceph osd reweight-by-pg
```

##### 11、重新设置OSD权重

> crush 显示osd磁盘本身大小的标签

```
ceph osd crush reweight osd.8 1
ceph osd reweight osd.8 0.2
```

##### 12、删除pool

```
ceph tell mon.\* injectargs '--mon-allow-pool-delete=true'
ceph osd pool rm tmp tmp --yes-i-really-really-mean-it
ceph tell mon.\* injectargs '--mon-allow-pool-delete=false'
```

##### 13、rbdmap

```
apt -y install ceph-common

cat > /etc/ceph/rbdmap <<EOF
foo/bar1    id=admin,keyring=/etc/ceph/ceph.client.admin.keyring
foo/bar2    id=admin,keyring=/etc/ceph/ceph.client.admin.keyring,options='lock_on_read,queue_depth=1024'
EOF

systemctl enable rbdmap.service
systemctl restart rbdmap.service
```

# 二、cephadm

## 1、安装依赖

```
apt install -y cephadm

#curl --silent --remote-name --location https://github.com/ceph/ceph/raw/pacific/src/cephadm/cephadm
#chmod +x cephadm

cephadm add-repo --release pacific
cephadm install ceph-common
```

## 2、初始化

```
cephadm bootstrap --mon-ip 192.168.0.31
```

```
#ceph config assimilate-conf -i <input file> -o <output file>

cat <<EOF > initial-ceph.conf
[global]
    osd_pool_default_size = 2
    osd_pool_default_min_size = 1
    mon_osd_full_ratio = .90
    mon_osd_nearfull_ratio = .80
    public_network = 192.168.0.0/24
    cluster_network = 10.0.0.0/24
EOF

cephadm bootstrap --config initial-ceph.conf --mon-ip 192.168.0.31

cephadm shell -- ceph -s
#cephadm shell --fsid XXX -c /etc/ceph/ceph.conf -k /etc/ceph/ceph.client.admin.keyring
```

## 3、host 配置

```
ssh-copy-id -f -i /etc/ceph/ceph.pub node2
ssh-copy-id -f -i /etc/ceph/ceph.pub node3
```

```
ceph orch host add node2 mon
#ceph orch host rm node2

ceph orch host add node3 mon

ceph orch host label add node1 mon
#ceph orch host label rm node1 mon
```

```
ceph orch apply mon "node1,node2,node3"
ceph orch host ls
```

```
禁用自动部署 mon
ceph orch apply mon --unmanaged

不同网络添加 mon
ceph orch apply mon --unmanaged
ceph orch daemon add mon newhost1:10.1.2.123
ceph orch daemon add mon newhost2:10.1.2.0/24
```

## 4、osd 配置

```
ceph orch device ls
ceph orch apply osd --all-available-devices  
#ceph orch daemon add osd node1:/dev/sdb
```

```
删除 osd
ceph osd crush remove {name}
ceph auth del osd.{osd-num}
ceph osd stop {osd-num}
ceph osd rm {osd-num}
```

## 5、使用 cephfs

```
ceph fs volume create fs
```

> _netdev：表示当系统联网后再进行挂载操作
> noatime：这个参数来禁止记录最近一次访问时间戳，提高性能

```
使用 ceph-fuse
cephadm install ceph-fuse
#apt install -y ceph-fuse

ceph-fuse -m 192.168.0.31:6789,192.168.0.32:6789,192.168.0.33:6789 /fs

scp -r 192.168.0.61:/etc/ceph /etc
vim /etc/fstab
192.168.0.61:6789,192.168.0.62:6789,192.168.0.64:6789:/ /fs fuse.ceph ceph.id=admin,ceph.conf=/etc/ceph/ceph.conf,noatime,_netdev,defaults 0 0
```

```
ceph fs authorize *file_system_name* client.*client_name* /*specified_directory* rw

ceph fs authorize fs client.fs / rw
[client.fs]
        key = AQA8DK5g7FkeFhAA4B6gk6M5nc+17XVKzjv27Q==

#ceph auth rm client.fs

mount -t ceph :/ /fs -o name=fs,secret=AQA8DK5g7FkeFhAA4B6gk6M5nc+17XVKzjv27Q==

ceph fs authorize fs client.fs /yani rw
mount -t ceph :/yani /fs -o name=fs,secret=AQA8DK5g7FkeFhAA4B6gk6M5nc+17XVKzjv27Q==
```

```
使用 linux kernel

ssh 192.168.0.61 "ceph-authtool -p /etc/ceph/ceph.client.admin.keyring" > admin.key

mount -t ceph 192.168.0.61:6789,192.168.0.62:6789,192.168.0.64:6789:/ /fs -o name=admin,secret=AQDXXXXX==
#mount -t ceph :/ /fs -o name=admin,secret=AQDXXXXX==
#mount -t ceph 192.168.0.61:6789,192.168.0.62:6789,192.168.0.64:6789:/ /fs -o name=admin,secretfile=admin.key
vim /etc/fstab
192.168.0.61:6789,192.168.0.62:6789,192.168.0.64:6789:/ /fs ceph name=admin,secret=AQDXXXXX==,noatime,_netdev,defaults 0 0
#:/ /fs ceph name=admin,secret=AQDXXXXX==,noatime,_netdev,defaults 0 0
```

## 6、使用 rbd

```
1、创建 rbd
ceph osd pool create rbd
rbd pool init rbd

#rbd create --size {megabytes} {pool-name}/{image-name}

要创建一个1GB的映像test，该映像将信息存储在rbd Pool中
rbd create --size 1024 rbd/test

#开机自动映射
ls /etc/ceph/
ceph.client.guest.keyring  ceph.conf  rbdmap

vim /etc/ceph/rbdmap
#poolname/imagename     id=client,keyring=/etc/ceph/ceph.client.guest.keyring
rbd/test id=guest,keyring=/etc/ceph/ceph.client.guest.keyring

vim /etc/fstab
/dev/rbd0 /test xfs noatime,_netdev,defaults 0 0

# 查看已映射
rbd showmapped 

rbdmap unmap 
rbdmap unmap-all

创建块存储 pool 用户
ceph auth get-or-create client.libvirt mon 'profile rbd' osd 'profile rbd pool=rbd'
[client.libvirt]
        key = AQCUD65goRI3ARAAtMHtLBivJ39eikR4ZRjEgg==
        
rbd map rbd/test
/dev/rbd1

mkfs.xfs -f /dev/rbd1
mount /dev/rbd1 /mnt

2、创建快照
rbd snap create --snap mysnap rbd/test
#rbd snap create rbd/test@mysnap
rbd snap list rbd/test
rbd ls -l
rbd info rbd/test@mysnap

回滚
rbd snap rollback rbd/test@mysnap

删除快照
rbd snap remove rbd/test@mysnap

3、模板与克隆

查看设备是否支持创建快照模板，image-format 必须为2
rbd info rbd/test
rbd image 'test':
        size 1 GiB in 256 objects
        order 22 (4 MiB objects)
        snapshot_count: 0
        id: 170e5acc1f4a7
        block_name_prefix: rbd_data.170e5acc1f4a7
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        op_features: 
        flags: 
        create_timestamp: Wed May 26 11:13:38 2021
        access_timestamp: Wed May 26 11:13:38 2021
        modify_timestamp: Wed May 26 11:13:38 2021

rbd create rbd/test --size 1024 --image-format 2

把该块做成模板，首先要把做成模板的快照做成protect
rbd snap protect rbd/test@mysnap

可以去掉这个保护，但是这样的话就不能克隆
rbd snap unprotect rbd/test@mysnap

克隆
rbd clone rbd/test@mysnap rbd/test1

查看快照children
rbd children rbd/test@mysnap

这个时候的test1还是依赖test的镜像mysnap的，如果test的mysnap被删除或者怎么样，test1也不能够使用了，要想独立出去，就必须将父镜像的信息合并flattern到子镜像中
去掉快照的parent
rbd flatten rbd/test1
```

```
rbd create       创建块设备映像
rbd ls           列出 rbd 存储池中的块设备
rbd info         查看块设备信息
rbd diff         可以统计 rbd 使用量
rbd map          映射块设备
rbd showmapped   查看已映射块设备
rbd remove       删除块设备
rbd resize       更改块设备的大小
rbd export rbd/test /tmp/test                     导出rbd镜像
rbd import /tmp/test rbd/test --image-format 2    导入rbd镜像
```

## 7、性能测试

```
rados bench -p $poolname 60 write  [ --no-cleanup ]  #测试写性能，不删除写入的数据用来测试读取性能
rados bench -p $poolname 60 [ seq | rand ]  #测试读取性能
rados -p $poolname cleanup #清理测试写性能时写入的数据
```

## 8、其他

```
创建 pool
ceph osd pool create test_metadata 32 32

重新设置权重
ceph osd crush reweight osd.3 3.3
ceph osd reweight osd.4 0.3
ceph osd crush tree --show-shadow

获取 osdmap
ceph osd dump
ceph osd getmap -o osds.map
osdmaptool --print osds.map

获取 monmap
ceph mon dump
ceph mon getmap -o monmap.map
monmaptool --print monmap.map

获取 crushmap
ceph osd getcrushmap -o crushmap.dump
crushtool -d crushmap.dump -o crushmap.txt

crushtool -c crushmap.txt -o crushmap.done
ceph osd setcrushmap -i crushmap.done

新创建的 osd 不会自动加入 crushmap
osd crush update on start = false

ceph osd lspools
rbd ls -l
rbd remove rbd-name
rbd map disk01
rbd showmapped
ceph fs ls
ceph mds stat
```

## 9、FAQ

### 1、新版本特性问题

> rbd: image foo: image uses unsupported features: 0x38

```
rbd info foo

rbd image 'foo':
    size 1024 MB in 256 objects
    order 22 (4096 kB objects)
    block_name_prefix: rbd_data.10612ae7234b
    format: 2    features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
    flags:
layering: 支持分层
striping: 支持条带化 v2
exclusive-lock: 支持独占锁
object-map: 支持对象映射（依赖 exclusive-lock ）
fast-diff: 快速计算差异（依赖 object-map ）
deep-flatten: 支持快照扁平化操作
journaling: 支持记录 IO 操作（依赖独占锁）
```

```
rbd feature disable foo exclusive-lock, object-map, fast-diff, deep-flatten
```

### 2、源

```
echo "deb https://download.ceph.com/debian-octopus/ bionic main" > /etc/apt/sources.list.d/ceph.list
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E84AC2C0460F3994
apt update
apt install ceph-common
```

### 3、时间同步问题

```
vim /etc/chrony/chrony.conf
pool node1 iburst
allow 192.168.0.0/24

chronyc sources -v
```

### 4、ceph在容器启动前先启动

```
需要在每个节点的 service 中添加 docker.service，等 docker 先启动

vim /etc/systemd/system/ceph-de9c839a-bd3e-11eb-aefc-377d8c7eac71@.service
...
After=network-online.target local-fs.target time-sync.target docker.service
Wants=network-online.target local-fs.target time-sync.target docker.service
...
```

### 5、Possible data damage: 1 pg inconsistent

> 不一致性校验失败，数据不一致性（inconsisitent）指对象的大小不正确，恢复结束后某副本出现了对象丢失的情况，数据不一致会导致清理失败（scrub error）。Ceph在存储过程中，由于特殊原因，可能遇到对象信息大小和物理磁盘上实际数据大小不一致的情况，也会导致清理失败。

##### (1)、停止osd

>  先查看所要恢复pg

```
ceph health detail
```

```
ceph pg repair 4.3f
```

```
systemctl stop ceph-de9c839a-bd3e-11eb-aefc-377d8c7eac71@osd.14.service
```

##### (2)、刷入日志

> 需将相应数据复制进容器

```
docker cp /etc/ceph b951114a1b9d:/etc
docker cp /var/lib/ceph/de9c839a-bd3e-11eb-aefc-377d8c7eac71/osd.14/ b951114a1b9d:/var/lib/ceph/osd/ceph-14/
docker exec -it b951114a1b9d bash
```

```
ceph-osd -i 14 --flush-journal
```

##### (3)、启动osd

```
systemctl start ceph-de9c839a-bd3e-11eb-aefc-377d8c7eac71@osd.14.service
```

##### (4)、修复

```
ceph health detail
```

```
ceph pg repair 4.3f
```

# 三、ceph deploy 

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
##### 23、修改osd权重

```
ceph osd crush reweight osd.1 1
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

## 四、使用glance task-create上传镜像并将格式转为raw。

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

## 五、移除osd。

```
ceph osd tree
ceph osd out osd.0
ceph osd crush rm osd.0
ceph auth del osd.0
ceph osd rm osd.0
```

## 六、添加mon，移除mon

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

## 七、启动ceph自带监控界面

```
ceph mgr module enable dashboard
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

## 九、pg状态表

| **状态**           | **描述**                                                     |
| ------------------ | ------------------------------------------------------------ |
| Activating         | PG已经互联，但是还没有active。  *Peering**已经完成，**PG**正在等待所有**PG**实例同步并固化**Peering**的结果**(Info**、**Log**等**)* |
| Active             | 活跃态。PG可以正常处理来自客户端的读写请求  PG正常的状态应该是Active+Clean的。 |
| Backfilling        | Ceph正常扫描并同步整个PG的数据，而不是从最近的操作日志中推断需要同步的数据，Backfill（回填）是恢复的一个特殊状态。  正在后台填充态。 backfill是recovery的一种特殊场景，指peering完成后，如果基于当前权威日志无法对Up Set当中的某些PG实例实施增量同步(例如承载这些PG实例的OSD离线太久，或者是新的OSD加入集群导致的PG实例整体迁移) 则通过完全拷贝当前Primary所有对象的方式进行全量同步 |
| Backfill-toofull   | *backfill_toofull* backfill操作因为目标OSD容量超过指标而挂起  某个需要被Backfill的PG实例，其所在的OSD可用空间不足，Backfill流程当前被挂起 |
| *backfill_unfound* | Backfill因为没有找到对应对象而停止                           |
| Backfill-wait      | *backfill_wait* PG正在等待backfill被调度执行。  等待Backfill 资源预留 |
| Clean              | 干净态。PG内所有的对象都被正确的复制了对应的份数。  *PG**当前不存在待修复的对象，* *Acting Set**和**Up Set**内容一致，并且大小等于存储池的副本数* |
| Creating           | PG正在被创建                                                 |
| Deep               | Ceph 正在检查PG数据和checksums的一致性。  *PG**正在或者即将进行对象一致性扫描清洗* |
| Degraded           | PG中的一些对象还没有被复制到规定的份数。  *降级状态。**Peering**完成后，**PG**检测到任意一个**PG**实例存在不一致**(**需要被同步**/**修复**)**的对象，或者当前**ActingSet* *小于存储池副本数* |
| Down               | 一个包含必备数据的副本离线，所以PG也离线了  *Peering**过程中，**PG**检测到某个不能被跳过的**Interval**中**(**例如该**Interval**期间，**PG**完成了**Peering**，并且成功切换至**Active**状态，从而有可能正常处理了来自客户端的读写请求**),**当前剩余在线的**OSD**不足以完成数据修复* |
| Incomplete         | Ceph 探测到某一PG可能丢失了写入信息，或者没有健康的副本。如果你看到了这个状态，尝试启动有可能包含所需信息的失败OSD，  如果是erasure coded  pool的话，临时调整一下`min_size`也可能完成恢复  *Peering**过程中，* *由于* *a.* *无非选出权威日志* *b.* *通过**choose_acting**选出的**Acting Set**后续不足以完成数据修复，导致**Peering**无非正常完成* |
| Inconsistent       | Ceph检测到PG中对象的一份或多份数据不一致（比如对象大学不一直，或者恢复成功后对象依然没有等）  *不一致态。集群清理和深度清理后检测到**PG**中的对象在副本存在不一致，例如对象的文件大小不一致或**Recovery**结束后一个对象的副本丢失* |
| Peered             | PG已互联，但是不能向客户端提供服务，因为其副本数没达到本存储池的配置值（ min_size 参数）。  在此状态下恢复会进行，所以此PG最终能达到 min_size 。  Peering已经完成，但是PG当前ActingSet规模小于存储池规定的最小副本数(min_size) |
| Peering            | PG正在互联过程中。正在同步态。PG正在执行同步处理。  类似Raft的Leader选举，使一个PG内的OSD达成一致，不涉及数据迁移等操作。 |
| Recovering         | 正在恢复态。集群正在执行迁移或同步对象和他们的副本           |
| Recovering-wait    | 等待recovery资源预留。PG正在等待恢复被调度执行。             |
| recovery_toofull   | 恢复操作因为目标OSD容量超过指标而挂起。                      |
| *recovery_unfound* | 恢复因为没有找到对应对象而停止。                             |
| Remapped           | PG被临时分配到了和CRUSH所指定的不同的OSD上。  重新映射态。PG活动集任何的一个改变，数据发生从老活动集到新活动集的迁移。在迁移期间还是用老的活动集中的主OSD处理客户端请求，一旦迁移完成新活动集中的主OSD开始处理 |
| Repair             | Ceph正在检查PG并且修复所有发现的不一致情况（如果有的话）  *PG**在执行**Scrub**过程中，如果发现存在不一致的对象，并且能够修复，则自动进行修复状态* |
| Scrubbing          | Ceph 正在检查PG metadata的一致性。  PG正在或者即将进行对象一致性扫描 |
| Unactive           | 非活跃态。PG不能处理读写请求                                 |
| Unclean            | 非干净态。PG不能从上一个失败中恢复                           |
| Stale              | PG状态未知，从PG mapping更新后Monitor一直没有收到更新  *未刷新态。**PG**状态没有被任何**OSD**更新，这说明所有存储这个**PG**的**OSD**可能挂掉**,* *或者**Mon**没有检测到**Primary**统计信息**(**网络抖动**)* |
| *snaptrim*         | 正在对快照做Trim操作。                                       |
| *snaptrim_Wait*    | Trim操作等待被调度执行                                       |
| *snaptrim_Error*   | Trim操作因为错误而停止                                       |
| Undersized         | 该PG的副本数量小于存储池所配置的副本数量。  PG当前Acting Set小于存储池副本数。ceph默认3副本，min_size参数通常为2，即副本数>=2时就可以进行IO，否则阻塞IO。 |
| forced_recovery    | 用户指定的PG高优先级恢复                                     |
| *forced_backfill*  | 用户指定的高优先级backfill                                   |

## 十、常见问题

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

##### 7、application not enabled on 1 pool(s)

```
ceph osd pool application enable k8s rbd
```

##### 8、手动刷新 ceph-cache

```
rados -p cache cache-flush-evict-all
```

