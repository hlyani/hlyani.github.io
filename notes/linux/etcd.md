# ETCD 相关

## 一、搭建

### 1、创建单独的网络

```
docker network create etcd --subnet 172.19.0.0/16
```

## 2、运行etcd0

```
docker run -d --name etcd0 --network etcd --ip 172.19.1.10 quay.io/coreos/etcd etcd \
-name etcd0 \
-advertise-client-urls http://172.19.1.10:2379,http://172.19.1.10:4001 \
-listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
-initial-advertise-peer-urls http://172.19.1.10:2380 \
-listen-peer-urls http://0.0.0.0:2380 \
-initial-cluster-token etcd-cluster-1 \
-initial-cluster etcd0=http://172.19.1.10:2380,etcd1=http://172.19.1.11:2380,etcd2=http://172.19.1.12:2380 \
-initial-cluster-state new
```

## 3、运行etcd1

```
docker run -d --name etcd1 --network etcd --ip 172.19.1.11 quay.io/coreos/etcd etcd \
-name etcd1 \
-advertise-client-urls http://172.19.1.11:2379,http://172.19.1.11:4001 \
-listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
-initial-advertise-peer-urls http://172.19.1.11:2380 \
-listen-peer-urls http://0.0.0.0:2380 \
-initial-cluster-token etcd-cluster-1 \
-initial-cluster etcd0=http://172.19.1.10:2380,etcd1=http://172.19.1.11:2380,etcd2=http://172.19.1.12:2380 \
-initial-cluster-state new
```

## 4、运行etcd2

```
docker run -d --name etcd2 --network etcd --ip 172.19.1.12 quay.io/coreos/etcd etcd \
-name etcd2 \
-advertise-client-urls http://172.19.1.12:2379,http://172.19.1.12:4001 \
-listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
-initial-advertise-peer-urls http://172.19.1.12:2380 \
-listen-peer-urls http://0.0.0.0:2380 \
-initial-cluster-token etcd-cluster-1 \
-initial-cluster etcd0=http://172.19.1.10:2380,etcd1=http://172.19.1.11:2380,etcd2=http://172.19.1.12:2380 \
-initial-cluster-state new
```

# 二、使用

## 1、进入容器

```
docker run -it --name client --network etcd quay.io/coreos/etcd sh
```

## 2、etcd2

```
etcdctl --endpoints http://172.19.1.10:2379,http://172.19.1.11:2379,http://172.19.1.12:2379 set /foo bar
etcdctl --endpoints http://172.19.1.10:2379,http://172.19.1.11:2379,http://172.19.1.12:2379 get /foo
```

## 3、etcd3

```
ETCDCTL_API=3 etcdctl --endpoints http://172.19.1.10:2379,http://172.19.1.11:2379,http://172.19.1.12:2379 put foo bar
ETCDCTL_API=3 etcdctl --endpoints http://172.19.1.10:2379,http://172.19.1.11:2379,http://172.19.1.12:2379 get foo
```

## 4、查看状态

```
etcdctl --endpoints http://172.19.1.10:2379 endpoint status
etcdctl --endpoints http://172.19.1.10:2379 endpoint status -w table
etcdctl --endpoints http://172.19.1.10:2379 endpoint health
etcdctl --endpoints http://172.19.1.10:2379 endpoint health -w table
etcdctl --endpoints http://172.19.1.10:2379 member list -w table
```

