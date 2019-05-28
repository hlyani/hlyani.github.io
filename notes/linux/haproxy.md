# haproxy 相关

##### 1、检查配置文件语法
```
haproxy -c -f /etc/haproxy/haproxy.cfg
```

##### 2、以daemon模式启动，以systemd管理的daemon模式启动
```
haproxy -D -f /etc/haproxy/haproxy.cfg [-p /var/run/haproxy.pid]
haproxy -Ds -f /etc/haproxy/haproxy.cfg [-p /var/run/haproxy.pid]
```

##### 3、启动调试功能，将显示所有连接和处理信息在屏幕
```
haproxy -d -f /etc/haproxy/haproxy.cfg
```

##### 4、restart，需要使用st选项指定pid列表

```
haproxy -f /etc/haproxy.cfg [-p /var/run/haproxy.pid] -st `cat /var/run/haproxy.pid
```

##### 5、graceful restart，即reload。需要使用sf选项指定pid列表
```
haproxy -f /etc/haproxy.cfg [-p /var/run/haproxy.pid] -sf `cat /var/run/haproxy.pid
```

##### 6、显示haproxy编译和启动信息
```
haproxy -vv
```

##### 7、配置实例

```
yum install haproxy -y

# 然后启动Haproxy

haproxy -f /usr/local/haproxy/etc/haproxy.cfg

# 停止Haproxy

killall haproxy

vim /etc/haproxy/haproxy.cfg

global
  chroot /var/lib/haproxy
  user haproxy
  group haproxy
  daemon
  log 127.0.0.1 local0 info
  maxconn 4000
  stats socket /var/lib/haproxy/haproxy.sock

defaults
  log global
  mode http
  option redispatch
  option httplog
  #option forwardfor
  retries 3
  timeout http-request 10s
  timeout queue 1m
  timeout connect 10s
  timeout client 1m
  timeout server 1m
  timeout check 10s

frontend stratos
  bind *:1234
  mode tcp
  option tcplog
  default_backend stratos_backend

frontend openstack
  bind *:80
  default_backend openstack_backend

backend stratos_backend
  mode tcp
  option tcplog
  stick-table type ip size 200k expire 30m
  stick on src
  option ssl-hello-chk
  server stratos 192.168.21.31:1234

backend openstack_backend
  http-request del-header X-Forwarded-Proto if { ssl_fc }
  http-request set-header X-Forwarded-Proto https if { ssl_fc }
  server kolla 192.168.21.100:80 check inter 2000 rise 2 fall 5
```