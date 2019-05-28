# 迁移虚拟机到其他平台

##### 1、查看虚拟机基本信息

```
openstack server list --all-project|grep hl_cirros

bf8f03d1-5e96-4f63-8348-5e703d2c8c01 | hl_cirros | ACTIVE | hl=192.168.10.7|| 1G/1核/1GB |
```

##### 2、查看虚拟机详情

```
openstack server  show bf8f03d1-5e96-4f63-8348-5e703d2c8c01

+-------------------------------------+----------------------------------------------------------+
| Field                               | Value                                                    |
+-------------------------------------+----------------------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                                   |
| OS-EXT-AZ:availability_zone         | nova                                                     |
| OS-EXT-SRV-ATTR:host                | tmp1                                                 |
| OS-EXT-SRV-ATTR:hypervisor_hostname | tmp1                                                 |
| OS-EXT-SRV-ATTR:instance_name       | instance-000028ae                                        |
| OS-EXT-STS:power_state              | Running                                                  |
| OS-EXT-STS:task_state               | None                                                     |
| OS-EXT-STS:vm_state                 | active                                                   |
| OS-SRV-USG:launched_at              | 2019-01-10T13:40:35.000000                               |
| OS-SRV-USG:terminated_at            | None                                                     |
| accessIPv4                          |                                                          |
| accessIPv6                          |                                                          |
| addresses                           | hl=192.168.10.7                                          |
| config_drive                        |                                                          |
| created                             | 2019-01-10T13:40:20Z                                     |
| flavor                              | 1G/1核/1GB (03ee3258-8e54-4b4f-ba58-86250b98bbb4)        |
| hostId                              | 1ca96ff2970f3f6ac5baac71da801bac8021b4a27ed4c5f7eb284d78 |
| id                                  | bf8f03d1-5e96-4f63-8348-5e703d2c8c01                     |
| image                               |                                                          |
| key_name                            | None                                                     |
| name                                | hl_cirros                                                |
| progress                            | 0                                                        |
| project_id                          | 14f974575ad04ce4ae60790c1470d707                         |
| properties                          | image='8b7e3b42-c5f3-4d4c-b083-a0aeaaf7df86'             |
| security_groups                     | name='default'                                           |
| status                              | ACTIVE                                                   |
| updated                             | 2019-01-10T13:40:35Z                                     |
| user_id                             | 8cf334fc34814d62acdaef2a7a862654                         |
| volumes_attached                    | id='b0db72fd-2440-4aba-bbf9-d768c966571a'                |
|                                     | id='7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b'                |
+-------------------------------------+----------------------------------------------------------+
```

##### 3、查看磁盘信息

```
openstack volume list --all-project|grep hl_cirros

|7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b|hl_cirros|in-use|1|Attached to bf8f03d1-5e96-4f63-8348-5e703d2c8c01 on /dev/vdb|
```

##### 4、查看磁盘详情

```
openstack volume show 7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b

+--------------------------------+------------------------------------------------------------------------------------------+
| attachments                    | [{u'server_id': u'bf8f03d1-5e96-4f63-8348-5e703d2c8c01',
 u'attachment_id': u'e91444a3-359e-4dc8-b53c-979af407e105',
 u'attached_at': u'2019-01-10T13:47:52.000000', u'host_name': u'tmp1', 
 u'volume_id': u'7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b', u'device': u'/dev/vdb', u'id': u'7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b'}] |

| availability_zone              | nova                                 
| bootable                       | false                                
| consistencygroup_id            | None                                 
| created_at                     | 2019-01-10T13:47:30.000000           
| description                    | None                                 
| encrypted                      | False                                
| id                             | 7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b 
| migration_status               | None                                 
| multiattach                    | False                                
| name                           | hl_cirros                            
| os-vol-host-attr:host          | tmp2@rbd-1#rbd-1                 
| os-vol-mig-status-attr:migstat | None                                 
| os-vol-mig-status-attr:name_id | None                                 
| os-vol-tenant-attr:tenant_id   | 14f974575ad04ce4ae60790c1470d707     
| properties                     | attached_mode='rw'                   
| replication_status             | None                                 
| size                           | 1                                    
| snapshot_id                    | None                                 
| source_volid                   | None                                 
| status                         | in-use                               
| type                           | None                                 
| updated_at                     | 2019-01-10T13:47:52.000000           
| user_id                        | 8cf334fc34814d62acdaef2a7a862654     
```

```
openstack volume show b0db72fd-2440-4aba-bbf9-d768c966571a

+--------------------------------+-----------------------------------------------------------------------------+
| attachments                    | [{u'server_id': u'bf8f03d1-5e96-4f63-8348-5e703d2c8c01',
 u'attachment_id': u'90c57b13-6e2a-407e-9e6c-8c9612a7cb6a', 
 u'attached_at': u'2019-01-10T13:40:28.000000', 
 u'host_name': None, u'volume_id': u'b0db72fd-2440-4aba-bbf9-d768c966571a',
  u'device': u'/dev/vda', u'id': u'b0db72fd-2440-4aba-bbf9-d768c966571a'}] |
| availability_zone              | nova                                    
| bootable                       | true                                    
| consistencygroup_id            | None                                    
| created_at                     | 2019-01-10T13:40:24.000000              
| description                    |                                       
| encrypted                      | False                                   
| id                             | b0db72fd-2440-4aba-bbf9-d768c966571a    
| migration_status               | None                                    
| multiattach                    | False                                   
| name                           |                                       
| os-vol-host-attr:host          | tmp1@rbd-1#rbd-1                    
| os-vol-mig-status-attr:migstat | None                                    
| os-vol-mig-status-attr:name_id | None                                    
| os-vol-tenant-attr:tenant_id   | 14f974575ad04ce4ae60790c1470d707        
| properties                     | attached_mode='rw'                      
| replication_status             | None                                    
| size                           | 1                                       
| snapshot_id                    | None                                    
| source_volid                   | None                                    
| status                         | in-use                                  
| type                           | None                                    
| updated_at                     | 2019-01-10T13:40:28.000000              
| user_id                        | 8cf334fc34814d62acdaef2a7a862654        
| volume_image_metadata          | {u'description': u'cirros', 
u'checksum': u'ba3cd24377dde5dfdd58728894004abb',
 u'min_ram': u'128', u'disk_format': u'raw', u'image_name': u'cirros', 
 u'image_id': u'8b7e3b42-c5f3-4d4c-b083-a0aeaaf7df86', u'container_format': u'bare',
  u'min_disk': u'1', u'os_type': u'windows', u'size': u'46137344'}               |
+--------------------------------+-------------------------------------------------+
```

##### 5、查看ceph存储池

```
(ceph-mon)[root@tmp1 opt]# rados lspools
.rgw.root
default.rgw.control
default.rgw.meta
default.rgw.log
images
volumes
backups
vms
gnocchi
default.rgw.buckets.index
default.rgw.buckets.data
default.rgw.buckets.non-ec
c_bak
```

##### 6、查看 vms pool

```
rbd -p vms ls
```

##### 7、查看 volumes pool

```
rbd -p volumes ls|grep b0db72fd-2440-4aba-bbf9-d768c966571a
volume-b0db72fd-2440-4aba-bbf9-d768c966571a

rbd -p volumes ls|grep 7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b
volume-7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b
```

##### 8、导出volumes rbd

```
rbd export  volumes/volume-b0db72fd-2440-4aba-bbf9-d768c966571a  /opt/volume-b0db72fd-2440-4aba-bbf9-d768c966571a

rbd export  volumes/volume-7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b  /opt/volume-7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b
```

##### 9、重新导入volumes rbd

```
rbd import /opt/volume-b0db72fd-2440-4aba-bbf9-d768c966571a volumes/volume-b0db72fd-2440-4aba-bbf9-d768c966571a --image-format 2

rbd import /opt/volume-7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b volumes/volume-7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b --image-format 2
```

##### 10、查看虚拟机xml配置文件

```
virsh # dumpxml instance-000028ae 
<domain type='kvm' id='53'>
  <name>instance-000028ae</name>
  <uuid>bf8f03d1-5e96-4f63-8348-5e703d2c8c01</uuid>
  <metadata>
    <nova:instance xmlns:nova="http://openstack.org/xmlns/libvirt/nova/1.0">
      <nova:package version="17.0.2"/>
      <nova:name>hl_cirros</nova:name>
      <nova:creationTime>2019-01-10 13:40:29</nova:creationTime>
      <nova:flavor name="1G/1&#x6838;/1GB">
        <nova:memory>1024</nova:memory>
        <nova:disk>1</nova:disk>
        <nova:swap>0</nova:swap>
        <nova:ephemeral>0</nova:ephemeral>
        <nova:vcpus>1</nova:vcpus>
      </nova:flavor>
      <nova:owner>
        <nova:user uuid="8cf334fc34814d62acdaef2a7a862654">admin</nova:user>
        <nova:project uuid="14f974575ad04ce4ae60790c1470d707">hl</nova:project>
      </nova:owner>
    </nova:instance>
  </metadata>
  <memory unit='KiB'>1048576</memory>
  <currentMemory unit='KiB'>1048576</currentMemory>
  <vcpu placement='static'>1</vcpu>
  <cputune>
    <shares>1024</shares>
  </cputune>
  <resource>
    <partition>/machine</partition>
  </resource>
  <sysinfo type='smbios'>
    <system>
      <entry name='manufacturer'>OpenStack Foundation</entry>
      <entry name='product'>OpenStack Nova</entry>
      <entry name='version'>17.0.2</entry>
      <entry name='serial'>4c4c4544-004a-5410-8056-b9c04f4c5032</entry>
      <entry name='uuid'>bf8f03d1-5e96-4f63-8348-5e703d2c8c01</entry>
      <entry name='family'>Virtual Machine</entry>
    </system>
  </sysinfo>
  <os>
    <type arch='x86_64' machine='pc-i440fx-rhel7.5.0'>hvm</type>
    <boot dev='hd'/>
    <smbios mode='sysinfo'/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <hyperv>
      <relaxed state='on'/>
      <vapic state='on'/>
      <spinlocks state='on' retries='8191'/>
    </hyperv>
  </features>
  <cpu mode='custom' match='exact' check='full'>
    <model fallback='forbid'>Broadwell</model>
    <topology sockets='1' cores='1' threads='1'/>
    <feature policy='require' name='vme'/>
    <feature policy='require' name='f16c'/>
    <feature policy='require' name='rdrand'/>
    <feature policy='require' name='hypervisor'/>
    <feature policy='require' name='arat'/>
    <feature policy='require' name='xsaveopt'/>
    <feature policy='require' name='abm'/>
  </cpu>
  <clock offset='localtime'>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='hpet' present='no'/>
    <timer name='hypervclock' present='yes'/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <devices>
    <emulator>/usr/libexec/qemu-kvm</emulator>
    <disk type='network' device='disk'>
      <driver name='qemu' type='raw' cache='writeback' discard='unmap'/>
      <auth username='cinder'>
        <secret type='ceph' uuid='3f22f224-7591-4837-aa78-792a5e61eb4d'/>
      </auth>
      <source protocol='rbd' name='volumes/volume-b0db72fd-2440-4aba-bbf9-d768c966571a'>
        <host name='192.168.110.1' port='6789'/>
        <host name='192.168.110.2' port='6789'/>
        <host name='192.168.110.4' port='6789'/>
      </source>
      <target dev='vda' bus='virtio'/>
      <serial>b0db72fd-2440-4aba-bbf9-d768c966571a</serial>
      <alias name='virtio-disk0'/>

      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>

​    </disk>
​    <disk type='network' device='disk'>
​      <driver name='qemu' type='raw' cache='writeback' discard='unmap'/>
​      <auth username='cinder'>
​        <secret type='ceph' uuid='3f22f224-7591-4837-aa78-792a5e61eb4d'/>
​      </auth>
​      <source protocol='rbd' name='volumes/volume-7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b'>
​        <host name='192.168.110.1' port='6789'/>
​        <host name='192.168.110.2' port='6789'/>
​        <host name='192.168.110.4' port='6789'/>
​      </source>
​      <target dev='vdb' bus='virtio'/>
​      <serial>7b7b6f1b-56dc-4502-8e13-9bcfa3a7168b</serial>
​      <alias name='virtio-disk1'/>

      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>

​    </disk>
​    <controller type='usb' index='0' model='piix3-uhci'>
​      <alias name='usb'/>

      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x2'/>

​    </controller>
​    <controller type='pci' index='0' model='pci-root'>
​      <alias name='pci.0'/>
​    </controller>
​    <interface type='bridge'>
​      <mac address='fa:16:3e:3e:b0:96'/>
​      <source bridge='qbrb7ddd311-c0'/>
​      <target dev='tapb7ddd311-c0'/>
​      <model type='virtio'/>
​      <alias name='net0'/>

      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>

​    </interface>
​    <serial type='pty'>
​      <source path='/dev/pts/8'/>
​      <log file='/var/lib/nova/instances/bf8f03d1-5e96-4f63-8348-5e703d2c8c01/console.log' append='off'/>
​      <target type='isa-serial' port='0'>
​        <model name='isa-serial'/>
​      </target>
​      <alias name='serial0'/>
​    </serial>
​    <console type='pty' tty='/dev/pts/8'>
​      <source path='/dev/pts/8'/>
​      <log file='/var/lib/nova/instances/bf8f03d1-5e96-4f63-8348-5e703d2c8c01/console.log' append='off'/>
​      <target type='serial' port='0'/>
​      <alias name='serial0'/>
​    </console>
​    <input type='tablet' bus='usb'>
​      <alias name='input0'/>

      <address type='usb' bus='0' port='1'/>

​    </input>
​    <input type='mouse' bus='ps2'>
​      <alias name='input1'/>
​    </input>
​    <input type='keyboard' bus='ps2'>
​      <alias name='input2'/>
​    </input>
​    <graphics type='vnc' port='5906' autoport='yes' listen='192.168.110.1' keymap='en-us'>
​      <listen type='address' address='192.168.110.1'/>
​    </graphics>

    <video>
      <model type='cirrus' vram='16384' heads='1' primary='yes'/>
      <alias name='video0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </video>

​    <memballoon model='virtio'>
​      <stats period='10'/>
​      <alias name='balloon0'/>

      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>

​    </memballoon>
  </devices>
  <seclabel type='dynamic' model='dac' relabel='yes'>
​    <label>+42436:+42436</label>
​    <imagelabel>+42436:+42436</imagelabel>
  </seclabel>
</domain>
```

##### 11、查看虚拟机网络信息

```
openstack network list|grep hl
| 1221dbf4-fb34-4517-8f2e-738a14070271 | hl      | fd08b9a6-5429-4015-a8ad-02df4c4103b0 |
```

##### 12、查看网络子网信息

```
openstack subnet list |grep fd08b9a6-5429-4015-a8ad-02df4c4103b0
| fd08b9a6-5429-4015-a8ad-02df4c4103b0 | tmp-subnet-1221dbf4-fb34-4517-8f2e-738a14070271 | 1221dbf4-fb34-4517-8f2e-738a14070271 | 192.168.10.0/24  |
```

##### 13、查看子网详情

```
openstack subnet show fd08b9a6-5429-4015-a8ad-02df4c4103b0
+-------------------+-----------------------------------------------------+
| Field             | Value                                               |
+-------------------+-----------------------------------------------------+
| allocation_pools  | 192.168.10.2-192.168.10.254                         |
| cidr              | 192.168.10.0/24                                     |
| created_at        | 2019-01-04T03:24:05Z                                |
| description       |                                                     |
| dns_nameservers   | 114.114.114.114                                     |
| enable_dhcp       | True                                                |
| gateway_ip        | 192.168.10.1                                        |
| host_routes       |                                                     |
| id                | fd08b9a6-5429-4015-a8ad-02df4c4103b0                |
| ip_version        | 4                                                   |
| ipv6_address_mode | None                                                |
| ipv6_ra_mode      | None                                                |
| name              | tmp-subnet-1221dbf4-fb34-4517-8f2e-738a14070271 |
| network_id        | 1221dbf4-fb34-4517-8f2e-738a14070271                |
| project_id        | 14f974575ad04ce4ae60790c1470d707                    |
| revision_number   | 0                                                   |
| segment_id        | None                                                |
| service_types     |                                                     |
| subnetpool_id     | None                                                |
| tags              |                                                     |
| updated_at        | 2019-01-04T03:24:05Z                                |
+-------------------+-----------------------------------------------------+
```

##### 14、查看网络详情

```
openstack network show 1221dbf4-fb34-4517-8f2e-738a14070271

+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        | nova                                 |
| created_at                | 2019-01-04T03:24:04Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 1221dbf4-fb34-4517-8f2e-738a14070271 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | None                                 |
| is_vlan_transparent       | None                                 |
| mtu                       | 1450                                 |
| name                      | hl                                   |
| port_security_enabled     | True                                 |
| project_id                | 14f974575ad04ce4ae60790c1470d707     |
| provider:network_type     | vxlan                                |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 80                                   |
| qos_policy_id             | None                                 |
| revision_number           | 4                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   | fd08b9a6-5429-4015-a8ad-02df4c4103b0 |
| tags                      |                                      |
| updated_at                | 2019-01-04T03:24:05Z                 |
+---------------------------+--------------------------------------+
```

##### 15、创建flavor

```
openstack flavor create --id 0 --vcpus 1 --ram 1024 --disk 1 111
```

##### 16、创建安全组

```
openstack security group rule create --proto icmp default
openstack security group rule create --proto tcp --dst-port 22 default
```

##### 17、创建网络

```
openstack network create hl --provider-network-type vxlan
openstack subnet create hl-subnet --network hl --subnet-range 192.168.10.0/24
openstack network create --external --provider-physical-network physnet1 \
    --provider-network-type flat public1
openstack subnet create --no-dhcp \
    --allocation-pool start=192.168.21.120,end=192.168.21.130 --network public1 \
    --subnet-range 192.168.21.0/24 --gateway 192.168.21.1 public1-subnet
    
openstack network create --provider-network-type vxlan --share --enable demo-net
openstack subnet create --subnet-range 10.0.0.0/24 --network demo-net \
    --gateway 10.0.0.1 --dns-nameserver 114.114.114.114 demo-subnet
openstack router create demo-router
openstack router add subnet demo-router demo-subnet
openstack router set --external-gateway public1 demo-router
```

##### 18、创建虚拟机

```
nova boot --flavor 0 --image 796cabf0-6ac1-4520-ad5c-c726fa370d9d --nic auto hl
```

https://ask.openstack.org/en/question/27156/how-to-customize-libvirtxml-for-an-instance/

##### 19、查看数据库中虚拟机信息

```
MariaDB [nova]> select*from block_device_mapping where instance_uuid='a168e661-73d5-4c9b-ba00-7c16557f7fe8';
+---------------------+---------------------+------------+-------+-------------+-----------------------+-------------+-----------+-------------+-----------+-----------------+--------------------------------------+---------+-------------+------------------+--------------+-------------+----------+------------+--------------------------------------+------+---------------+--------------------------------------+
| created_at          | updated_at          | deleted_at | id    | device_name | delete_on_termination | snapshot_id | volume_id | volume_size | no_device | connection_info | instance_uuid                        | deleted | source_type | destination_type | guest_format | device_type | disk_bus | boot_index | image_id                             | tag  | attachment_id | uuid                                 |
+---------------------+---------------------+------------+-------+-------------+-----------------------+-------------+-----------+-------------+-----------+-----------------+--------------------------------------+---------+-------------+------------------+--------------+-------------+----------+------------+--------------------------------------+------+---------------+--------------------------------------+
| 2019-01-15 08:23:48 | 2019-01-15 08:23:48 | NULL       | 51874 | /dev/vda    |                     1 | NULL        | NULL      |        NULL |         0 | NULL            | a168e661-73d5-4c9b-ba00-7c16557f7fe8 |       0 | image       | local            | NULL         | disk        | NULL     |          0 | 8b7e3b42-c5f3-4d4c-b083-a0aeaaf7df86 | NULL | NULL          | 2febe1b0-9730-4c4a-9a73-34f9bccffb01 |
+---------------------+---------------------+------------+-------+-------------+-----------------------+-------------+-----------+-------------+-----------+-----------------+--------------------------------------+---------+-------------+------------------+--------------+-------------+----------+------------+--------------------------------------+------+---------------+--------------------------------------+
1 row in set (0.00 sec)
```

##### 20、创建虚拟机

```
nova boot --flavor 03ee3258-8e54-4b4f-ba58-86250b98bbb4 --image 8b7e3b42-c5f3-4d4c-b083-a0aeaaf7df86 --nic auto hl

nova boot --flavor 0 --image 796cabf0-6ac1-4520-ad5c-c726fa370d9d --nic auto hl
```


