# frp 相关

# 1、容器镜像

```
docker pull docker.io/snowdreamtech/frps:latest
```

# 2、服务端

```
cat > ./frps.toml << EOF
[common]
bind_port = 7000
bind_udp_port = 7001
token = xxx
dashboard_user = admin
dashboard_pwd =  xxx
dashboard_port = 7500
heartbeat_timeout = 90
max_pool_count = 50

EOF
```

```
docker run --restart=always --network host -d -v $PWD/frps.toml:/etc/frp/frps.toml --name frps snowdreamtech/frps
```

# 3、客户端

```
cat > /etc/frp/frpc.toml << EOF
[common]
server_addr=xxx.xxx.xxx.xxx
server_port=7000
server_udp_port=7001
token=xxx

[ssh]
type=tcp
local_ip=127.0.0.1
local_port=22
remote_port=22

[nas]
type=tcp
local_ip=127.0.0.1
local_port=5000
remote_port=5000

[smb]
type=tcp
local_ip=127.0.0.1
local_port=445
remote_port=4445

EOF
```

```
docker run --restart=always --network host -d -v $PWD/frpc.toml:/etc/frp/frpc.toml --name frpc snowdreamtech/frpc
```

