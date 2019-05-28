# MQTT 相关

## 一、MQTT 安装
##### 1、下载
```
wget https://emqx.io/static/brokers/emqttd-docker-v2.3.11.zip --no-check-certificate

unzip emqttd-docker-v2.3.11.zip
docker load emqttd-docker-v2.3.11

https://github.com/emqx/emqx-docker
docker pull devicexx/emqttd

http://emqtt.com/downloads
```
##### 2、启动mqtt
####### 1.直接容器启动
```
docker run -tid --name emqtt -p 1883:1883 -p 8083:8083 -p 8883:8883 -p 8084:8084 -p 18083:18083 emqttd-docker-v2.3.11

docker run --rm -ti --name emq -p 18083:18083 -p 1883:1883 -p 4369:4369 -p 6000-6100:6000-6100 \
    -e EMQ_NAME="emq1" \
    -e EMQ_HOST="172.17.0.3" \
    -e EMQ_LISTENER__TCP__EXTERNAL=1883 \
    -e EMQ_JOIN_CLUSTER="emq@172.17.0.2" \
    emq

```
####### 2.docker swarm 启动
``` docker-compose.yaml
version: '3.5'

configs:
  haproxy_config:
    file: /tmp/tmpxzqwwynp
networks:
  emq_net_a0087c90-78ac-420a-86cd-b5f1a1720625:
    driver: overlay
    name: emq_net_a0087c90-78ac-420a-86cd-b5f1a1720625
services:
  haproxy:
    configs:
    - source: haproxy_config
      target: /usr/local/etc/haproxy/haproxy.cfg
    depends_on:
    - my_mqtt1
    - my_mqtt2
    - my_mqtt3
    environment:
      MQTT_ADDRS: my_mqtt1.emq.tt my_mqtt2.emq.tt my_mqtt3.emq.tt
    hostname: haproxy
    image: 192.168.21.12:5000/tmp/haproxy:1.9
    labels:
    - com.tmp.mqtt.role=haproxy
    networks:
      emq_net_a0087c90-78ac-420a-86cd-b5f1a1720625:
        aliases:
        - haproxy
    ports:
    - mode: ingress
      protocol: tcp
      published: 20201
      target: 1883
    - mode: ingress
      protocol: tcp
      published: 20202
      target: 18083
  my_mqtt1:
    environment:
      EMQ_HOST: my_mqtt1.emq.tt
      EMQ_LISTENER__TCP__EXTERNAL: 1883
      EMQ_NAME: emq
    healthcheck:
      interval: 5s
      retries: 30
      test:
      - CMD
      - nc
      - -z
      - localhost
      - '1883'
      timeout: 10s
    image: 192.168.21.12:5000/tmp/emqttd:2.3.11
    networks:
      emq_net_a0087c90-78ac-420a-86cd-b5f1a1720625:
        aliases:
        - my_mqtt1.emq.tt
  my_mqtt2:
    depends_on:
    - my_mqtt1
    environment:
      EMQ_HOST: my_mqtt2.emq.tt
      EMQ_JOIN_CLUSTER: emq@my_mqtt1.emq.tt
      EMQ_LISTENER__TCP__EXTERNAL: 1883
      EMQ_NAME: emq
    healthcheck:
      interval: 5s
      retries: 30
      test:
      - CMD
      - nc
      - -z
      - localhost
      - '1883'
      timeout: 10s
    image: 192.168.21.12:5000/tmp/emqttd:2.3.11
    networks:
      emq_net_a0087c90-78ac-420a-86cd-b5f1a1720625:
        aliases:
        - my_mqtt2.emq.tt
  my_mqtt3:
    depends_on:
    - my_mqtt1
    environment:
      EMQ_HOST: my_mqtt3.emq.tt
      EMQ_JOIN_CLUSTER: emq@my_mqtt1.emq.tt
      EMQ_LISTENER__TCP__EXTERNAL: 1883
      EMQ_NAME: emq
    healthcheck:
      interval: 5s
      retries: 30
      test:
      - CMD
      - nc
      - -z
      - localhost
      - '1883'
      timeout: 10s
    image: 192.168.21.12:5000/tmp/emqttd:2.3.11
    networks:
      emq_net_a0087c90-78ac-420a-86cd-b5f1a1720625:
        aliases:
        - my_mqtt3.emq.tt
```
启动
```
docker stack deploy -c docker-compose.yml mqtt

docker config create site-v2.conf site.conf

docker service create --name emqtt -p 1883:1883 -p 8083:8083 -p 8883:8883 -p 8084:8084 -p 8080:8080 -p 18083:18083 --config src=emq.conf,target=/opt/emqttd/etc/emq.conf emq

docker service update --config-rm emq.conf --config-add source=emq1,target=/opt/emqttd/etc/emq.conf emqtt

docker service update \
  --config-rm 405a57ac-616f-448d-b103-7cfc3a8734e0_haproxy_config \
  --config-add source=haproxy,target=/usr/local/etc/haproxy/haproxy.cfg \
  7kwtrjr320s6

docker network create \
  --driver overlay \
  --ingress \
  --subnet=10.10.10.0/24 \
  --opt com.docker.network.driver.mtu=1500 \
  myingress
  
docker run -tid --name emqtt_hl -p 31883:1883 -p 38083:18083 --network u8idzog4wrw6 emqttd-docker-v2.3.11
```
##### 3、haproxy配置
```
global
  daemon
  log 127.0.0.1 local0 info
  stats socket /usr/local/etc/haproxy/haproxy.sock
  maxconn 2000000

defaults
  log global
  mode http
  option redispatch
  option httplog
  option forwardfor
  retries 10
  timeout http-request 10s
  timeout connect 10h
  timeout client 10h
  timeout server 10h
  maxconn 2000000

listen dashborad
  bind *:18083
  balance source
  http-request del-header X-Forwarded-Proto if { ssl_fc }
  server mqtt1.emq.tt mqtt1.emq.tt:18083 check inter 10000 fall 2 rise 5 weight 1
  server mqtt2.emq.tt mqtt2.emq.tt:18083 check inter 10000 fall 2 rise 5 weight 1
  server mqtt3.emq.tt mqtt3.emq.tt:18083 check inter 10000 fall 2 rise 5 weight 1

listen api
  bind *:1883
  mode tcp
  option tcplog
  balance source
  http-request del-header X-Forwarded-Proto if { ssl_fc }
  server mqtt1.emq.tt mqtt1.emq.tt:1883 check inter 10000 fall 2 rise 5 weight 1
  server mqtt2.emq.tt mqtt2.emq.tt:1883 check inter 10000 fall 2 rise 5 weight 1
  server mqtt3.emq.tt mqtt3.emq.tt:1883 check inter 10000 fall 2 rise 5 weight 1
```
##### 4、mqtt端口相关
```
18083 Dashboard 管理控制台端口
1883 MQTT 协议端口
8083 MQTT/WebSocket 端口

1883	MQTT Port
8883	MQTT/SSL Port
8083	MQTT/WebSocket Port
8084	MQTT/WebSocket/SSL Port
8080	HTTP Management API Port
```
##### 5、常用命令
```
#查看状态
docker exec -itu0 emqtt1 ./bin/emqttd_ctl status
#加入集群
docker run -itd --name emqtt2 --link emqtt1 devicexx/emqttd
#离开集群
docker exec -itu0 emqtt2  ./bin/emqttd_ctl cluster leave
#emqttd1 节点下删除 emqttd2:
docker exec -itu0 emqtt1 ./bin/emqttd_ctl cluster remove emqttd2@127.0.0.1
#修改密码
docker exec -itu0 emqtt1 ./bin/emqttd_ctl admins passwd admin 123456

使用文档
http://emqtt.com/docs/v2/getstarted.html
```
###### 6、编译mqtt镜像
```
git clone -b v2.3.11 https://github.com/emqtt/emq_docker.git
cd emq_docker
docker build -t emq:latest .
```
##### 7、服务器优化
```
cat << EOF >> /etc/sysctl.conf
fs.file-max=2097152 
fs.nr_open=2097152
net.core.somaxconn=32768
net.ipv4.tcp_max_syn_backlog=16384
net.core.netdev_max_backlog=16384
net.ipv4.ip_local_port_range=1000 65535
net.core.rmem_default=262144
net.core.wmem_default=262144
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.core.optmem_max=16777216
net.ipv4.tcp_rmem=1024 4096 16777216
net.ipv4.tcp_wmem=1024 4096 16777216
net.nf_conntrack_max=1000000
net.netfilter.nf_conntrack_max=1000000
net.netfilter.nf_conntrack_tcp_timeout_time_wait=30
net.ipv4.tcp_max_tw_buckets=1048576
net.ipv4.tcp_fin_timeout = 15
EOF

cat << EOF >>/etc/security/limits.conf
*      soft   nofile      1048576
*      hard   nofile      1048576
EOF
 
echo DefaultLimitNOFILE=1048576 >>/etc/systemd/system.conf
echo 5097152 > /proc/sys/fs/nr_open

sysctl -w kernel.msgmnb=65536
sysctl -w kernel.msgmax=65536
sysctl -w kernel.shmmax=68719476736
sysctl -w kernel.shmall=4294967296
sysctl -w net.ipv4.tcp_fin_timeout=30
sysctl -w fs.nr_open=5097152
sysctl -w fs.file-max=5097152
sysctl -w net.ipv4.tcp_tw_recycle=1
sysctl -w net.ipv4.tcp_tw_reuse=1
sysctl -w net.core.rmem_default=524288
sysctl -w net.core.wmem_default=524288
sysctl -w net.core.rmem_max=67108864
sysctl -w net.core.wmem_max=67108864
sysctl -w net.core.optmem_max=67108864
sysctl -w net.ipv4.tcp_rmem='4096 87380 16777216'
sysctl -w net.ipv4.tcp_wmem='4096 65536 16777216'
sysctl -w net.ipv4.ip_local_port_range='1024 65535'
sysctl -w net.core.somaxconn=32768
sysctl -w net.ipv4.tcp_max_syn_backlog=16384
sysctl -w net.core.netdev_max_backlog=16384
sysctl -w net.nf_conntrack_max=1000000
sysctl -w net.netfilter.nf_conntrack_max=1000000
sysctl -w net.netfilter.nf_conntrack_tcp_timeout_time_wait=30
sysctl -w net.ipv4.tcp_fin_timeout=15

ulimit -n 200000
```
## 二、MQTT使用与测试
##### 1、使用
```
1.安装
yum install -y mosquitto
systemctl start mosquitto

2.使用
# emqttd1节点上订阅x
mosquitto_sub -t x -q 1 -p 1883

# emqttd2节点上向x发布消息
mosquitto_pub -t x -q 1 -p 1883 -m hello
```
##### 2、测试
```
https://github.com/emqtt/emqtt_benchmark

netstat -nat|grep -i "1883"|wc -l

1.安装
yum -y install gcc gcc-c++ glibc-devel make ncurses-devel openssl-devel autoconf java-1.8.0-openjdk-devel git
git clone https://github.com/erlang/otp.git
cd otp
./otp_build autoconf
./configure
make
make install

git clone https://github.com/erlang/rebar3.git
cd rebar3
./bootstrap
cp rebar3 /usr/local/bin

git clone https://github.com/emqtt/emqtt_benchmark.git
make

2.使用
./emqtt_bench_pub -t bench/%i  -h 192.168.21.204 -p 20155 -c 2300
./emqtt_bench_sub -t bench/%i -h 192.168.21.204 -p 20155 -c 2000

3.参数解释
//服务器ip地址
-h, --host       mqtt server hostname or IP address [default: localhost]
//服务器端口号
-p, --port       mqtt server port number [default: 1883]
//最大连接客户端数量 默认200
-c, --count      max count of clients [default: 200]
//客户端连接间隔时间，默认10毫秒
-i, --interval   interval of connecting to the broker [default: 10]
//订阅的主题 %i=自增长序号
-t, --topic      topic subscribe, support %u, %c, %i variables
//消息服务qos等级，
//0=最多一次 服务器与 客户端 交互1次
//1=至少一次 服务器与 客户端 交互2次
//2=仅有一次 服务器与 客户端 交互4次
-q, --qos        subscribe qos [default: 0]
//客户端用户名
-u, --username   username for connecting to server
//用户端密码
-P, --password   password for connecting to server
//维持客户端活跃的时间 默认300秒
-k, --keepalive  keep alive in seconds [default: 300]
//客户端断开后是否清除session 默认true
-C, --clean      clean session [default: true]
//代理ip接口
--ifaddr         local ipaddress or interface address
```
