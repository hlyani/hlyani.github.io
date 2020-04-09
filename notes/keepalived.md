# keepalived 相关

### 1、安装

```
yum install -y keepalived
```

### 2、主节点配置

```
vim /etc/keepalived/keepalived.conf
```

```
! Configuration File for keepalived

global_defs {
   router_id node1
}

vrrp_instance VI_1 {
    state MASTER
    interface ens192
    virtual_router_id 127
    priority 250
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 35f18af7190d51c9f7f78f37300a0cbd
    }
    unicast_src_ip 10.11.1.11
    unicast_peer {
    10.11.1.12
    10.11.1.13
    }
    virtual_ipaddress {
        10.11.1.10/22 dev ens192
    }
}
```

### 3、从节点配置

```
vim /etc/keepalived/keepalived.conf
```

```
! Configuration File for keepalived

global_defs {
   router_id node2
}

vrrp_instance VI_1 {
    state SLAVE
    interface ens192
    virtual_router_id 127
    nopreempt
    priority 240
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 35f18af7190d51c9f7f78f37300a0cbd
    }
    unicast_src_ip 10.11.1.12
    unicast_peer {
    10.11.1.11
    10.11.1.13
    }
    virtual_ipaddress {
        10.11.1.10/22 dev ens192
    }
}
```

### 4、重启服务

```
systemctl start keepalived
systemctl enable keepalived
```

