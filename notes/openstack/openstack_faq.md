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

## 八、镜像验证问题

###### 问题描述：通过volume创建镜像，镜像的元数据中会多一个signature_verified=False, 并且使用该镜像创建虚拟机会失败。

[Support Image Signature Verification](https://specs.openstack.org/openstack/cinder-specs/specs/rocky/support-image-signature-verification.html)

[cinder.conf](https://docs.openstack.org/cinder/train/configuration/block-storage/samples/cinder.conf.html)

###### 解决方式，1、删除镜像元数据中的signature_verified。2、修改cinder的配置文件 verify_glance_signatures 默认是enabled。

```
verify_glance_signatures=disabled
```

## 九、cinder 配置

> cinder.conf

```
[DEFAULT]
enable_force_upload = true
verify_glance_signatures = disabled
```

```
enable_force_upload = true
表示卷上传为镜像时，允许在 in-use 状态上传。

verify_glance_signatures = disabled
表示在创建虚拟机时选择创建新卷的方式，不检查镜像是否签名正确。 如果不增加该配置，从卷创建的镜像无法直接启动虚拟机，并且后端报错为无法创建硬盘（尝试多次均失败，无关键问题信息）。
```

## 十、nova 配置

> nova.conf

```
[DEFAULT]
resize_confirm_window = 1
allow_resize_to_same_host = true
```

```
resize_confirm_window = 1
表示1s后自动确认修改。
在 resize 虚拟机时，需要再次确认修改。增加该配置，无需手动确认。

allow_resize_to_same_host = true
允许 resize 虚拟机时，调度到同一台主机上。
```

```
resize_confirm_window = 0
自动确认时间
0是禁用
1代表1s后自动确认
```

## 十一、keystone 配置

```
policy.yaml

# 提权，让普通用户也可以查询角色信息。api 在登录时会向 openstack 查询权限信息，并返回给前端使用。
# 如果登录时不反回角色信息，前端无法渲染合适的页面给多种角色。
"identity:list_role_assignments": "(role:reader and system_scope:all) or (role:reader and domain_id:%(target.domain_id)s) or (role:__member__) or (role:member)"
```

```
用户、角色、项目，这三者共同协作实现了权限功能。
一个用户在不同的项目下可以具有不同的角色，不同的角色对应着不同的行为。
认证方式：

project_id + 无作用域 token。出现此类认证方式是为了减少反复生成token，一份token可以无缝在多个项目下使用。
有作用域的 token。 不建议采用此类方式
用户名密码。不建议采用此类方式


角色介绍
openstack 内置了多个可用角色，目前常用角色为 admin、member。
admin 角色： 在该项目下，用户具有超级管理员权限，可以管理所有项目的资源（不包含特殊资源，比如 swift、keypair）。
member 角色： 在该项目下，用户具有项目管理员权限，可以管理该项目下的所有资源（不包含特殊资源，比如 keypair）。
当前项目需求，增加一个新的角色，该角色权限低于 member。本意为只允许使用项目基本资源，而不能管理项目基本资源。
junior_member 角色： 在该项目下，用户仅能使用基本资源，无法管理基本资源。
角色是实际权限行为由 policy 中的规则控制。每个组件对应一个 policy，分别控制组件自身的权限。
创建一个新的角色，通常意味着需要改写 policy 文件来生成对应的实际控制权限。

policy 文件编写注意事项
新创建的角色实际权限等同于member。
通过网络组件进行测试，内置角色之间似乎存在继承的关系，因此改变一个内置角色的权限，可能会影响其它的内置角色。（未深究）
在使用 kolla-ansible 进行部署时，注意不同版本支持的编写方式不同。当前 T 版本的 kolla-ansible 不支持 neutron policy.yaml，不支持 nova 的 policy 配置。

junior_member 实际权限参考
其它权限参考member。
额外权限限制：
网络
禁止创建网络
禁止修改网络
禁止删除网络
禁止创建子网
禁止修改子网
禁止删除子网
路由
禁止创建路由
禁止修改路由
禁止删除路由
禁止更新外网
禁止添加，删除子网
浮动ip
禁止创建浮动ip
禁止删除浮动ip
```

