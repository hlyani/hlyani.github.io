# Containerd

# 一、Containerd

TODO

# 二、ctr

##### 1、拉取

```
ctr i pull docker.io/library/ubuntu:latest
ctr -n k8s.io i pull --plain-http=true 192.168.0.31:30002/library/ubuntu_armv8_edge:1.1 
https_proxy=http://192.168.0.169:1080 http_proxy=http://192.168.0.169:1080 ctr -n k8s.io i pull k8s.gcr.io/pause:3.2
```

##### 2、查看

```
ctr c ls
ctr -n k8s.io i ls -q
```

##### 3、运行

```
ctr run -t docker.io/library/ubuntu:latest test bash
ctr run -d docker.io/library/ubuntu:latest test bash
```

##### 4、删除

```
ctr c rm test
```

##### 5、导入导出镜像

```
bzip2 -cd aaa_v1.0.0_image_amd64.tar.bz2 | k3s ctr -n k8s.io i import -
```

```
docker save aaa:v1.0.0 | bzip2 > aaa_v1.0.0_image_amd64.tar.bz2
```

# 三、crictl

TODO

# 四、其他

##### 1、解压OCI镜像

```
ctr i export - docker.io/library/vm1:1.0.0 | tar xvf -
find blobs/sha256/* -size +4k |xargs -n 1 tar -xvf
```

