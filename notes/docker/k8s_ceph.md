# k8s 使用 ceph 储存

# 一、latest

## 1、创建pool

```
ceph osd pool create k8s
rbd pool init k8s
```

## 2、配置 ceph-csi

##### 1、生成 ceph client 认证

```
ceph auth get-or-create client.k8s mon 'profile rbd' osd 'profile rbd pool=k8s' mgr 'profile rbd pool=k8s'
[client.k8s]
    key = AQD9o0Fd6hQRChAAt7fMaSZXduT3NWEqylNpmg==
```

##### 2、生成 ceph-csi configmap

```
ceph mon dump
<...>
fsid b9127830-b0cc-4e34-aa47-9d1a2e9949a8
<...>
0: [v2:192.168.1.1:3300/0,v1:192.168.1.1:6789/0] mon.a
1: [v2:192.168.1.2:3300/0,v1:192.168.1.2:6789/0] mon.b
2: [v2:192.168.1.3:3300/0,v1:192.168.1.3:6789/0] mon.c
```

```
cat <<EOF > csi-config-map.yaml
---
apiVersion: v1
kind: ConfigMap
data:
  config.json: |-
    [
      {
        "clusterID": "b9127830-b0cc-4e34-aa47-9d1a2e9949a8",
        "monitors": [
          "192.168.1.1:6789",
          "192.168.1.2:6789",
          "192.168.1.3:6789"
        ]
      }
    ]
metadata:
  name: ceph-csi-config
EOF
```

```
kubectl apply -f csi-config-map.yaml
```

```
cat <<EOF > csi-kms-config-map.yaml
---
apiVersion: v1
kind: ConfigMap
data:
  config.json: |-
    {}
metadata:
  name: ceph-csi-encryption-kms-config
EOF
```

```
kubectl apply -f csi-kms-config-map.yaml
```

```
cat <<EOF > ceph-config-map.yaml
---
apiVersion: v1
kind: ConfigMap
data:
  ceph.conf: |
    [global]
    auth_cluster_required = cephx
    auth_service_required = cephx
    auth_client_required = cephx
  # keyring is a required key and its value should be empty
  keyring: |
metadata:
  name: ceph-config
EOF
```

```
kubectl apply -f ceph-config-map.yaml
```

##### 3、生成 ceph-csi cephx secret

```
cat <<EOF > csi-rbd-secret.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: csi-rbd-secret
  namespace: default
stringData:
  userID: k8s
  userKey: AQD9o0Fd6hQRChAAt7fMaSZXduT3NWEqylNpmg==
EOF
```

```
kubectl apply -f csi-rbd-secret.yaml
```

##### 4、配置 ceph-csi 插件

```
cat <<EOF > csi-provisioner-rbac.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: rbd-csi-provisioner
  # replace with non-default namespace name
  namespace: default

---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rbd-external-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["list", "watch", "create", "update", "patch"]
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "update", "delete", "patch"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims/status"]
    verbs: ["update", "patch"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["snapshot.storage.k8s.io"]
    resources: ["volumesnapshots"]
    verbs: ["get", "list", "patch"]
  - apiGroups: ["snapshot.storage.k8s.io"]
    resources: ["volumesnapshots/status"]
    verbs: ["get", "list", "patch"]
  - apiGroups: ["snapshot.storage.k8s.io"]
    resources: ["volumesnapshotcontents"]
    verbs: ["create", "get", "list", "watch", "update", "delete", "patch"]
  - apiGroups: ["snapshot.storage.k8s.io"]
    resources: ["volumesnapshotclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["volumeattachments"]
    verbs: ["get", "list", "watch", "update", "patch"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["volumeattachments/status"]
    verbs: ["patch"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["csinodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["snapshot.storage.k8s.io"]
    resources: ["volumesnapshotcontents/status"]
    verbs: ["update", "patch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get"]
  - apiGroups: [""]
    resources: ["serviceaccounts"]
    verbs: ["get"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rbd-csi-provisioner-role
subjects:
  - kind: ServiceAccount
    name: rbd-csi-provisioner
    # replace with non-default namespace name
    namespace: default
roleRef:
  kind: ClusterRole
  name: rbd-external-provisioner-runner
  apiGroup: rbac.authorization.k8s.io

---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  # replace with non-default namespace name
  namespace: default
  name: rbd-external-provisioner-cfg
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list", "watch", "create", "update", "delete"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["get", "watch", "list", "delete", "update", "create"]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rbd-csi-provisioner-role-cfg
  # replace with non-default namespace name
  namespace: default
subjects:
  - kind: ServiceAccount
    name: rbd-csi-provisioner
    # replace with non-default namespace name
    namespace: default
roleRef:
  kind: Role
  name: rbd-external-provisioner-cfg
  apiGroup: rbac.authorization.k8s.io
EOF
```

```
cat <<EOF > csi-nodeplugin-rbac.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: rbd-csi-nodeplugin
  # replace with non-default namespace name
  namespace: default
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rbd-csi-nodeplugin
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get"]
  # allow to read Vault Token and connection options from the Tenants namespace
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get"]
  - apiGroups: [""]
    resources: ["serviceaccounts"]
    verbs: ["get"]
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["volumeattachments"]
    verbs: ["list", "get"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rbd-csi-nodeplugin
subjects:
  - kind: ServiceAccount
    name: rbd-csi-nodeplugin
    # replace with non-default namespace name
    namespace: default
roleRef:
  kind: ClusterRole
  name: rbd-csi-nodeplugin
  apiGroup: rbac.authorization.k8s.io
EOF
```

```
kubectl apply -f https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-provisioner-rbac.yaml
kubectl apply -f https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-nodeplugin-rbac.yaml
```

```
cat <<EOF > csi-rbdplugin-provisioner.yaml
---
kind: Service
apiVersion: v1
metadata:
  name: csi-rbdplugin-provisioner
  # replace with non-default namespace name
  namespace: default
  labels:
    app: csi-metrics
spec:
  selector:
    app: csi-rbdplugin-provisioner
  ports:
    - name: http-metrics
      port: 8080
      protocol: TCP
      targetPort: 8680

---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: csi-rbdplugin-provisioner
  # replace with non-default namespace name
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: csi-rbdplugin-provisioner
  template:
    metadata:
      labels:
        app: csi-rbdplugin-provisioner
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - csi-rbdplugin-provisioner
              topologyKey: "kubernetes.io/hostname"
      serviceAccountName: rbd-csi-provisioner
      priorityClassName: system-cluster-critical
      containers:
        - name: csi-provisioner
          image: k8s.gcr.io/sig-storage/csi-provisioner:v3.1.0
          args:
            - "--csi-address=$(ADDRESS)"
            - "--v=5"
            - "--timeout=150s"
            - "--retry-interval-start=500ms"
            - "--leader-election=true"
            #  set it to true to use topology based provisioning
            - "--feature-gates=Topology=false"
            # if fstype is not specified in storageclass, ext4 is default
            - "--default-fstype=ext4"
            - "--extra-create-metadata=true"
          env:
            - name: ADDRESS
              value: unix:///csi/csi-provisioner.sock
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
        - name: csi-snapshotter
          image: k8s.gcr.io/sig-storage/csi-snapshotter:v5.0.1
          args:
            - "--csi-address=$(ADDRESS)"
            - "--v=5"
            - "--timeout=150s"
            - "--leader-election=true"
          env:
            - name: ADDRESS
              value: unix:///csi/csi-provisioner.sock
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
        - name: csi-attacher
          image: k8s.gcr.io/sig-storage/csi-attacher:v3.4.0
          args:
            - "--v=5"
            - "--csi-address=$(ADDRESS)"
            - "--leader-election=true"
            - "--retry-interval-start=500ms"
          env:
            - name: ADDRESS
              value: /csi/csi-provisioner.sock
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
        - name: csi-resizer
          image: k8s.gcr.io/sig-storage/csi-resizer:v1.4.0
          args:
            - "--csi-address=$(ADDRESS)"
            - "--v=5"
            - "--timeout=150s"
            - "--leader-election"
            - "--retry-interval-start=500ms"
            - "--handle-volume-inuse-error=false"
          env:
            - name: ADDRESS
              value: unix:///csi/csi-provisioner.sock
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
        - name: csi-rbdplugin
          # for stable functionality replace canary with latest release version
          image: quay.io/cephcsi/cephcsi:canary
          args:
            - "--nodeid=$(NODE_ID)"
            - "--type=rbd"
            - "--controllerserver=true"
            - "--endpoint=$(CSI_ENDPOINT)"
            - "--csi-addons-endpoint=$(CSI_ADDONS_ENDPOINT)"
            - "--v=5"
            - "--drivername=rbd.csi.ceph.com"
            - "--pidlimit=-1"
            - "--rbdhardmaxclonedepth=8"
            - "--rbdsoftmaxclonedepth=4"
            - "--enableprofiling=false"
          env:
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: NODE_ID
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            # - name: KMS_CONFIGMAP_NAME
            #   value: encryptionConfig
            - name: CSI_ENDPOINT
              value: unix:///csi/csi-provisioner.sock
            - name: CSI_ADDONS_ENDPOINT
              value: unix:///csi/csi-addons.sock
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
            - mountPath: /dev
              name: host-dev
            - mountPath: /sys
              name: host-sys
            - mountPath: /lib/modules
              name: lib-modules
              readOnly: true
            - name: ceph-csi-config
              mountPath: /etc/ceph-csi-config/
            - name: ceph-csi-encryption-kms-config
              mountPath: /etc/ceph-csi-encryption-kms-config/
            - name: keys-tmp-dir
              mountPath: /tmp/csi/keys
            - name: ceph-config
              mountPath: /etc/ceph/
        - name: csi-rbdplugin-controller
          # for stable functionality replace canary with latest release version
          image: quay.io/cephcsi/cephcsi:canary
          args:
            - "--type=controller"
            - "--v=5"
            - "--drivername=rbd.csi.ceph.com"
            - "--drivernamespace=$(DRIVER_NAMESPACE)"
          env:
            - name: DRIVER_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: ceph-csi-config
              mountPath: /etc/ceph-csi-config/
            - name: keys-tmp-dir
              mountPath: /tmp/csi/keys
            - name: ceph-config
              mountPath: /etc/ceph/
        - name: liveness-prometheus
          image: quay.io/cephcsi/cephcsi:canary
          args:
            - "--type=liveness"
            - "--endpoint=$(CSI_ENDPOINT)"
            - "--metricsport=8680"
            - "--metricspath=/metrics"
            - "--polltime=60s"
            - "--timeout=3s"
          env:
            - name: CSI_ENDPOINT
              value: unix:///csi/csi-provisioner.sock
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
          imagePullPolicy: "IfNotPresent"
      volumes:
        - name: host-dev
          hostPath:
            path: /dev
        - name: host-sys
          hostPath:
            path: /sys
        - name: lib-modules
          hostPath:
            path: /lib/modules
        - name: socket-dir
          emptyDir: {
            medium: "Memory"
          }
        - name: ceph-config
          configMap:
            name: ceph-config
        - name: ceph-csi-config
          configMap:
            name: ceph-csi-config
        - name: ceph-csi-encryption-kms-config
          configMap:
            name: ceph-csi-encryption-kms-config
        - name: keys-tmp-dir
          emptyDir: {
            medium: "Memory"
          }
EOF
```

```
cat <<EOF > csi-rbdplugin.yaml
---
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: csi-rbdplugin
  # replace with non-default namespace name
  namespace: default
spec:
  selector:
    matchLabels:
      app: csi-rbdplugin
  template:
    metadata:
      labels:
        app: csi-rbdplugin
    spec:
      serviceAccountName: rbd-csi-nodeplugin
      hostNetwork: true
      hostPID: true
      priorityClassName: system-node-critical
      # to use e.g. Rook orchestrated cluster, and mons' FQDN is
      # resolved through k8s service, set dns policy to cluster first
      dnsPolicy: ClusterFirstWithHostNet
      containers:
        - name: driver-registrar
          # This is necessary only for systems with SELinux, where
          # non-privileged sidecar containers cannot access unix domain socket
          # created by privileged CSI driver container.
          securityContext:
            privileged: true
          image: k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.4.0
          args:
            - "--v=5"
            - "--csi-address=/csi/csi.sock"
            - "--kubelet-registration-path=/var/lib/kubelet/plugins/rbd.csi.ceph.com/csi.sock"
          env:
            - name: KUBE_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
            - name: registration-dir
              mountPath: /registration
        - name: csi-rbdplugin
          securityContext:
            privileged: true
            capabilities:
              add: ["SYS_ADMIN"]
            allowPrivilegeEscalation: true
          # for stable functionality replace canary with latest release version
          image: quay.io/cephcsi/cephcsi:canary
          args:
            - "--nodeid=$(NODE_ID)"
            - "--pluginpath=/var/lib/kubelet/plugins"
            - "--stagingpath=/var/lib/kubelet/plugins/kubernetes.io/csi/pv/"
            - "--type=rbd"
            - "--nodeserver=true"
            - "--endpoint=$(CSI_ENDPOINT)"
            - "--csi-addons-endpoint=$(CSI_ADDONS_ENDPOINT)"
            - "--v=5"
            - "--drivername=rbd.csi.ceph.com"
            - "--enableprofiling=false"
            # If topology based provisioning is desired, configure required
            # node labels representing the nodes topology domain
            # and pass the label names below, for CSI to consume and advertise
            # its equivalent topology domain
            # - "--domainlabels=failure-domain/region,failure-domain/zone"
          env:
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: NODE_ID
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            # - name: KMS_CONFIGMAP_NAME
            #   value: encryptionConfig
            - name: CSI_ENDPOINT
              value: unix:///csi/csi.sock
            - name: CSI_ADDONS_ENDPOINT
              value: unix:///csi/csi-addons.sock
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
            - mountPath: /dev
              name: host-dev
            - mountPath: /sys
              name: host-sys
            - mountPath: /run/mount
              name: host-mount
            - mountPath: /etc/selinux
              name: etc-selinux
              readOnly: true
            - mountPath: /lib/modules
              name: lib-modules
              readOnly: true
            - name: ceph-csi-config
              mountPath: /etc/ceph-csi-config/
            - name: ceph-csi-encryption-kms-config
              mountPath: /etc/ceph-csi-encryption-kms-config/
            - name: plugin-dir
              mountPath: /var/lib/kubelet/plugins
              mountPropagation: "Bidirectional"
            - name: mountpoint-dir
              mountPath: /var/lib/kubelet/pods
              mountPropagation: "Bidirectional"
            - name: keys-tmp-dir
              mountPath: /tmp/csi/keys
            - name: ceph-logdir
              mountPath: /var/log/ceph
            - name: ceph-config
              mountPath: /etc/ceph/
        - name: liveness-prometheus
          securityContext:
            privileged: true
          image: quay.io/cephcsi/cephcsi:canary
          args:
            - "--type=liveness"
            - "--endpoint=$(CSI_ENDPOINT)"
            - "--metricsport=8680"
            - "--metricspath=/metrics"
            - "--polltime=60s"
            - "--timeout=3s"
          env:
            - name: CSI_ENDPOINT
              value: unix:///csi/csi.sock
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
          imagePullPolicy: "IfNotPresent"
      volumes:
        - name: socket-dir
          hostPath:
            path: /var/lib/kubelet/plugins/rbd.csi.ceph.com
            type: DirectoryOrCreate
        - name: plugin-dir
          hostPath:
            path: /var/lib/kubelet/plugins
            type: Directory
        - name: mountpoint-dir
          hostPath:
            path: /var/lib/kubelet/pods
            type: DirectoryOrCreate
        - name: ceph-logdir
          hostPath:
            path: /var/log/ceph
            type: DirectoryOrCreate
        - name: registration-dir
          hostPath:
            path: /var/lib/kubelet/plugins_registry/
            type: Directory
        - name: host-dev
          hostPath:
            path: /dev
        - name: host-sys
          hostPath:
            path: /sys
        - name: etc-selinux
          hostPath:
            path: /etc/selinux
        - name: host-mount
          hostPath:
            path: /run/mount
        - name: lib-modules
          hostPath:
            path: /lib/modules
        - name: ceph-config
          configMap:
            name: ceph-config
        - name: ceph-csi-config
          configMap:
            name: ceph-csi-config
        - name: ceph-csi-encryption-kms-config
          configMap:
            name: ceph-csi-encryption-kms-config
        - name: keys-tmp-dir
          emptyDir: {
            medium: "Memory"
          }
---
# This is a service to expose the liveness metrics
apiVersion: v1
kind: Service
metadata:
  name: csi-metrics-rbdplugin
  # replace with non-default namespace name
  namespace: default
  labels:
    app: csi-metrics
spec:
  ports:
    - name: http-metrics
      port: 8080
      protocol: TCP
      targetPort: 8680
  selector:
    app: csi-rbdplugin
EOF
```

```
wget https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-rbdplugin-provisioner.yaml
kubectl apply -f csi-rbdplugin-provisioner.yaml
wget https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-rbdplugin.yaml
kubectl apply -f csi-rbdplugin.yaml
```

## 3、使用 ceph 块设备

##### 1、创建 storageclass

```
cat <<EOF > csi-rbd-sc.yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: csi-rbd-sc
provisioner: rbd.csi.ceph.com
parameters:
   clusterID: b9127830-b0cc-4e34-aa47-9d1a2e9949a8
   pool: k8s
   imageFeatures: layering
   csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
   csi.storage.k8s.io/provisioner-secret-namespace: default
   csi.storage.k8s.io/controller-expand-secret-name: csi-rbd-secret
   csi.storage.k8s.io/controller-expand-secret-namespace: default
   csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
   csi.storage.k8s.io/node-stage-secret-namespace: default
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
   - discard
EOF
```

```
kubectl apply -f csi-rbd-sc.yaml
```

##### 2、创建 PesistentVolumeClaim

```
cat <<EOF > raw-block-pvc.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: raw-block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-rbd-sc
EOF
```

```
kubectl apply -f raw-block-pvc.yaml
```

```
cat <<EOF > raw-block-pod.yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-raw-block-volume
spec:
  containers:
    - name: fc-container
      image: fedora:26
      command: ["/bin/sh", "-c"]
      args: ["tail -f /dev/null"]
      volumeDevices:
        - name: data
          devicePath: /dev/xvda
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: raw-block-pvc
EOF
```

```
kubectl apply -f raw-block-pod.yaml
```

```
cat <<EOF > pvc.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rbd-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-rbd-sc
EOF
```

```
kubectl apply -f pvc.yaml
```

## 4、使用 demo

```
cat <<EOF > pod.yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: csi-rbd-demo-pod
spec:
  containers:
    - name: web-server
      image: nginx
      volumeMounts:
        - name: mypvc
          mountPath: /var/lib/www/html
  volumes:
    - name: mypvc
      persistentVolumeClaim:
        claimName: rbd-pvc
        readOnly: false
EOF
```

```
kubectl apply -f pod.yaml
```

# 二、before

### 一、介绍

[https://medium.com/velotio-perspectives/an-innovators-guide-to-kubernetes-storage-using-ceph-a4b919f4e469](https://medium.com/velotio-perspectives/an-innovators-guide-to-kubernetes-storage-using-ceph-a4b919f4e469)

##### 使用两种存储类型来与 k8s 集成

* 1、Ceph-RBD
* 2、CephFS

##### PV 与 PVC

> ​        PersistentVolume (持久卷， 简称 PV)和Persistent VolumeClaim(持久卷声明，简称 PVC)使得K8s集群具备了存储的逻辑抽象能力，使得在配置Pod的逻辑里可以忽略对实际后台存储技术的配置，而把这项配置的工作交给PV的配置者，即集群的管理者。存储的PV和PVC的这种关系，跟计算的Node和Pod的关系是非常类似的；PV和Node是资源的提供者，根据集群的基础设施变化而变化，由K8s集群管理员配置；而PVC和Pod是资源的使用者，根据业务服务的需求变化而变化，由K8s集群的使用者即服务的管理员来配置。

> ​        当集群用户需要在其pod中使用持久化存储时，他们首先创建PVC清单，指定所需要的最低容量要求和访问模式，然后用户将待久卷声明清单提交给Kubernetes API服务器，Kubernetes将找到可匹配的PV并将其绑定到PVC。PVC可以当作pod中的一个卷来使用，其他用户不能使用相同的PV，除非先通过删除PVC绑定来释放。

##### PV 的访问模型:

accessModes:

* ReadWriteOnce【简写:RWO】: 单路读写，即仅能有一个节点挂载读写，是最基本的方式，可读可写，但只支持被单个Pod挂载。
* ReadOnlyMany【ROX】: 多路只读，可以以只读的方式被多个Pod挂载。
* ReadWriteMany【RWX】：多路读写，这种存储可以以读写的方式被多个Pod共享。不是每一种存储都支持这三种方式，像共享方式，目前支持的还比较少，比较常用的是NFS。

##### **卷可以处于以下的某种状态：**

- Available（可用），一块空闲资源还没有被任何声明绑定
- Bound（已绑定），卷已经被声明绑定
- Released（已释放），声明被删除，但是资源还未被集群重新声明
- Failed（失败），该卷的自动回收失败

![pv_pvc](../../imgs/pv_pvc.png)

### 二、部署 Ceph（略）

### 三、Ceph-RBD

[https://github.com/ajaynemade/K8s-Ceph](https://github.com/ajaynemade/K8s-Ceph)

##### 1、安装 Ceph-RBD client

```
vim Ceph-RBD-Provisioner.yaml
```

```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rbd-provisioner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
  - apiGroups: [""]
    resources: ["services"]
    resourceNames: ["kube-dns","coredns"]
    verbs: ["list", "get"]
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rbd-provisioner
subjects:
  - kind: ServiceAccount
    name: rbd-provisioner
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: rbd-provisioner
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: rbd-provisioner
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rbd-provisioner
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: rbd-provisioner
subjects:
- kind: ServiceAccount
  name: rbd-provisioner
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: rbd-provisioner
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: rbd-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: rbd-provisioner
    spec:
      containers:
      - name: rbd-provisioner
        image: "quay.io/external_storage/rbd-provisioner:latest"
        env:
        - name: PROVISIONER_NAME
          value: ceph.com/rbd
      serviceAccount: rbd-provisioner
```

```
kubectl create -n kube-system -f  Ceph-RBD-Provisioner.yaml
```

> output

```
clusterrole.rbac.authorization.k8s.io/rbd-provisioner created
clusterrolebinding.rbac.authorization.k8s.io/rbd-provisioner created
role.rbac.authorization.k8s.io/rbd-provisioner created
rolebinding.rbac.authorization.k8s.io/rbd-provisioner created
serviceaccount/rbd-provisioner created
deployment.extensions/rbd-provisioner created
```

##### 2、检查 RBD volume 状态，等待状态为 Running

```
kubectl get pods -l app=rbd-provisioner -n kube-system
NAME                               READY     STATUS    RESTARTS   AGE
rbd-provisioner-857866b5b7-vc4pr   1/1       Running   0          16s
```

##### 3、等待运行起来过后，配置 admin key 作为认证

```
ceph auth get-key client.admin
AQDyWw9dOUm/FhAA4JCA9PXkPo6+OXpOj9N2ZQ==

kubectl create secret generic ceph-secret \
    --type="kubernetes.io/rbd" \
    --from-literal=key='AQDyWw9dOUm/FhAA4JCA9PXkPo6+OXpOj9N2ZQ==' \
    --namespace=kube-system
```

##### 4、为 k8s 创建一个 ceph pool，并且创建一个 client key

```
ceph --cluster ceph osd pool create k8s 1024 1024

ceph --cluster ceph auth get-or-create client.k8s mon 'allow r' osd 'allow rwx pool=k8s'
```

##### 5、获取创建的 auth token，并且为 kube pool 创建 k8s secret

```
ceph --cluster ceph auth get-key client.k8s
AQDabg9d4MBeIBAAaOhTjqsYpsNa4X10V0qCfw==

kubectl create secret generic ceph-secret-k8s \
    --type="kubernetes.io/rbd" \
    --from-literal=key="AQDabg9d4MBeIBAAaOhTjqsYpsNa4X10V0qCfw==" \
    --namespace=kube-system
```

#####  6、创建存储 class

```
vim Ceph-RBD-StorageClass.yaml
```

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: k8s
provisioner: ceph.com/rbd
# allowVolumeExpansion: true
parameters:
  monitors: 10.0.1.118:6789, 10.0.1.227:6789, 10.0.1.172:6789
  adminId: admin
  adminSecretName: ceph-secret
  adminSecretNamespace: kube-system
  pool: k8s
  userId: k8s
  userSecretName: ceph-secret-k8s
  userSecretNamespace: kube-system
  imageFormat: "2"
  imageFeatures: layering 
```

```
kubectl create -f Ceph-RBD-StorageClass.yaml
```

> allowVolumeExpansion: true，允许扩容
>
> bug：需要在pod/kube-controller-manager中安装ceph-mon，并拷贝ceph.client.admin.keyring到/etc/ceph/文件夹中。
>
> 如下编辑pvc，再重启pod即可实现扩容
>
> kubectl edit pvc data-kafka-0
>
> status:
>   accessModes:
>   - ReadWriteOnce
>     storage: 8Gi

##### 7、到此已经配置完。接下来通过创建 PVC 来测试 Ceph-RBD。

```
vim Ceph-RBD-PVC.yaml
```

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: testclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: k8s
```

```
kubectl create -f Ceph-RBD-PVC.yaml
```

##### 8、检查pvc，就会发现它表明它已与存储类创建的pv绑定。

```
kubectl get pvc
NAME      STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
testclaim  Bound     pvc-c215ad98-95b3-11e9-8b5d-12e154d66096   1Gi        RWO            fast-rbd       2m
```

##### 9、检查 persistent volume（pv）

```
kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM             STORAGECLASS   REASON    AGE
pvc-c215ad98-95b3-11e9-8b5d-12e154d66096   1Gi        RWO            Delete           Bound     default/testclaim   fast-rbd                 8m
```

### 四、CephFS

##### 1、创建一个单独的命名空间

```
kubectl create ns cephfs
```

##### 2、使用 ceph admin auth token，创建 k8s secret

```
ceph auth get-key client.admin
AQDyWw9dOUm/FhAA4JCA9PXkPo6+OXpOj9N2ZQ==

kubectl create secret generic ceph-secret-admin --from-literal=key="AQDyWw9dOUm/FhAA4JCA9PXkPo6+OXpOj9N2ZQ==" -n cephfs
```

##### 3、创建 cluster role、role bonding、provisioner

```
vim Ceph-FS-Provisioner.yaml
```

```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cephfs-provisioner
  namespace: cephfs
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
  - apiGroups: [""]
    resources: ["services"]
    resourceNames: ["kube-dns","coredns"]
    verbs: ["list", "get"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cephfs-provisioner
subjects:
  - kind: ServiceAccount
    name: cephfs-provisioner
    namespace: cephfs
roleRef:
  kind: ClusterRole
  name: cephfs-provisioner
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cephfs-provisioner
  namespace: cephfs
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["create", "get", "delete"]
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cephfs-provisioner
  namespace: cephfs
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cephfs-provisioner
subjects:
- kind: ServiceAccount
  name: cephfs-provisioner
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cephfs-provisioner
  namespace: cephfs
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: cephfs-provisioner
  namespace: cephfs
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: cephfs-provisioner
    spec:
      containers:
      - name: cephfs-provisioner
        image: "quay.io/external_storage/cephfs-provisioner:latest"
        env:
        - name: PROVISIONER_NAME
          value: ceph.com/cephfs
        - name: PROVISIONER_SECRET_NAMESPACE
          value: cephfs
        command:
        - "/usr/local/bin/cephfs-provisioner"
        args:
        - "-id=cephfs-provisioner-1"
      serviceAccount: cephfs-provisioner
```

```
kubectl create -n cephfs -f Ceph-FS-Provisioner.yaml
```

##### 4、创建 storage class

```
vim Ceph-FS-StorageClass.yaml
```

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: cephfs
provisioner: ceph.com/cephfs
parameters:
    monitors: 10.0.1.226:6789, 10.0.1.205:6789, 10.0.1.82:6789
    adminId: admin
    adminSecretName: ceph-secret-admin
    adminSecretNamespace: cephfs
    claimRoot: /pvc-volumes
```

```
kubectl create -f Ceph-FS-StorageClass.yaml
```

##### 5、到此配置完成。等待状态变为 Running

```
kubectl get pods -n cephfs
NAME                                 READY     STATUS    RESTARTS   AGE
cephfs-provisioner-8d957f95f-s7mdq   1/1       Running   0          1m
```

##### 6、CephFS提供程序启动后，请尝试创建持久卷声明。 在此步骤中，存储类将负责动态创建持久卷。

```
vim Ceph-FS-PVC.yaml
```

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim1
spec:
  storageClassName: cephfs
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

```
kubectl create -f Ceph-FS-PVC.yaml
```

##### 7、检查 pv 和 pvc

```
kubectl get pvc
NAME      STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
claim1    Bound     pvc-a7db18a7-9641-11e9-ab86-12e154d66096   1Gi        RWX            cephfs         2m

kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM            STORAGECLASS   REASON    AGE
pvc-a7db18a7-9641-11e9-ab86-12e154d66096   1Gi        RWX            Delete           Bound     default/claim1   cephfs                   2m
```

### 五、测试 CephFS
##### 1、创建 pvc

```
vim cephfs-pvc-test.yaml
```

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: cephfs-claim
spec:
  accessModes:     
    - ReadWriteOnce
  storageClassName: dynamic-cephfs
  resources:
    requests:
      storage: 2Gi
```

```
kubectl apply -f cephfs-pvc-test.yaml
```
##### 2、查看

```javascript
kubectl get pvc
kubectl get pv
```
##### 3、创建 nginx pod 挂载测试

```
vim nginx-pod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod1
  labels:
    name: nginx-pod1
spec:
  containers:
  - name: nginx-pod1
    image: nginx:alpine
    ports:
    - name: web
      containerPort: 80
    volumeMounts:
    - name: ceph-rdb
      mountPath: /usr/share/nginx/html
  volumes:
  - name: ceph-rdb
    persistentVolumeClaim:
      claimName: cephfs-claim
```

```
kubectl apply -f nginx-pod.yaml
```

##### 4、查看

```
kubectl get pods -o wide
```
##### 5、修改访问内容

```
kubectl exec -ti nginx-pod1 -- /bin/sh -c 'echo Hello World from Ceph RBD!!! > /usr/share/nginx/html/index.html'
```
##### 6、访问测试

```
POD_ID=$(kubectl get pods -o wide | grep nginx-pod1 | awk '{print $6}')
curl http://$POD_ID
```
##### 7、清理

```
kubectl delete -f nginx-pod.yaml
kubectl delete -f cephfs-pvc-test.yaml
```

### 六、测试  Ceph-RBD

```
# 创建pvc测试
cat >ceph-rdb-pvc-test.yaml<<EOF
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: ceph-rdb-claim
spec:
  accessModes:     
    - ReadWriteOnce
  storageClassName: dynamic-ceph-rdb
  resources:
    requests:
      storage: 2Gi
EOF
kubectl apply -f ceph-rdb-pvc-test.yaml
 
# 查看
kubectl get pvc
kubectl get pv
 
# 创建 nginx pod 挂载测试
cat >nginx-pod.yaml<<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod1
  labels:
    name: nginx-pod1
spec:
  containers:
  - name: nginx-pod1
    image: nginx:alpine
    ports:
    - name: web
      containerPort: 80
    volumeMounts:
    - name: ceph-rdb
      mountPath: /usr/share/nginx/html
  volumes:
  - name: ceph-rdb
    persistentVolumeClaim:
      claimName: ceph-rdb-claim
EOF
kubectl apply -f nginx-pod.yaml
 
# 查看
kubectl get pods -o wide
 
# 修改文件内容
kubectl exec -ti nginx-pod1 -- /bin/sh -c 'echo Hello World from Ceph RBD!!! > /usr/share/nginx/html/index.html'
 
# 访问测试
POD_ID=$(kubectl get pods -o wide | grep nginx-pod1 | awk '{print $6}')
curl http://$POD_ID

# 清理
kubectl delete -f nginx-pod.yaml
kubectl delete -f ceph-rdb-pvc-test.yaml
```

