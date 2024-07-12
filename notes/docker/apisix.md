# APISIX

# 一、安装

## 1、chart 包 

```
helm repo add apisix https://charts.apiseven.com
helm repo update
helm pull apisix/apisix-ingress-controller
helm install apisix apisix/apisix --create-namespace --namespace ingress-apisix
```

## 2、镜像

```
docker pull busybox:1.28
docker pull apache/apisix:3.9.1-debian
docker pull bitnami/etcd:3.5.10-debian-11-r2
docker pull apache/apisix-ingress-controller:1.8.0
docker pull apache/apisix-dashboard:3.0.0-alpine
```

## 3、配置

```
vim apisix/values.yaml

image:
  repository: easzlab.io.local:5000/apache/apisix
replicaCount: 1
nodeSelector:
  nginx/ingress: ai
initContainer:
  image: easzlab.io.local:5000/busybox
metrics:
  serviceMonitor:
    enabled: true
    namespace: "ingress-apisix"
    labels:
      release: "ai-kube-prometheus-stack"
  prometheus:
    enabled: true
etcd:
  enabled: true
dashboard:
  enabled: true
ingress-controller:
  enabled: true
  config:
    apisix:
      adminAPIVersion: "v3"
```

```
vim apisix/charts/etcd/values.yaml

image:
  registry: easzlab.io.local:5000
replicaCount: 1
nodeSelector:
  nginx/ingress: ai
persistence:
  enabled: false
```

```
vim apisix/charts/apisix-dashboard/values.yaml

replicaCount: 1
image:
  repository: easzlab.io.local:5000/apache/apisix-dashboard
nodeSelector:
  nginx/ingress: ai
```

```
vim apisix/charts/apisix-ingress-controller/values.yaml

replicaCount: 1
image:
  repository: easzlab.io.local:5000/apache/apisix-ingress-controller
  apisix:
    serviceNamespace: ingress-apisix
    adminAPIVersion: "v3"
initContainer:
  image: easzlab.io.local:5000/busybox
nodeSelector:
  nginx/ingress: ai
serviceMonitor:
  enabled: true
  namespace: "ingress-apisix"
  labels:
    release: "ai-kube-prometheus-stack"
```

## 4、部署

``` 
helm install -n ingress-apisix apisix .
```

# 二、Ingress

## 1、ingress

> vim ingress-apisix.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apisix-ingress
spec:
  ingressClassName: apisix
  rules:
    - host: apisix.ingress.org
      http:
        paths:
          - backend:
              service:
                name: nginx1-svc
                port:
                  number: 80
            path: /v1
            pathType: Prefix
          - backend:
              service:
                name: nginx2-svc
                port:
                  number: 80
            path: /v2
            pathType: Prefix
          - backend:
              service:
                name: httpbin
                port:
                  number: 80
            path: /get
            pathType: Prefix
```

## 2、ApisixRoute

```
apiVersion: apisix.apache.org/v2
kind: ApisixRoute
metadata:
  name: httpserver-route
spec:
  http:
  - name: rule1
    match:
      hosts:
      - httpbin.example.com
      paths:
      - /*
    backends:
       - serviceName: httpbin
         servicePort: 80
```

> 配置多个域名和路径

```
  - name: rule1
    match:
      hosts:
      - local.example.com
      - local01.example.com
      paths:
      - /*
      - /api
    backends:
       - serviceName: httpbin
             servicePort: 80
```

> regex

```
  - name: httpserver-route
    match:
      hosts:
      - local.example.com
      paths:
      - /api*
    backends:
    - serviceName: httpbin
      servicePort: 8000
    plugins:
    - name: proxy-rewrite 
      enable: true
      config: 
        regex_uri: ["^/api(/|$)(.*)", "/$2"]
```

> gzip

```
      plugins:
        - name: gzip
          enable: true
```

> http to https

```
    plugins:
    - name: redirect
      enable: true
      config: 
        http_to_https: true
```

> 域名跳转

```
    plugins:
    - name: redirect
      enable: true
      config: 
        uri: "https://local01.example.com$request_uri"
```

> rewrite路径跳转，/api/header -> /header

```
    plugins:
    - name: redirect
      enable: true
      config: 
        regex_uri: ["^/api(/|$)(.*)", "/$2"] 
```

```
    plugins:
    - name: proxy-rewrite 
      enable: true
      config: 
        regex_uri: ["^/api(/|$)(.*)", "/$2"] 
```

> /header -> /api/header

```
    plugins:
    - name: proxy-rewrite 
      enable: true
      config: 
        uri: /api/$uri
```

> 基于用户名和密码认证

```
apiVersion: apisix.apache.org/v2
kind: ApisixConsumer
metadata:
  name: httpserver-basicauth
spec:
  authParameter:
    basicAuth:
      value:
        username: admin
        password: admin
---
apiVersion: apisix.apache.org/v2
kind: ApisixRoute
metadata:
  name: httpserver-route
spec:
  http:
  - name: httpserver-route
    match:
      hosts:
      - local.example.com
      paths:
      - /*
    backends:
    - serviceName: httpbin
      servicePort: 8000
    authentication:
      enable: true 
      type: basicAuth
```

```
curl -i -uadmin:admin https://local.example.com/
```

# 三、instance

```
kubectl run httpbin --image kennethreitz/httpbin --port 80
kubectl expose pod httpbin --port 80
```

```
curl --location -H "Host: apisix.ingress.org" --request GET "http://192.168.0.127:32431/get?foo1=bar1&foo2=bar2"
```

> example-nginx.yaml

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

# 四、监控

```
curl -i http://10.66.225.229:9091/apisix/prometheus/metrics
```

> 主要指标

[https://apisix.apache.org/docs/apisix/plugins/prometheus/#using-grafana-to-graph-the-metrics](https://apisix.apache.org/docs/apisix/plugins/prometheus/#using-grafana-to-graph-the-metrics)

[https://apisix.apache.org/zh/docs/apisix/plugins/prometheus/](https://apisix.apache.org/zh/docs/apisix/plugins/prometheus/)

```
apisix_http_status
apisix_bandwidth
apisix_http_requests_total
apisix_nginx_http_current_connections
```

# 五、Annotations and Config

[https://apisix.apache.org/zh/docs/ingress-controller/concepts/annotations/](https://apisix.apache.org/zh/docs/ingress-controller/concepts/annotations/)

[https://github.com/apache/apisix/blob/master/conf/config-default.yaml](https://github.com/apache/apisix/blob/master/conf/config-default.

# 六、使用

## 1、dashbord

> port

```
kubectl get svc -n ingress-apisix -o jsonpath="{.spec.ports[0].nodePort}" apisix-dashboard
```

> username/password

```
admin/admin
```

## 2、gateway 连接信息

```
export NODE_PORT=$(kubectl get -n ingress-apisix -o jsonpath="{.spec.ports[0].nodePort}" services apisix-gateway)
export NODE_IP=$(kubectl get nodes -n ingress-apisix -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT
```

## 3、查看已注册路由信息

```
curl "http://10.66.73.199:9180/apisix/admin/routes?page=1&page_size=10" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X GET
```

## 4、gzip

```
_meta:
  disable: false
buffers.number: 32
buffers.size: 4096
comp_level: 5
http_version: 1.1
min_length: 20
types: '*'
vary: true
```

> enable plugin

```
curl -i http://127.0.0.1:9180/apisix/admin/routes/1  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/index.html",
    "plugins": {
        "gzip": {
            "buffers": {
                "number": 8
            }
        }
    },
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    }
}'
```

> delete plugin

```
curl http://127.0.0.1:9180/apisix/admin/routes/1  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/index.html",
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    }
}'
```

> 验证

> Accept-Encoding:  gzip, deflate
> Content-Encoding: gzip

```
curl apisix.ingress.org:30962/index.html -i -H "Accept-Encoding: gzip"
```

## 5、http2

> config.yaml

```
apisix:
  enable_http2: true
```

## 6、header

```
k8s.apisix.apache.org/response-rewrite-add-header: "testkey1:testval1,testkey2:testval2"
```

```
k8s.apisix.apache.org/response-rewrite-set-header: "testkey1:testval1,testkey2:testval2"
```

```
k8s.apisix.apache.org/response-rewrite-remove-header: "testkey1,testkey2"
```

## 7、upstream timeout

```
  annotations:
    k8s.apisix.apache.org/upstream-read-timeout.: "5s"
    k8s.apisix.apache.org/upstream-connect-timeout: "10s"
    k8s.apisix.apache.org/upstream-send-timeout: "10s"
```

## 8、http-to-https

```
  annotations:
    k8s.apisix.apache.org/http-to-https: "true"
```

## 9、use-regex

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apisix-ingress
  annotations:
    k8s.apisix.apache.org/use-regex: "true"
spec:
  ingressClassName: apisix
  rules:
    - host: apisix.ingress.org
      http:
        paths:
          - backend:
              service:
                name: nginx1-svc
                port:
                  number: 80
            path: /v1/.*/action1
            pathType: ImplementationSpecific
```

## 10、websocket

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apisix-ingress
  annotations:
    k8s.apisix.apache.org/enable-websocket: "true"
spec:
  ingressClassName: apisix
  rules:
    - host: apisix.ingress.org
      http:
        paths:
          - backend:
              service:
                name: websocket
                port:
                  number: 8765
            path: /ws
            pathType: Exact
```

> client

```
import asyncio
import websockets

async def main():
  async with websockets.connect("ws://localhost:8765") as websocket:
    message = "Hello, server"
    await websocket.send(message)
    print(f"Sent: {message}")
    response = await websocket.recv()
    print(f"Received: {response}")

# asyncio.run(main()) # python3
asyncio.get_event_loop().run_until_complete(main())
```

> server

```
import asyncio
import websockets

# 连接处理器
async def echo(websocket, path):
    async for message in websocket:
        print(message)
        await websocket.send(message)

# 启动服务器
start_server = websockets.serve(echo, "0.0.0.0", 8765)

# 创建一个事件循环并运行服务器
asyncio.get_event_loop().run_until_complete(start_server)
asyncio.get_event_loop().run_forever()
```

```
FROM python:3.12.4-alpine3.20

COPY socket_server.py /opt/

RUN pip install websockets -i https://pypi.tuna.tsinghua.edu.cn/simple

CMD ["python", "/opt/socket_server.py"]
```

```
kubectl run websocket --image easzlab.io.local:5000/websocket-test:latest --port 8765
kubectl expose pod websocket --port 8765
```

> postman

```
ws://apisix.ingress.org:30962/ws
```

# 七、wrk 压力测试

[https://github.com/wg/wrk/tree/master/scripts](https://github.com/wg/wrk/tree/master/scripts)

```
yum -y install gcc openssl-devel git
git clone https://github.com/wg/wrk.git wrk
cd wrk
make
cp wrk /usr/bin/
```

> -t, --threads 线程
>
> -c, --connections 连接数
>
> -d, --duration 时间
>
> --latency   Print latency statistics

```
wrk -t10 -c1000 -d1h --latency http://apisix.ingress.org:30962/index.html
```

> Avg 平均值
>
> Stdev 标准方差
>
> Max 最大值
>
> Latency 延迟
>
> Req/Sec 每秒请求数
>
> Latency Distribution 延迟分布
>
> Requests/sec 平均每秒处理完成请求个数
>
> Transfer/sec 平均每秒读取数据量

```
Running 60m test @ http://apisix.ingress.org:30962/index.html
  10 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    48.95ms   78.86ms   2.00s    90.06%
    Req/Sec     1.69k   334.68     3.68k    68.25%
  Latency Distribution
     50%   23.36ms
     75%   31.49ms
     90%  126.79ms
     99%  379.73ms
  60512663 requests in 60.00m, 222.33GB read
  Socket errors: connect 0, read 2102, write 16, timeout 27480
  Non-2xx or 3xx responses: 36
Requests/sec:  16808.64
Transfer/sec:  63.24MB
```

[https://www.nginx-cn.net/blog/testing-performance-nginx-ingress-controller-kubernetes/](https://www.nginx-cn.net/blog/testing-performance-nginx-ingress-controller-kubernetes/)

# 八、FAQ

[https://apisix.apache.org/zh/docs/apisix/FAQ/](https://apisix.apache.org/zh/docs/apisix/FAQ/)



​	

