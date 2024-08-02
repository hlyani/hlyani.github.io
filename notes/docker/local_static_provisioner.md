# local static provisioner

[https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner)

# 一、安装

## 1、拉取代码

```
git clone --depth=1 -b v2.7.0 https://github.com/kubernetes/kubernetes
```

```
helm repo add sig-storage-local-static-provisioner https://kubernetes-sigs.github.io/sig-storage-local-static-provisioner
```

```
helm template --debug sig-storage-local-static-provisioner/local-static-provisioner --version <version> --namespace <namespace> > local-volume-provisioner.generated.yaml

kubectl create -f local-volume-provisioner.generated.yaml
```

## 2、安装

```
helm install lp ./helm/provisioner
```

## 3、配置

vim values.yaml

```
classes:
  - name: local-storage
    hostDir: /opt/local-provisioner
    volumeMode: Filesystem
    fsType: ext4
    storageClass: true
image: registry.k8s.io/sig-storage/local-volume-provisioner:v2.6.0
```

> 必须mount，provisioner会实时检测hostDir目录下的mount目录，并自动创建pv

```
mkdir -p /opt/local-provisioner/pv0
mount --bind /opt/local-provisioner/pv0 /opt/local-provisioner/pv0
```

## 二、其他

根据本地文件夹创建 pv

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /opt/local-pv
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: ingress
          operator: In
          values:
          - master
```

创建pvc

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeMode: Filesystem
  storageClassName: local-storage
```

test

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: local-storage
---
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx:1.21.3
      volumeMounts:
      - mountPath: "/data"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

