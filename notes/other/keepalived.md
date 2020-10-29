# keepalived 相关

# 一、lvs+keepalived

```
yum install -y ipvsadm keepalived
```

```
! Configuration File for keepalived
global_defs {
   router_id lvs-keepalived01   #router_id 机器标识，通常为hostname，但不一定非得是hostname。故障发生时，邮件通知会用到。
}
vrrp_instance VI_1 {            #vrrp实例定义部分
    state MASTER                #设置lvs的状态，MASTER和BACKUP两种，必须大写 
    interface ens192            #设置对外服务的接口
    virtual_router_id 100       #设置虚拟路由标示，这个标示是一个数字，同一个vrrp实例使用唯一标示 
    priority 100                #定义优先级，数字越大优先级越高，在一个vrrp——instance下，master的优先级必须大于backup
    advert_int 1                #设定master与backup负载均衡器之间同步检查的时间间隔，单位是秒
    authentication {            #设置验证类型和密码
        auth_type PASS          #主要有PASS和AH两种
        auth_pass 1111          #验证密码，同一个vrrp_instance下MASTER和BACKUP密码必须相同
    }
    virtual_ipaddress {         #设置虚拟ip地址，可以设置多个，每行一个
        10.11.1.10
    }
}
virtual_server 10.11.1.10 32600 {    #设置虚拟服务器，需要指定虚拟ip和服务端口
    delay_loop 6                     #健康检查时间间隔
    lb_algo wrr                      #负载均衡调度算法
    lb_kind DR                       #负载均衡转发规则
    #persistence_timeout 50          #设置会话保持时间，对动态网页非常有用
    protocol TCP                     #指定转发协议类型，有TCP和UDP两种
    real_server 10.11.1.11 32600 {   #配置服务器节点1，需要指定real server的真实IP地址和端口
    weight 10                        #设置权重，数字越大权重越高
    TCP_CHECK {                      #realserver的状态监测设置部分单位秒
       connect_timeout 10            #连接超时为10秒
       retry 3                       #重连次数
       delay_before_retry 3          #重试间隔
       connect_port 32600            #连接端口为32600，要和上面的保持一致
       }
    }
    real_server 10.11.1.12 32600 {   #配置服务器节点1，需要指定real server的真实IP地址和端口
    weight 10                        #设置权重，数字越大权重越高
    TCP_CHECK {                      #realserver的状态监测设置部分单位秒
       connect_timeout 10            #连接超时为10秒
       retry 3                       #重连次数
       delay_before_retry 3          #重试间隔
       connect_port 32600            #连接端口为32600，要和上面的保持一致
       }
    }
    real_server 10.11.1.13 32600 {   #配置服务器节点1，需要指定real server的真实IP地址和端口
    weight 10                        #设置权重，数字越大权重越高
    TCP_CHECK {                      #realserver的状态监测设置部分单位秒
       connect_timeout 10            #连接超时为10秒
       retry 3                       #重连次数
       delay_before_retry 3          #重试间隔
       connect_port 32600            #连接端口为32600，要和上面的保持一致
       }
    }
}
```

```
! Configuration File for keepalived
global_defs {
   router_id lvs-keepalived02   #router_id 机器标识，通常为hostname，但不一定非得是hostname。故障发生时，邮件通知会用到。
}
vrrp_instance VI_1 {            #vrrp实例定义部分
    state BACKUP                #设置lvs的状态，MASTER和BACKUP两种，必须大写 
    interface ens192            #设置对外服务的接口
    virtual_router_id 100       #设置虚拟路由标示，这个标示是一个数字，同一个vrrp实例使用唯一标示 
    priority 90                 #定义优先级，数字越大优先级越高，在一个vrrp——instance下，master的优先级必须大于backup
    advert_int 1                #设定master与backup负载均衡器之间同步检查的时间间隔，单位是秒
    authentication {            #设置验证类型和密码
        auth_type PASS          #主要有PASS和AH两种
        auth_pass 1111          #验证密码，同一个vrrp_instance下MASTER和BACKUP密码必须相同
    }
    virtual_ipaddress {         #设置虚拟ip地址，可以设置多个，每行一个
        10.11.1.10
    }
}
virtual_server 10.11.1.10 32600 {    #设置虚拟服务器，需要指定虚拟ip和服务端口
    delay_loop 6                     #健康检查时间间隔
    lb_algo wrr                      #负载均衡调度算法
    lb_kind DR                       #负载均衡转发规则
    #persistence_timeout 50          #设置会话保持时间，对动态网页非常有用
    protocol TCP                     #指定转发协议类型，有TCP和UDP两种
    real_server 10.11.1.11 32600 {   #配置服务器节点1，需要指定real server的真实IP地址和端口
    weight 10                        #设置权重，数字越大权重越高
    TCP_CHECK {                      #realserver的状态监测设置部分单位秒
       connect_timeout 10            #连接超时为10秒
       retry 3                       #重连次数
       delay_before_retry 3          #重试间隔
       connect_port 32600            #! Configuration File for keepalived
global_defs {
   router_id lvs-keepalived02   #router_id 机器标识，通常为hostname，但不一定非得是hostname。故障发生时，邮件通知会用到。
}
vrrp_instance VI_1 {            #vrrp实例定义部分
    state BACKUP                #设置lvs的状态，MASTER和BACKUP两种，必须大写 
    interface ens192            #设置对外服务的接口
    virtual_router_id 100       #设置虚拟路由标示，这个标示是一个数字，同一个vrrp实例使用唯一标示 
    priority 90                 #定义优先级，数字越大优先级越高，在一个vrrp——instance下，master的优先级必须大于backup
    advert_int 1                #设定master与backup负载均衡器之间同步检查的时间间隔，单位是秒
    authentication {            #设置验证类型和密码
        auth_type PASS          #主要有PASS和AH两种
        auth_pass 1111          #验证密码，同一个vrrp_instance下MASTER和BACKUP密码必须相同
    }
    virtual_ipaddress {         #设置虚拟ip地址，可以设置多个，每行一个
        10.11.1.10
    }
}
virtual_server 10.11.1.10 32600 {    #设置虚拟服务器，需要指定虚拟ip和服务端口
    delay_loop 6                     #健康检查时间间隔
    lb_algo wrr                      #负载均衡调度算法
    lb_kind DR                       #负载均衡转发规则
    #persistence_timeout 50          #设置会话保持时间，对动态网页非常有用
    protocol TCP                     #指定转发协议类型，有TCP和UDP两种
    real_server 10.11.1.11 32600 {   #配置服务器节点1，需要指定real server的真实IP地址和端口
    weight 10                        #设置权重，数字越大权重越高
    TCP_CHECK {                      #realserver的状态监测设置部分单位秒
       connect_timeout 10            #连接超时为10秒
       retry 3                       #重连次数
       delay_before_retry 3          #重试间隔
       connect_port 32600            #连接端口为32600，要和上面的保持一致
       }
    }
    real_server 10.11.1.12 32600 {   #配置服务器节点1，需要指定real server的真实IP地址和端口
    weight 10                        #设置权重，数字越大权重越高
    TCP_CHECK {                      #realserver的状态监测设置部分单位秒
       connect_timeout 10            #连接超时为10秒
       retry 3                       #重连次数
       delay_before_retry 3          #重试间隔
       connect_port 32600            #连接端口为32600，要和上面的保持一致
       }
    }
    real_server 10.11.1.13 32600 {   #配置服务器节点1，需要指定real server的真实IP地址和端口
    weight 10                        #设置权重，数字越大权重越高
    TCP_CHECK {                      #realserver的状态监测设置部分单位秒
       connect_timeout 10            #连接超时为10秒
       retry 3                       #重连次数
       delay_before_retry 3          #重试间隔
       connect_port 32600            #连接端口为32600，要和上面的保持一致
       }
    }
}，要和上面的保持一致
       }
    }
    real_server 10.11.1.12 32600 {   #配置服务器节点1，需要指定real server的真实IP地址和端口
    weight 10                        #设置权重，数字越大权重越高
    TCP_CHECK {                      #realserver的状态监测设置部分单位秒
       connect_timeout 10            #连接超时为10秒
       retry 3                       #重连次数
       delay_before_retry 3          #重试间隔
       connect_port 32600            #连接端口为32600，要和上面的保持一致
       }
    }
    real_server 10.11.1.13 32600 {   #配置服务器节点1，需要指定real server的真实IP地址和端口
    weight 10                        #设置权重，数字越大权重越高
    TCP_CHECK {                      #realserver的状态监测设置部分单位秒
       connect_timeout 10            #连接超时为10秒
       retry 3                       #重连次数
       delay_before_retry 3          #重试间隔
       connect_port 32600            #连接端口为32600，要和上面的保持一致
       }
    }
}
```



# 二、keepalived

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

