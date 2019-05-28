# kolla镜像构建

[官网地址](https://docs.openstack.org/kolla/latest/admin/image-building.html)

##### 1、环境准备

```
git clone https://github.com/openstack/kolla.git
cd kolla
git checkout -b queens origin/stable/queens
pip install tox
tox -e genconfig
```

##### 2、配置

> build配置文件 kolla/etc/kolla/kolla-build.conf

```kolla-build.conf
[DEFAULT]
base = centos
base_tag = 7.4.1708

#tarballs_base = http://tarballs.openstack.org
install_type = sourc
tag = queens
logs_dir = /home/hl/kolla_build/kolla/log

[ceilometer-base]
location = http://127.0.0.1/tars/ceilometer-10.0.0.tar.gz

[cinder-base]
location = http://127.0.0.1/tars/cinder-12.0.1.tar.gz

[horizon]
location = http://127.0.0.1/tars/horizon-13.0.0.tar.gz

[horizon-plugin-fwaas-dashboard]
#location = $tarballs_base/neutron-fwaas-dashboard/neutron-fwaas-dashboard-1.3.0.tar.gz
location = http://127.0.0.1/tars/neutron-fwaas-dashboard-1.3.0.tar.gz

[horizon-plugin-neutron-lbaas-dashboard]
location = http://192.168.110.12/tars/neutron-lbaas-dashboard-4.0.0.tar.gz

[horizon-plugin-trove-dashboard]
location = http://192.168.110.12/tars/trove-dashboard-10.0.0.tar.gz

[neutron-base]
location = http://192.168.110.12/tars/neutron-12.0.4.tar.gz

[nova-base]
location = http://192.168.110.12/tars/nova-17.0.2.tar.gz

```

> 自定义编译某个组件tar包，将http://tarballs.openstack.org中的相应包，下载，解压，上传到自己git仓库

```
cd /var/www/html/git/nova/
git pull origin master
rm -rf nova-17.0.2.tar.gz
git archive --format=tar.gz --prefix=nova-17.0.2/ master >nova-17.0.2.tar.gz
\cp -rf /var/www/html/git/nova/nova-17.0.2.tar.gz /var/www/html/tars/

cd /var/www/html/git/cinder/
git pull origin master
rm -rf cinder-12.0.1.tar.gz
git archive --format=tar.gz --prefix=cinder-12.0.1/ master >cinder-12.0.1.tar.gz
\cp -rf /var/www/html/git/cinder/cinder-12.0.1.tar.gz /var/www/html/tars/
```



> build需要的项目列表文件 ~/kp.list

```kp.list
nova neutron keystone glance cinder ceph magnum horizon toolbox fluentd openvswitch mariadb memcached rabbitmq keepalived haproxy cron iscsi
```

##### 3、修改kolla/docker/base中的源为国内源

```
略
```

##### 4、编译

```
cd kolla
cat ~/kp.list | xargs ./tools/build.py --config-file ./etc/kolla/kolla-build.conf
```

##### 5、kolla image

```
kolla/centos-source-elasticsearch
kolla/centos-source-kibana
kolla/centos-source-ceilometer-compute
kolla/centos-source-ceilometer-notification
kolla/centos-source-ceilometer-central
kolla/centos-source-heat-api
kolla/centos-source-heat-api-cfn
kolla/centos-source-heat-engine
kolla/centos-source-cinder-volume
kolla/centos-source-cinder-backup
kolla/centos-source-cinder-api
kolla/centos-source-cinder-scheduler
kolla/centos-source-neutron-openvswitch-agent
kolla/centos-source-nova-compute
kolla/centos-source-neutron-lbaas-agent
kolla/centos-source-neutron-l3-agent
kolla/centos-source-nova-api
kolla/centos-source-neutron-metadata-agent
kolla/centos-source-neutron-dhcp-agent
kolla/centos-source-nova-ssh
kolla/centos-source-neutron-server
kolla/centos-source-nova-placement-api
kolla/centos-source-nova-novncproxy
kolla/centos-source-nova-scheduler
kolla/centos-source-nova-consoleauth
kolla/centos-source-nova-conductor
kolla/centos-source-keystone
kolla/centos-source-panko-api
kolla/centos-source-glance-api
kolla/centos-source-glance-registry
kolla/centos-source-gnocchi-statsd
kolla/centos-source-gnocchi-api
kolla/centos-source-gnocchi-metricd
kolla/centos-source-collectd
kolla/centos-source-ceph-mon
kolla/centos-source-ceph-rgw
kolla/centos-source-ceph-mgr
kolla/centos-source-ceph-osd
kolla/centos-source-openvswitch-db-server
kolla/centos-source-openvswitch-vswitchd
kolla/centos-source-kolla-toolbox
kolla/centos-source-rabbitmq
kolla/centos-source-nova-libvirt
kolla/centos-source-fluentd
kolla/centos-source-cron
kolla/centos-source-mariadb
kolla/centos-source-keepalived
kolla/centos-source-haproxy
kolla/centos-source-memcached
```

