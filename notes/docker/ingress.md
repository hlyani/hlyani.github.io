# Ingress

[https://github.com/kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx)

[https://github.com/nginxinc/kubernetes-ingress](https://github.com/nginxinc/kubernetes-ingress)

https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/
https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/configmap.md
https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/
https://github.com/bitnami/charts/tree/main/bitnami/nginx-ingress-controller

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

[https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#plugins](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#plugins)

[https://github.com/kubernetes/ingress-nginx/blob/helm-chart-4.10.1/docs/user-guide/nginx-configuration/annotations.md](https://github.com/kubernetes/ingress-nginx/blob/helm-chart-4.10.1/docs/user-guide/nginx-configuration/annotations.md)

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

# 五、gzip

| 配置项            | 作用                                                         | 示例                                                    |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------- |
| gzip              | 是否开启gzip压缩                                             | gzip on;                                                |
| gzip_types        | 指定要压缩的MIME类型                                         | gzip_types text/html text/plain application/javascript; |
| gzip_min_length   | 指定最小压缩文件大小                                         | gzip_min_length 1000;                                   |
| gzip_comp_level   | 指定压缩级别 范围为1到9,值越大压缩程度越大                   | gzip_comp_level 6;                                      |
| gzip_buffers      | 指定用于gzip压缩的内存缓冲区大小                             | gzip_buffers 16 8k;                                     |
| gzip_disable      | 指定不使用gzip压缩的User-Agent                               | gzip_disable “MSIE [1-6].(?!.*SV1)”;                    |
| gzip_proxied      | 根据客户端请求中的"Accept-Encoding"头部决定是否压缩响应，取值可以是 “off”、“expired”、“no-cache”、“no-store”、“private”、“no_last_modified”、“no_etag”、“auth” 或 “any” | gzip_proxied any；                                      |
| gzip_vary         | 如果发送的响应被gzip压缩，则在响应头部加上"Vary: Accept-Encoding"，以通知缓存服务器响应内容可能以压缩或非压缩形式存在 | gzip_vary:on;                                           |
| gzip_http_version | 设置进行gzip压缩的HTTP协议版本。                             | gzip_http_version:1.0                                   |

```
Accept-Encoding:  gzip, deflate
Content-Encoding: gzip
```

# 六、Websocket

```
server.py

import asyncio
import websockets
import logging

# 配置日志记录到标准输出
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(message)s")

async def echo(websocket, path):
    async for message in websocket:
        logging.info(f"Received message: {message}")
        await websocket.send(f"Echo: {message}")

# WebSocket 服务地址和端口
start_server = websockets.serve(echo, "0.0.0.0", 8765)

if __name__ == "__main__":
    logging.info("Starting WebSocket server...")
    asyncio.get_event_loop().run_until_complete(start_server)
    asyncio.get_event_loop().run_forever()
```

```
FROM python:3.8.14

COPY server.py /opt/server.py

RUN pip install websockets

CMD ["python", "/opt/server.py"]
```

```
docker build -t ingress_websocket .
```

```
kubectl run websocket --image easzlab.io.local:5000/ingress_websocket --port 8765
kubectl expose pod websocket --port 8765
```

```
ingress_websocket.py

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: websocket
  annotations:
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
    nginx.ingress.kubernetes.io/server-snippets: |
      location / {
        proxy_set_header Upgrade $http_upgrade;
        proxy_http_version 1.1;
        proxy_set_header Connection "upgrade";
      }
spec:
  ingressClassName: nginx
  rules:
    - host: websocket.aa.com
      http:
        paths:
          - backend:
              service:
                name: websocket
                port:
                  number: 8765
            path: /
            pathType: Exact
```

```
kubectl apply -f ingress_websocket.py
```

> ws://：用于普通 WebSocket 连接。
> wss://：用于安全 WebSocket 连接（SSL/TLS 支持）。

```
websockt 连接
wss://websocket.aa.com:8888
```

# 七、timeout

```
from flask import Flask
import time

app = Flask(__name__)

@app.route("/")
def slow_response():
    time.sleep(10)  # 模拟 10 秒的延迟
    return "Hello, World!"

if __name__ == "__main__":
    app.run(debug=True)
```

```
curl --max-time 10 --connect-timeout 2 -H "Host: timeout.aa.com" 192.168.0.127:32000/headers
```

# 八、性能参数优化

```
config:
  worker-processes: "auto"
  worker-cpu-affinity: "auto"
  max-worker-connections: "8192"
  reuse-port: "true"
  server-tokens: "false"
  ssl-redirect: "false"
  proxy-connect-timeout: "900"
  proxy-read-timeout: "900"
  proxy-send-timeout: "900"
  proxy-body-size: "10m"
  proxy-buffer-size: "16k"
  proxy-buffers-number: "4"
  upstream-keepalive-timeout: "900"
  upstream-keepalive-requests: "900"
  upstream-keepalive-connections: "900"
  keep-alive: "900"
  keep-alive-requests: "900"
  client-body-timeout: "900"
  client-body-buffer-size: "64k"
  use-http2: "true"
  use-gzip: "true"
  gzip-min-length: "1024"
  gzip-level: "6"
  gzip-types: "text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss"
  custom-http-errors: "400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,421,422,423,424,425,426,428,429,431,451,500,501,502,503,504,505,506,507,508,510,511"
```

# 九、性能测试

```
nginx.conf: |
  master_process on;

  worker_processes 1;
  events {
      worker_connections 4096;
  }

  http {
      access_log off;
      server_tokens off;
      keepalive_requests 10000000;

      server {
          listen 80;
          server_name _;

          location / {
              proxy_set_header Connection "";
              return 200 "hello world\n";
          }
      }
  }
```

```
image.ac.com:5000/k8s/bitnami/nginx:1.27.3-debian-12-r0

/opt/bitnami/nginx/conf/nginx.conf
```

```
kubectl expose pod websocket --port 80
```

```

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - host: ingress.sugon.fun
    http:
      paths:
      - backend:
          service:
            name: ingress-default-backend
            port:
              number: 80
        path: /
        pathType: Prefix
```

