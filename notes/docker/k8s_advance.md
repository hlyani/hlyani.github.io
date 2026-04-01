# K8S 进阶

# 一、kubectx

https://github.com/ahmetb/kubectx/releases/download/v0.11.0/kubectx_v0.11.0_linux_x86_64.tar.gz

> /hl/config

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdxxxx
    server: https://10.0.0.1:6443
  name: aa
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDxxx
    server: https://10.0.1.1:6443
  name: bb
contexts:
- context:
    cluster: aa
    namespace: hl
    user: aa-admin
  name: ctx-aa
- context:
    cluster: bb
    user: bb-admin
  name: ctx-bb
current-context: ctx-aa
kind: Config
users:
- name: aa-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUxxxx
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJxx
- name: bb-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0Fxxx
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSxxx
```

```
export KUBECONFIG=/hl/config
```

```
kubectx ctx-bb
```

# 二、kubens

https://github.com/ahmetb/kubectx/releases/download/v0.11.0/kubens_v0.11.0_linux_x86_64.tar.gz

```
kubectl config set-context --current --namespace=hl
```

```
kubens hl
```

