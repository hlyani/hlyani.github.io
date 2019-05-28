# openstack中遇到的问题

[TOC]

## 一、配置ceilometer采集间隔

#####  ceilometer相关配置
```
1、修改ceilometer采集数据时间间隔。
sed -i '/^-\ interval: /c -\ interval:\ 60' /etc/kolla/ceilometer-compute/polling.yaml

docker restart ceilometer_compute

2、使用配置在重新部署kolla时依然生效。
cp /etc/kolla/ceilometer-compute/polling.yaml /etc/kolla/config/
```
##### gnocchi相关配置
```
将基于low策略的metric更改基于medium：
1，删除原有 policy-rule 。
gnocchi archive-policy-rule delete default
2，创建新的 policy-rule 。
gnocchi archive-policy-rule create -a medium -m "*" default
-a 指明<POLICY-NAME>。
3，删除所有metric，gnocchi会重新自动的建立基于新policy的metric 。
for i in `gnocchi metric list -c id -f value`; do gnocchi metric delete $i; done
```
##### gnocchi相关配置
```
gnocchi archive-policy create -d granularity:0:03:00,points:3360 <POLICY-NAME>
granularity为时间频率
points为 保存周期/时间频率
```

## 二、ceph bug，虚拟机异常关机之后启动失败
######  由于 ceph版本过新，虚拟机异常关机之后导致启动失败，需要手动配置一下 caps，bug地址：https://bugs.launchpad.net/kolla-ansible/+bug/1760065 。
```
1、配置完成需要重启 osd, 首先将 osd 标记为 unout。
# docker exec ceph_mon ceph osd set noout

noout is set

2、更新 client.nova 的 caps，首先获取目前的 caps, 记住 caps osd 的值。
# docker exec ceph_mon ceph auth get client.nova

[client.nova]
        key = AQBu0BdbeUD1DBAAgBiKOqKCK71j2T+gC0xQRw==
        caps mon = "allow r"
        caps osd = "allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=volumes-cache, allow rwx pool=vms, allow rwx pool=vms-cache, allow rwx pool=images, allow rwx pool=images-cache"

然后更新 mon 的caps，添加 allow command "osd blacklist" 权限, 注意osd 的cap不变，使用上面获取的。

# docker exec ceph_mon ceph auth caps client.nova mon 'allow r, allow command "osd blacklist"' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=volumes-cache, allow rwx pool=vms, allow rwx pool=vms-cache, allow rwx pool=images, allow rwx pool=images-cache'

updated caps for client.nova

3、更新 client.cinder 的 caps。
首先获取目前的 cinder 的caps, 记住 caps osd 的值
# docker exec ceph_mon ceph auth get client.cinder

[client.cinder]
        key = AQC0zxdbAxupKRAAaMi5VM/pkJXU99IIIQjhCA==
        caps mon = "allow r"
        caps osd = "allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=volumes-cache, allow rwx pool=vms, allow rwx pool=vms-cache, allow rx pool=images, allow rx pool=images-cache"

然后更新 cinder的 mon 的caps，添加 allow command “osd blacklist” 权限, 注意osd 的cap不变，使用上面获取的cap。
# docker exec ceph_mon ceph auth caps client.cinder mon 'allow r, allow command "osd blacklist"' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=volumes-cache, allow rwx pool=vms, allow rwx pool=vms-cache, allow rx pool=images, allow rx pool=images-cache'

updated caps for client.cinder

4、重启所有 ceph_mon, ceph_osd, ceph_mgr

5、配置完成取消 osd 的 unout 标记
# docker exec ceph_mon ceph osd unset noout

noout is set

6、如要重新部署或者更新组件需参照上面步骤，将“osd blacklist”权限移除。
```
## 三、neutron ovs 导致CPU占用过高的问题
###### 查看进程,openvswith agent CPU利用率,一直100%
###### bug:https://bugs.launchpad.net/neutron/+bug/1750777
###### commit：
* "Do not start conntrack worker thread from __init__"
	* https://opendev.org/openstack/neutron/commit/4c8b97eca32c9c2beadf95fef14ed5b7d8981c5a
* "Remove race and simplify conntrack state management"
	* https://opendev.org/openstack/neutron/commit/cca870abbc4f8a784c9cbed817ff307ab3094daa

## 四、loadbalancer状态始终处于pending_update
```
修改数据库
update lbaas_loadbalancers set provisioning_status='ACTIVE' where id='c3b48167-9924-493e-a6c5-c4df5a8fd643';
```

## 五、虚拟机在线迁移失败
###### error : "Live Migration failure: 'ascii' codec can't encode characters in position 251-252“
###### bug：https://bugs.launchpad.net/nova/+bug/1768807
###### commit:Handle unicode characters in migration params
###### 原因如下：

```
                if six.PY2:
-                    params = {key: str(value) if isinstance(value, unicode)
-                                              else value
+                    params = {key: encodeutils.to_utf8(value)
+                              if isinstance(value, six.text_type) else value
                              for key, value in params.items()}

                self._domain.migrateToURI3(
```

## 六、ceph间隙失败导致"ImageNotFound"
###### error:'ceph job fails intermittently with "ImageNotFound: [errno 2] error opening image volume-ac0586ce-26c6-4e04-a4d6-78a941a93ad7 at snapshot None"'
###### bug: https://bugs.launchpad.net/cinder/+bug/1765845
###### commit:Handle ImageNotFound exception in _get_usage_info correctly
* https://opendev.org/openstack/cinder/commit/3cc8cbdee7c98e1fcf41bb27d81e34fef3a2d032

## 七、ceilometer内存监控不准确
###### commit：inspector: memory: use usable of memoryStats if available
* https://github.com/openstack/ceilometer/commit/2dee485da7a6f2cdf96525fabc18a8c27c8be570



