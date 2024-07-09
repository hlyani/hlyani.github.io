# Ingress

[https://github.com/kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx)

[https://github.com/nginxinc/kubernetes-ingress](https://github.com/nginxinc/kubernetes-ingress)

# 一、安装

## 1、ingress-nginx-4.10.1.tgz

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

## 2、修改配置 

```
vim ingress-nginx/values.yaml

controller:
  image:
    registry: easzlab.io.local:5000
    image: ingress-nginx/controller
    tag: "v1.10.1"
  nodeSelector:
    nginx/ingress: ai
  admissionWebhooks:
    enabled: false
  allowSnippetAnnotations: true
  metrics:
    port: 10254
    portName: metrics
    enabled: true
    service:
      annotations:
       prometheus.io/scrape: "true"
       prometheus.io/port: "10254"
    serviceMonitor:
      enabled: true
      additionalLabels:
        release: "ai-kube-prometheus-stack"
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "10254"
    prometheusRule:
      enabled: true
      additionalLabels:
        release: "ai-kube-prometheus-stack"
```

> enable-vts-status: "true" ，以导出 Prometheus 指标.
> prometheus.io/scrape: "true" ，以启用自动发现.
> prometheus.io/port: "10254" ，以指定度量标准端口.
>
> admissionWebhooks.enabled.false  如果为true创建ingress会报错

## 3、部署

```
helm install -n kube-system nginx-ingress .
```

## 4、检查

```
kubectl get servicemonitor -A|grep ingress
```

```
kubectl get svc -A|grep ingress
```

```
kubectl get po -A|grep ingress
```

# 二、验证

## 1、创建测试应用

> vim example_nginx.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment-1
  labels:
    app: nginx-1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-1
  template:
    metadata:
      labels:
        app: nginx-1
    spec:
      containers:
      - name: nginx
        image: easzlab.io.local:5000/nginx:1.21.3
        command: ["/bin/sh", "-c", "mkdir /usr/share/nginx/html/v1;echo 'hello nginx-1'>/usr/share/nginx/html/v1/index.html;nginx -g 'daemon off;'"]
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment-2
  labels:
    app: nginx-2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-2
  template:
    metadata:
      labels:
        app: nginx-2
    spec:
      containers:
      - name: nginx
        image: easzlab.io.local:5000/nginx:1.21.3
        command: ["/bin/sh", "-c", "mkdir /usr/share/nginx/html/v2;echo 'hello nginx-2'>/usr/share/nginx/html/v2/index.html;nginx -g 'daemon off;'"]
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx1-svc
  labels:
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: nginx-1
---
apiVersion: v1
kind: Service
metadata:
  name: nginx2-svc
  labels:
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: nginx-2
```

```
docker pull kennethreitz/httpbin
kubectl run httpbin --image kennethreitz/httpbin --port 80
kubectl expose pod httpbin --port 80

curl --location -H "Host: k8s.ingress.org" --request GET "http://10.15.200.45:32431/get?foo1=bar1&foo2=bar2"
```

## 2、创建ingress

```
vim ingress-k8s.yaml
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: k8s-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: k8s.ingress.org
    http:
      paths:
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: nginx1-svc
            port:
              number: 80
      - path: /v2
        pathType: Prefix
        backend:
          service:
            name: nginx2-svc
            port:
              number: 80
```

> ```yaml
>   annotations:
>     nginx.ingress.kubernetes.io/rewrite-target: /$2
>     nginx.ingress.kubernetes.io/proxy-connect-timeout: 10s
>     nginx.ingress.kubernetes.io/rewrite-target: /$1
> ```
>
> ```
> - path: /foo(/|$)(.*)
> ```

## 3、配置hosts

```
kubectl get ingress|grep test-ingress
test-ingress            nginx   testnginx.com             80      36m
```

```
kubectl get po -A -o wide|grep ingress
kube-system   ai-nginx-ingress-ingress-nginx-controller-6fc498bdb7-8js29     1/1     Running       0              38m     172.22.163.246   192.168.0.127   <none>           <none>
```

```
vim /etc/hosts
172.22.163.246 testnginx.com
```

## 4、测试

```
curl testnginx.com/v1
curl testnginx.com/v2
```

# 三、主要监控指标

[https://github.com/kubernetes/ingress-nginx/blob/helm-chart-4.10.1/docs/user-guide/monitoring.md](https://github.com/kubernetes/ingress-nginx/blob/helm-chart-4.10.1/docs/user-guide/monitoring.md)

[https://grafana.com/grafana/dashboards/14314-kubernetes-nginx-ingress-controller-nextgen-devops-nirvana/](https://grafana.com/grafana/dashboards/14314-kubernetes-nginx-ingress-controller-nextgen-devops-nirvana/)

[https://grafana.com/grafana/dashboards/?search=ingress-nginx](https://grafana.com/grafana/dashboards/?search=ingress-nginx)

[https://grafana.com/api/dashboards/14314/revisions/2/download](https://grafana.com/api/dashboards/14314/revisions/2/download)

[https://grafana.com/api/dashboards/20510/revisions/1/download](https://grafana.com/api/dashboards/20510/revisions/1/download)

```
nginx_ingress_controller_requests
```

# 四、config

```
kubectl edit configmap -n kube-system ai-nginx-ingress-ingress-nginx-controller
```

```
apiVersion: v1
data:
  allow-snippet-annotations: "true"
  plugins: hello_world, hello_hl
```

```
vim ingress-k8s.yaml
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: k8s-ingress
  annotations:
    nginx.ingress.kubernetes.io/server-snippet: |
      gzip on;
      gzip_comp_level 5;
      gzip_min_length 256;
      gzip_proxied any;
      gzip_vary on;
      gzip_types
        application/atom+xml
        application/javascript
        application/x-javascript
        application/json
        application/rss+xml
        application/vnd.ms-fontobject
        application/x-font-ttf
        application/x-web-app-manifest+json
        application/xhtml+xml
        application/xml
        font/opentype
        image/svg+xml
        image/x-icon
        text/css
        text/plain
        text/x-component;
```

