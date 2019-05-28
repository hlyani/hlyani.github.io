# OVS 相关

## 一、OVS调试

##### 1、ovs-vsctl show

```
96a55a7e-f49c-4dbe-b359-bafdff2ccad7
    Manager "ptcp:6640:92.0.0.12"
    Bridge br-tun
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        Port br-tun
            Interface br-tun
                type: internal
        Port "vxlan-5c00000b"
            Interface "vxlan-5c00000b"
                type: vxlan
                options: {df_default="true", in_key=flow, local_ip="92.0.0.12", out_key=flow, remote_ip="92.0.0.11"}
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
    Bridge br-int
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        Port "qvo4fab3e51-fc"
            tag: 3
            Interface "qvo4fab3e51-fc"
        Port int-br-ex
            Interface int-br-ex
                type: patch
                options: {peer=phy-br-ex}
        Port patch-tun
            Interface patch-tun
                type: patch
                options: {peer=patch-int}
        Port br-int
            Interface br-int
                type: internal
    Bridge br-ex
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        Port phy-br-ex
            Interface phy-br-ex
                type: patch
                options: {peer=int-br-ex}
        Port "ens4"
            Interface "ens4"
        Port br-ex
            Interface br-ex
                type: internal
```

##### 2、网桥查询

```
ovs-vsctl list-br

br-ex
br-int
br-tun
```

##### 3、端口查询

```
ovs-vsctl list-ports br-tun

patch-int
vxlan-5c00000b
```

##### 4、接口查询

```
ovs-vsctl list-ifaces br-tun

patch-int
vxlan-5c00000b
```

##### 5、端口、接口归属查询

```
ovs-vsctl port-to-br vxlan-5c00000b

br-tun

ovs-vsctl iface-to-br vxlan-5c00000b

br-tun
ovs-ofctl
```

##### 6、查询网桥流表

###### 样例
```
# ovs-ofctl dump-flows br-tun

NXST_FLOW reply (xid=0x4):

# 从port1进来的包转到表1处理

  cookie=0x0, duration=10970.064s, table=0, n_packets=189, n_bytes=16232, idle_age=16, priority=1,in_port=1 actions=resubmit(,1)

# 从port2进来的包转到表2处理

  cookie=0x0, duration=10906.954s, table=0, n_packets=29, n_bytes=5736, idle_age=16, priority=1,in_port=2 actions=resubmit(,2)

# 不匹配上面两条则drop

  cookie=0x0, duration=10969.922s, table=0, n_packets=3, n_bytes=230, idle_age=10962, priority=0 actions=drop

# 表1，单播包转到表20处理

  cookie=0x0, duration=10969.777s, table=1, n_packets=26, n_bytes=5266, idle_age=16, priority=0,dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20)

# 多播包转到表21处理

  cookie=0x0, duration=10969.631s, table=1, n_packets=163, n_bytes=10966, idle_age=21, priority=0,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,21)

# 表2，port2进来的包在这里处理了.同样是转给表10处理

  cookie=0x0, duration=688.456s, table=2, n_packets=29, n_bytes=5736, idle_age=16, priority=1,tun_id=0x1 actions=mod_vlan_vid:1,resubmit(,10)

# 表10，进行规则学习,具体就不解释了。学习到的规则后续会给表20来使用

  cookie=0x0, duration=10969.2s, table=10, n_packets=29, n_bytes=5736, idle_age=16, priority=1 actions=learn(table=20,hard_timeout=300,priority=1,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:1

# 表20, 根据目的mac设置tun_id,通过指定的port发出去

  cookie=0x0, duration=682.603s, table=20, n_packets=26, n_bytes=5266, hard_timeout=300, idle_age=16, hard_age=16, priority=1,vlan_tci=0x0001/0x0fff,dl_dst=fa:16:3e:32:0d:db actions=load:0->NXM_OF_VLAN_TCI[],load:0x1->NXM_NX_TUN_ID[],output:2

# 无规则的交给表21处理

  cookie=0x0, duration=10969.057s, table=20, n_packets=0, n_bytes=0, idle_age=10969, priority=0 actions=resubmit(,21)

# 表21，根据vlan找到对应的出去的口

  cookie=0x0, duration=688.6s, table=21, n_packets=161, n_bytes=10818, idle_age=21, priority=1,dl_vlan=1 actions=strip_vlan,set_tunnel:0x1,output:2

# drop

  cookie=0x0, duration=10968.912s, table=21, n_packets=2, n_bytes=148, idle_age=689, priority=0 actions=drop
```

##### 7、查询网桥信息

```
# ovs-ofctl show br-tun

OFPT_FEATURES_REPLY (xid=0x2): dpid:000096d30367a84a
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 1(patch-int): addr:4e:cb:5f:17:d4:d6
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 6(vxlan-5c00000b): addr:ca:48:f4:a1:7e:cb
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 LOCAL(br-tun): addr:96:d3:03:67:a8:4a
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
ovs-dpctl
Datapath统计信息查询：hit表示datapath命中数，missed未命中，lost表示没有传递到用户空间就丢弃了
```

```
# ovs-dpctl show

system@ovs-system:
    lookups: hit:99183 missed:37588 lost:1
    flows: 2
    masks: hit:231338 total:4 hit/pkt:1.69
    port 0: ovs-system (internal)
    port 1: br-ex (internal)
    port 2: ens4
    port 3: br-int (internal)
    port 4: br-tun (internal)
    port 5: qvo4fab3e51-fc
    port 6: vxlan_sys_4789 (vxlan)
查询端口详细统计信息

# ovs-dpctl show -s
system@ovs-system:
    lookups: hit:99202 missed:37594 lost:1
    flows: 5
    masks: hit:231423 total:4 hit/pkt:1.69
    port 0: ovs-system (internal)
        RX packets:0 errors:0 dropped:0 overruns:0 frame:0
        TX packets:0 errors:0 dropped:0 aborted:0 carrier:0
        collisions:0
        RX bytes:0  TX bytes:0
    port 1: br-ex (internal)
        RX packets:0 errors:0 dropped:0 overruns:0 frame:0
        TX packets:0 errors:0 dropped:136729 aborted:0 carrier:0
        collisions:0
        RX bytes:0  TX bytes:0
    port 2: ens4
        RX packets:138249 errors:0 dropped:0 overruns:0 frame:0
        TX packets:24986 errors:0 dropped:0 aborted:0 carrier:0
        collisions:0
        RX bytes:8046532 (7.7 MiB)  TX bytes:1052004 (1.0 MiB)
    port 3: br-int (internal)
        RX packets:0 errors:0 dropped:0 overruns:0 frame:0
        TX packets:0 errors:0 dropped:57 aborted:0 carrier:0
        collisions:0
        RX bytes:0  TX bytes:0
    port 4: br-tun (internal)
        RX packets:0 errors:0 dropped:0 overruns:0 frame:0
        TX packets:0 errors:0 dropped:0 aborted:0 carrier:0
        collisions:0
        RX bytes:0  TX bytes:0
    port 5: qvo4fab3e51-fc
        RX packets:23 errors:0 dropped:0 overruns:0 frame:0
        TX packets:8 errors:0 dropped:0 aborted:0 carrier:0
        collisions:0
        RX bytes:2364 (2.3 KiB)  TX bytes:648
    port 6: vxlan_sys_4789 (vxlan)
        RX packets:0 errors:? dropped:? overruns:? frame:?
        TX packets:0 errors:? dropped:? aborted:? carrier:?
        collisions:?
        RX bytes:0  TX bytes:0
查询指定端口统计信息

# ovs-ofctl dump-ports br-tun 6
OFPST_PORT reply (xid=0x2): 1 ports
  port  6: rx pkts=0, bytes=0, drop=?, errs=?, frame=?, over=?, crc=?
           tx pkts=0, bytes=0, drop=?, errs=?, coll=?
ovs-appctl
查询网桥转发规则

# ovs-appctl fdb/show br-tun
 port  VLAN  MAC                Age
调试
日志查询

### 可使用 ps -ef|grep ovsdb-server 查询conf.db的具体路径

# ovsdb-tool show-log -m /var/lib/openvswitch/conf.db

端口抓包（方式一）

### 通过进入设备OVS端口所在的网络空间进行监听，例如监听br-tun的patch-int端口

# ip netns list

# ip netns exec [NAME] bash

# ip addr show

# tcpdump -i [DEV]

端口抓包(方式二)

### 通过设置端口镜像来抓取没有具体设备的OVS端口，例如监听br-tun的patch-int端口

# ip link add name snooper0 type dummy

# ip link set dev snooper0 up

# ovs-vsctl add-port br-tun snooper0

# ovs-vsctl -- set Bridge br-tun mirrors=@m  -- --id=@snooper0   get Port snooper0  -- --id=@patch-int get Port patch-int   -- --id=@m create Mirror name=mymirror select-dst-port=@patch-int   select-src-port=@patch-int output-port=@snooper0 select_all=1

# ovs-vsctl clear Bridge br-tun mirrors

# ovs-vsctl del-port br-tun snooper0

# ip link delete dev snooper0

流表匹配

# ovs-appctl ofproto/trace br-tun dl_vlan=1
```

