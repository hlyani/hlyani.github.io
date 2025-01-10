# 更新当前集群 ssl 证书

# 1、备份

```
cp -r /etc/kubeasz/clusters/tmp/ssl /etc/kubeasz/clusters/k8s/ssl_backup
```

# 2、编辑kubernetes-csr.json文件

```JSON
vi /etc/kubeasz/clusters/tmp/ssl/kubernetes-csr.json

{
  "CN": "kubernetes",
  "hosts": [
    "127.0.0.1",
    "10.0.0.127",
    "kubernetes",
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "HangZhou",
      "L": "XS",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

# 3、重新生成证书

```
cd /etc/kubeasz/clusters/tmp/ssl/

/etc/kubeasz/bin/cfssl gencert \
        -ca=ca.pem \
        -ca-key=ca-key.pem \
        -config=ca-config.json \
        -profile=kubernetes kubernetes-csr.json | /etc/kubeasz/bin/cfssljson -bare kubernetes
```

# 4、分发证书到所有 master 节点

```
scp /etc/kubeasz/clusters/tmp/ssl/kubernetes.pem root@xxx:/etc/kubernetes/ssl/
scp /etc/kubeasz/clusters/tmp/ssl/kubernetes-key.pem root@xxx:/etc/kubernetes/ssl/
```

# 5、重启 master 节点 k8s 服务

```
systemctl restart kube-apiserver
systemctl restart kube-controller-manager.service
systemctl restart kube-scheduler.service
```

# 6、验证

```
修改  ~/.kube/config
```

```
kubectl get node
```

