# docker swarm 相关

## 一、docker swarm常用命令

>  docker swarm集群开放了三个端口：
>
> 2377端口， 用于集群管理通信
>
> 7946端口， 用于集群节点之间的通信
>
> 4789端口， 用于overlay网络流量

##### 1、初始化swarm manager并制定网卡地址

```
docker swarm init --advertise-addr 192.168.10.10

Swarm initialized: current node (anvkkvtgrxloqwleiyjjsw2gh) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-08ejd2bmn0yhns2cvgkpa0vnu095i4yc2t8jlbdqyvd3tekx6h-e5ore8jctvoycgw2vb1elne2z 192.168.110.82:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

##### 2、强制删除集群，如果是manager，需要添加 --force、

```
docker swarm leave --force
docker node rm node1 --force

# 在manager节点上执行
docker swarm update
```

#####3、查看swarm worker的连接令牌
```
docker swarm join-token worker
```

#####4、查看swarm manager的连接令牌
```
docker swarm join-token manager
```

#####5、使旧令牌无效并生成新令牌
```
docker swarm join-token --rotate

docker node inspect node1 --pretty
```

##### 6、将节点升级为manager

```
docker node promote node1
```

#####7、将节点降级为worker
```
docker node demote node1
```

##### 8、查看服务列表

```
docker service ls
```

##### 9、查看服务的具体信息

docker service ps nginx

##### 10、创建一个指定name、version、run cmd的服务

```
docker service create --name helloworld alping:3.6 ping docker.com
```

##### 11、创建一个指定name、port、replicas的服务

```
docker service create --name my_web --replicas 3 -p 80:80 nginx
```

##### 12、为指定的服务更新一个端口

```
docker service update --publish-add 80:80 my_web
```

##### 13、为指定的服务删除一个端口

```
docker service update --publish-rm 80:80 my_web
```

##### 14、将redis:3.0.6更新至redis:3.0.7

```
docker service update --image redis:3.0.7 redis
```

##### 15、配置运行环境，指定工作目录及环境变量

```
docker service create --name helloworld --env MYVAR=myvalue --workdir /tmp --user my_user alping ping docker.com
```

##### 16、更新helloworld服务的运行命令

```
docker service update --args "ping www.baidu.com" helloworld
```

##### 17、创建一个overlay网络

```
docker network create --driver overlay my_network
docker network create --driver overlay --subnet 10.10.10.0/24 --gateway 10.10.10.1 my-network

docker service create --name test --replicas 3 --network my-network redis

docker service update --network-rm my-network test

docker service update --network-add my_network test
```

##### 18、创建群组并配置cpu和内存

```
docker service create --name my_nginx --reserve-cpu 2 --reserve-memory 512m --replicas 3 nginx

docker service update --reserve-cpu 1 --reserve-memory 256m my_nginx
```

##### 19、更新配置

```
docker config ls
docker config inspect mysql
docker config rm mysql
docker service update --config-add mysql mysql
docker service update --config-rm mysql mysql

docker config create homepage index.html
docker service create --name nginx --publish 80:80 --replicas 3 --config src=homepage,target=/usr/share/nginx/html/index.html nginx 
```

##### 20、使用编排文件

```kafka.yaml 
version: '3.5'

networks:
  kafka_net_218aacad-5a9d-4f70-86a4-309031c76ea8:
    driver: overlay
    name: kafaka_net_218aacad-5a9d-4f70-86a4-309031c76ea8
services:
  my_kafka1:
    depends_on:
    - my_kafka_zk1
    - my_kafka_zk2
    - my_kafka_zk3
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 192.168.21.192
      KAFKA_ADVERTISED_PORT: 20229
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: my_kafka_zk1:2181,my_kafka_zk2:2181,my_kafka_zk3:2181,
    healthcheck:
      interval: 5s
      retries: 30
      test:
      - CMD
      - nc
      - -z
      - my_kafka_zk1
      - '2181'
      timeout: 10s
    image: 192.168.21.12:5000/tmp/kafka:2.12-2.1.0
    labels:
    - com.tmp.kafka.role=kafka
    networks:
      kafka_net_218aacad-5a9d-4f70-86a4-309031c76ea8:
        aliases:
        - my_kafka1
    ports:
    - mode: ingress
      protocol: tcp
      published: 20229
      target: 9092
  my_kafka2:
    depends_on:
    - my_kafka_zk1
    - my_kafka_zk2
    - my_kafka_zk3
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 192.168.21.192
      KAFKA_ADVERTISED_PORT: 20230
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: my_kafka_zk1:2181,my_kafka_zk2:2181,my_kafka_zk3:2181,
    healthcheck:
      interval: 5s
      retries: 30
      test:
      - CMD
      - nc
      - -z
      - my_kafka_zk1
      - '2181'
      timeout: 10s
    image: 192.168.21.12:5000/tmp/kafka:2.12-2.1.0
    labels:
    - com.tmp.kafka.role=kafka
    networks:
      kafka_net_218aacad-5a9d-4f70-86a4-309031c76ea8:
        aliases:
        - my_kafka2
    ports:
    - mode: ingress
      protocol: tcp
      published: 20230
      target: 9092
  my_kafka3:
    depends_on:
    - my_kafka_zk1
    - my_kafka_zk2
    - my_kafka_zk3
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 192.168.21.192
      KAFKA_ADVERTISED_PORT: 20231
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: my_kafka_zk1:2181,my_kafka_zk2:2181,my_kafka_zk3:2181,
    healthcheck:
      interval: 5s
      retries: 30
      test:
      - CMD
      - nc
      - -z
      - my_kafka_zk1
      - '2181'
      timeout: 10s
    image: 192.168.21.12:5000/tmp/kafka:2.12-2.1.0
    labels:
    - com.tmp.kafka.role=kafka
    networks:
      kafka_net_218aacad-5a9d-4f70-86a4-309031c76ea8:
        aliases:
        - my_kafka3
    ports:
    - mode: ingress
      protocol: tcp
      published: 20231
      target: 9092
  my_kafka_kafka-manager:
    depends_on:
    - my_kafka_zk1
    - my_kafka_zk2
    - my_kafka_zk3
    environment:
      ZK_HOSTS: my_kafka_zk1:2181,my_kafka_zk2:2181,my_kafka_zk3:2181,
    healthcheck:
      interval: 10s
      retries: 30
      test:
      - CMD-SHELL
      - curl 127.0.0.1:9000 | grep Active
      timeout: 15s
    image: 192.168.21.12:5000/tmp/kafka-manager:1.3.1.8
    labels:
    - com.tmp.kafka.role=kafka_manager
    networks:
      kafka_net_218aacad-5a9d-4f70-86a4-309031c76ea8:
        aliases:
        - my_kafka_kafka-manager
    ports:
    - mode: ingress
      protocol: tcp
      published: 20232
      target: 9000
  my_kafka_zk1:
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=my_kafka_zk1:2888:3888 server.2=my_kafka_zk2:2888:3888
        server.3=my_kafka_zk3:2888:3888
    hostname: my_kafka_zk1
    image: 192.168.21.12:5000/tmp/zookeeper:3.4.13
    labels:
    - com.tmp.kafka.role=zookeeper
    networks:
      kafka_net_218aacad-5a9d-4f70-86a4-309031c76ea8:
        aliases:
        - my_kafka_zk1
    ports:
    - mode: ingress
      protocol: tcp
      published: 20226
      target: 2181
  my_kafka_zk2:
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=my_kafka_zk1:2888:3888 server.2=my_kafka_zk2:2888:3888
        server.3=my_kafka_zk3:2888:3888
    hostname: my_kafka_zk2
    image: 192.168.21.12:5000/tmp/zookeeper:3.4.13
    labels:
    - com.tmp.kafka.role=zookeeper
    networks:
      kafka_net_218aacad-5a9d-4f70-86a4-309031c76ea8:
        aliases:
        - my_kafka_zk2
    ports:
    - mode: ingress
      protocol: tcp
      published: 20227
      target: 2181
  my_kafka_zk3:
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=my_kafka_zk1:2888:3888 server.2=my_kafka_zk2:2888:3888
        server.3=my_kafka_zk3:2888:3888
    hostname: my_kafka_zk3
    image: 192.168.21.12:5000/tmp/zookeeper:3.4.13
    labels:
    - com.tmp.kafka.role=zookeeper
    networks:
      kafka_net_218aacad-5a9d-4f70-86a4-309031c76ea8:
        aliases:
        - my_kafka_zk3
    ports:
    - mode: ingress
      protocol: tcp
      published: 20228
      target: 2181
```

```
docker stack deploy -c kafka.yaml my_kafka
```

## 二、docker swarm 网络相关

[swarm 网络](https://hlyani.github.io/notes/docker/docker_net.html)

