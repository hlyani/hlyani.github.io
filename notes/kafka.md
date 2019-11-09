# Kafka 相关

## 一、kafka介绍
```
   一个topic是对一组消息的归纳。对每个topic，Kafka 对它的日志进行了分区。
```
```
Produce
   将消息发布到它指定的topic中，并负责决定发布到哪个分区。通常简单的由负责均衡机制随机选择分区，但也可以通过特定的分区函数选择分区。使用更多的是第二种。
```
```
consumer
   发布消息通常有两种模式：队列模式（queuing）和发布-订阅模式(publish-subscribe)。队列模式中，consumers可以同时从服务端读取消息，每个消息只被其中一个consumer读到；发布-订阅模式中消息被广播到所有的consumer中。Consumers可以加入一个consumer
组，共同竞争一个topic，topic中的消息将被分发到组中的一个成员中。同一组中的consumer可以在不同的程序中，也可以在不同的机器上。如果所有的consumer都在一个组中，这就成为了传统的队列模式，在各consumer中实现负载均衡。如果所有的consumer都不在不同的组中，这就成为了发布-订阅模式，所有的消息都被分发到所有的consumer中。更常见的是，每个topic都有若干数量的consumer组，每个组都是一个逻辑上的“订阅者”，为了容错和更好的稳定性，每个组由若干consumer组成。这其实就是一个发布-订阅模式，只不过订阅者是个组而不是单个consumer。
```
```
   相比传统的消息系统，Kafka可以很好的保证有序性。
传统的队列在服务器上保存有序的消息，如果多个consumers同时从这个服务器消费消息，服务器就会以消息存储的顺序向consumer分发消息。虽然服务器按顺序发布消息，但是消息是被异步的分发到各consumer上，所以当消息到达时可能已经失去了原来的顺序，这意味着并发消费将导致顺序错乱。为了避免故障，这样的消息系统通常使用“专用consumer”的概念，其实就是只允许一个消费者消费消息，当然这就意味着失去了并发性。

   在这方面Kafka做的更好，通过分区的概念，Kafka可以在多个consumer组并发的情况下提供较好的有序性和负载均衡。将每个分区分只分发给一个consumer组，这样一个分区就只被这个组的一个consumer消费，就可以顺序的消费这个分区的消息。因为有多个分区，依然可以在多个consumer组之间进行负载均衡。注意consumer组的数量不能多于分区的数量，也就是有多少分区就允许多少并发消费。

   Kafka只能保证一个分区之内消息的有序性，在不同的分区之间是不可以的，这已经可以满足大部分应用的需求。如果需要topic中所有消息的有序性，那就只能让这个topic只有一个分区，当然也就只有一个consumer组消费它。
```
## 二、安装
###### 1、拉取zookeeper、kafka、kafka-manager镜像
```
docker pull zookeeper
docker pull ches/kafka
docker pull sheepkiller/kafka-manager
```
##### 2、使用docker启动相关容器
```
docker network create kafka-net
docker run -d --name zookeeper --network kafka-net -p 2181:2181 zookeeper
docker run -d --name kafka --network kafka-net --env ZOOKEEPER_IP=zookeeper -p 7203:7203 -p 9092:9092 ches/kafka

#外网访问
docker run -d \
    --hostname localhost \
    --name kafka \
    --publish 9092:9092 --publish 7203:7203 \
    --env KAFKA_ADVERTISED_HOST_NAME=110.202.198.96 --env ZOOKEEPER_IP=110.202.198.96 --env KAFKA_ADVERTISED_PORT=50007 --env ZOOKEEPER_PORT=50008 \
    ches/kafka
```
##### 3、使用docker swarm启动相关容器
``` kafka.yaml
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
##### 4、使用

[下载kafka](https://kafka.apache.org/downloads)

```
yum -y install java-1.8.0-openjdk

curl -O http://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.3.0/kafka_2.12-2.3.0.tgz 
tar -zxvf kafka_2.12-2.3.0.tgz 
cd kafka_2.12-2.3.0/bin

./kafka-topics.sh --list --zookeeper 192.168.110.224:12181,192.168.110.224:22181,192.168.110.224:32181

./kafka-topics.sh --create --topic test --replication-factor 1 --partitions 1 --zookeeper 192.168.110.224:12181,192.168.110.224:22181,192.168.110.224:32181

./kafka-console-consumer.sh --topic test --from-beginning --bootstrap-server 192.168.110.224:19092,192.168.110.224:29092,192.168.110.224:39092

./kafka-console-producer.sh --topic test --broker-list 192.168.110.224:19092,192.168.110.224:29092,192.168.110.224:39092


#查看topic
docker run --rm --network kafka-net ches/kafka kafka-topics.sh --list --zookeeper zookeeper:2181

#查看topic详情
docker run --rm --network kafka-net ches/kafka kafka-topics.sh --describe --zookeeper zookeeper:2181 --topic test
```

##### 5、更新env允许外网能访问

[https://docs.docker.com/engine/reference/commandline/service_update/](https://docs.docker.com/engine/reference/commandline/service_update/)

```
docker service ls|grep kafka

docker service update --env-add KAFKA_ADVERTISED_HOST_NAME=61.136.145.19 --env-add ZOOKEEPER_IP=61.136.145.19 --env-add KAFKA_ADVERTISED_PORT=20029 --env-add ZOOKEEPER_PORT=20026  m4yxp38h3jrs
```

##### 6、kafkacat

[github kafkacat](https://github.com/edenhill/kafkacat)

```
# Publish
kafkacat -b 192.168.1.2:32000,192.168.1.2:32001 -P -t test

# Consume
kafkacat -b 192.168.1.2:32000,192.168.1.2:32001 -C -t test
```

