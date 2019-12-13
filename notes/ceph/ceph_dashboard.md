# Ceph 启用 Dashboard

##### 1、查看 ceph 状态

```
ceph -s
```

##### 2、启用 dashboard

```
ceph mgr module enable dashboard
```

##### 3、生成并安装一个自签名证书

```
ceph dashboard create-self-signed-cert
```

##### 4、配置服务地址、端口，默认的端口是8443

```
ceph config set mgr mgr/dashboard/server_addr 192.168.0.10
ceph config set mgr mgr/dashboard/server_port 8443
ceph mgr services
```

##### 5、创建一个用户、密码

```
ceph dashboard set-login-credentials admin admin
```

##### 6、重启 mgr

```
systemctl restart ceph-mgr@node1
```

##### 7、访问

```
https://192.168.0.10:8443
```

