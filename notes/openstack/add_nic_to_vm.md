# 给虚拟机添加网卡

##### 1、查看虚拟机
```
openstack server list

+--------------------------------------+---------------------------+---------+--------------------------------+------------------------------------+-------------+
| ID                                   | Name                      | Status  | Networks                       | Image                              | Flavor      |
+--------------------------------------+---------------------------+---------+--------------------------------+------------------------------------+-------------+
| dc15eae5-3974-4e57-bdcd-6d6ebd7451d5 | qwehl                     | ACTIVE  | test=20.1.1.3, 20.1.1.19       |                                    | m1.medium   |
+--------------------------------------+---------------------------+---------+--------------------------------+------------------------------------+-------------+
```
##### 2、查看虚拟机详情（后面为虚拟机id）
```
openstack server show dc15eae5-3974-4e57-bdcd-6d6ebd7451d5

+-----------------------------+----------------------------------------------------------+
| Field                       | Value                                                    |
+-----------------------------+----------------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                                   |
| OS-EXT-AZ:availability_zone | nova                                                     |
| OS-EXT-STS:power_state      | Running                                                  |
| OS-EXT-STS:task_state       | None                                                     |
| OS-EXT-STS:vm_state         | active                                                   |
| OS-SRV-USG:launched_at      | 2019-03-01T04:29:23.000000                               |
| OS-SRV-USG:terminated_at    | None                                                     |
| accessIPv4                  |                                                          |
| accessIPv6                  |                                                          |
| addresses                   | test=20.1.1.3, 20.1.1.19                                 |
| config_drive                |                                                          |
| created                     | 2019-03-01T04:29:10Z                                     |
| flavor                      | m1.medium (3)                                            |
| hostId                      | f3eaa989744784defe74e480a76a8af1a6e0960a78f1e98ff8e6e463 |
| id                          | dc15eae5-3974-4e57-bdcd-6d6ebd7451d5                     |
| image                       |                                                          |
| key_name                    | None                                                     |
| name                        | qwehl                                                    |
| progress                    | 0                                                        |
| project_id                  | 97eaa64704bd4f549f3abf99a6decdc1                         |
| properties                  | image='443cceba-8ac0-4336-9818-bc524bb49c74'             |
| security_groups             | name='default'                                           |
|                             | name='default'                                           |
| status                      | ACTIVE                                                   |
| updated                     | 2019-03-01T04:29:23Z                                     |
| user_id                     | 8cf334fc34814d62acdaef2a7a862654                         |
| volumes_attached            | id='43a49283-f333-41fb-928f-4ce91762dc58'                |
+-----------------------------+----------------------------------------------------------+
```
##### 3、查看网络详情
```
openstack network show test

+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        | nova                                 |
| created_at                | 2019-02-18T01:59:37Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 2e6ff4d6-b5ee-4390-892c-568ab5d71107 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | None                                 |
| is_vlan_transparent       | None                                 |
| location                  | None                                 |
| mtu                       | 1450                                 |
| name                      | test                                 |
| port_security_enabled     | True                                 |
| project_id                | 97eaa64704bd4f549f3abf99a6decdc1     |
| provider:network_type     | None                                 |
| provider:physical_network | None                                 |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 4                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   | 70ee3596-2aa9-4367-aaa8-d95d183ad802 |
| tags                      |                                      |
| updated_at                | 2019-02-18T01:59:38Z                 |
+---------------------------+--------------------------------------+
```
##### 4、创建端口
```
openstack port create --network 2e6ff4d6-b5ee-4390-892c-568ab5d71107 port
```
##### 5、查看端口
```
openstack port list
```
##### 6、查看虚拟机端口
```
nova interface-list dc15eae5-3974-4e57-bdcd-6d6ebd7451d5

+------------+--------------------------------------+--------------------------------------+--------------+-------------------+
| Port State | Port ID                              | Net ID                               | IP addresses | MAC Addr          |
+------------+--------------------------------------+--------------------------------------+--------------+-------------------+
| ACTIVE     | 47197121-e07a-4b81-9dda-7eac2918ab93 | 2e6ff4d6-b5ee-4390-892c-568ab5d71107 | 20.1.1.3     | fa:16:3e:53:e7:cd |
+------------+--------------------------------------+--------------------------------------+--------------+-------------------+
```
##### 7、挂载端口
```
nova interface-attach --port-id 57258e65-9e27-42a8-8039-10bf7f6624ba dc15eae5-3974-4e57-bdcd-6d6ebd7451d5
```
##### 8、再次查看虚拟机端口
```
nova interface-list dc15eae5-3974-4e57-bdcd-6d6ebd7451d5

+------------+--------------------------------------+--------------------------------------+--------------+-------------------+
| Port State | Port ID                              | Net ID                               | IP addresses | MAC Addr          |
+------------+--------------------------------------+--------------------------------------+--------------+-------------------+
| ACTIVE     | 47197121-e07a-4b81-9dda-7eac2918ab93 | 2e6ff4d6-b5ee-4390-892c-568ab5d71107 | 20.1.1.3     | fa:16:3e:53:e7:cd |
| ACTIVE     | 57258e65-9e27-42a8-8039-10bf7f6624ba | 2e6ff4d6-b5ee-4390-892c-568ab5d71107 | 20.1.1.19    | fa:16:3e:9c:c5:42 |
+------------+--------------------------------------+--------------------------------------+--------------+-------------------+
```
##### 9、重启虚拟机
```
openstack server reboot dc15eae5-3974-4e57-bdcd-6d6ebd7451d5
```