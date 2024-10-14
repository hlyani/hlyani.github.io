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
docker pull apache/apisix-ingress-controller:1.8.2
docker pull apache/apisix-dashboard:3.0.1-alpine
```

```
docker save busybox:1.28 apache/apisix:3.9.1-debian bitnami/etcd:3.5.10-debian-11-r2 apache/apisix-ingress-controller:1.8.2 apache/apisix-dashboard:3.0.1-alpine |gzip > ingress-apisix_3.9.1_imgs.tar.gz
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
nodeSelector:
  kubernetes.io/hostname: 10.13.1.12
 
nodeSelector:
  nginx/ingress: ai
```

```
resources: 
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 16
    memory: 32Gi
```

```
tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "ingress"
    effect: "NoExecute"
```

```
affinity:
   podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - nginx-ingress
          topologyKey: kubernetes.io/hostname
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
  enabled: false
  namespace: "ingress-apisix"
  labels:
    release: "ai-kube-prometheus-stack"
```

修改Ingress class

```
vim charts/apisix-ingress-controller/values.yaml
```

```
.Values.config.kubernetes.ingressClass
```

```
 ingressClass: "apisix" ->  ingressClass: "nginx"
```

## 4、部署

``` 
helm install -n ingress-apisix apisix .
```

## 5、test

```
kubectl apply -f -<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-httpbin
  labels:
    app: test-httpbin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-httpbin
  template:
    metadata:
      labels:
        app: test-httpbin
    spec:
      containers:
      - name: nginx
        image: easzlab.io.local:5000/kennethreitz/httpbin
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: test-httpbin-svc
  labels:
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: test-httpbin
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-httpbin-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: apisix.ingress.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: test-httpbin-svc
            port:
              number: 80
EOF
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

httpbin-route.yaml

```
apiVersion: apisix.apache.org/v2
kind: ApisixRoute
metadata:
  name: httpbin-route
spec:
  http:
    - name: route-1
      match:
        hosts:
          - httpbin.org
        paths:
          - /*
      backends:
        - serviceName: httpbin
          servicePort: 80
```

```
curl httpbin.org:32060/headers
```

# 四、监控

```
curl -i http://$(kubectl get svc -n ingress-apisix apisix-prometheus-metrics -o jsonpath="{.spec.clusterIP}"):9091/apisix/prometheus/metrics
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

[https://github.com/apache/apisix/blob/release/3.3/conf/config-default.yaml](https://github.com/apache/apisix/blob/release/3.3/conf/config-default.yaml)

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

> bug
>
> [https://github.com/apache/apisix-dashboard/issues/2791](https://github.com/apache/apisix-dashboard/issues/2791)
>
> apisix dashboard与apisix的配置文件中都配置plugins

## 2、gateway 连接信息

```
export NODE_PORT=$(kubectl get -n ingress-apisix -o jsonpath="{.spec.ports[0].nodePort}" services apisix-gateway)
export NODE_IP=$(kubectl get nodes -n ingress-apisix -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT
```

```
export NODE_PORT=$(kubectl get -n ingress-apisix -o jsonpath="{.spec.ports[0].nodePort}" services apisix-gateway)
export apisix=$(echo http://apisix.ingress.org:$NODE_PORT)
```

## 3、查看已注册路由信息

```
curl "http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/routes?page=1&page_size=10" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X GET
```

```
curl -s http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/routes -H 'X-API-Key: edd1c9f034335f136f87ad84b625c8f1'|jq
```

## 4、API 使用

[admin-api](https://apisix.apache.org/zh/docs/apisix/admin-api/)

[control-api](https://apisix.apache.org/zh/docs/apisix/control-api/)

```
curl -s http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/global_rules -H 'X-API-Key: edd1c9f034335f136f87ad84b625c8f1'|jq
```

```
curl -s http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/routes -H 'X-API-Key: edd1c9f034335f136f87ad84b625c8f1'|jq
```

```
curl -s http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/plugins/list -H 'X-API-Key: edd1c9f034335f136f87ad84b625c8f1'|jq
```

## 5、gzip

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

```
{
  "_meta": {
    "disable": false
  },
  "buffers": {
    "number": 32,
    "size": 4096
  },
  "comp_level": 5,
  "http_version": 1.1,
  "min_length": 20,
  "types": "*",
  "vary": true
}
```

> enable plugin

```
curl -i http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/global_rules/gzip  \
-H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "plugins": {
        "gzip": {
           "_meta": {
            "disable": false
          },
          "comp_level": 5,
          "http_version": 1.1,
          "min_length": 20,
          "types": "*",
          "vary": true,
          "buffers": {
          	 "number": 32,
             "size": 4096
          }
        }
    }
}'
```

> 查看

```
curl -s http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/global_rules/gzip -H 'X-API-Key: edd1c9f034335f136f87ad84b625c8f1'|jq
```

> delete plugin

```
curl http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/global_rules/gzip -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X DELETE
```

> 验证

> Accept-Encoding:  gzip, deflate
> Content-Encoding: gzip

```
curl apisix.ingress.org:30962/index.html -i -H "Accept-Encoding: gzip"
```

> 根据代码中所做的更改重新加载插件

```
curl http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/plugins/reload -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT
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

nginx

```
location / {
    // 启用支持websocket连接
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

annotation

```
curl http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/routes/665bd12e  \
-H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PATCH -i -d '
{
    "enable_websocket": true
}'
```

```
curl http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/routes/f7c122fc  \
-H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PATCH -i -d '
{
    "enable_websocket": false
}'
```

```
curl -s $(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/routes/f7c122fc -H 'X-API-Key: edd1c9f034335f136f87ad84b625c8f1'|jq
```

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

## 11、Customize Nginx configuration

[https://github.com/apache/apisix/blob/master/docs/zh/latest/customize-nginx-configuration.md](https://github.com/apache/apisix/blob/master/docs/zh/latest/customize-nginx-configuration.md)

[https://github.com/apache/apisix/blob/release/3.3/conf/config-default.yaml](https://github.com/apache/apisix/blob/release/3.3/conf/config-default.yaml)

[http://nginx.org/en/docs/http/ngx_http_core_module.html#http](http://nginx.org/en/docs/http/ngx_http_core_module.html#http)

[https://nginx.org/en/docs/](https://nginx.org/en/docs/)

## 12、prometheus 插件

启用插件

```
curl -s http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/global_rules -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
   "id":"1",
   "plugins":{
      "prometheus":{
      }
   }
}'
```





```
curl -s http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/global_rules -X PUT -d '
{
    "uri": "/test/index.html",
    "plugins": {
        "redirect": {
            "uri": "https://192.168.0.127/error",
            "ret_code": 301
        }
    }
}'
```



## 13、limit-req

limit-req插件使用`漏桶算法`限制单个客户端对服务的请求速率。

[https://apisix.apache.org/zh/docs/apisix/plugins/limit-req/](https://apisix.apache.org/zh/docs/apisix/plugins/limit-req/)

插件主要参数：

* key用来做请求计数的依据
* rate请求速率（以秒为单位）
* burst支持突发请求量
* nodelay不延迟突发请求

```
curl http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b635c8f1' -X PUT -d '
{
	"uri": "/search/*",
	"plugins": {
		"limit-conn": {
			"conn": 10,
			"burst": 0,
			"default_conn_delay": 0.1,
			"rejected_code": 503,
			"key": "remote_addr"
		}
	},
	"upstream": {
		"type": "roundrobin",
		"nodes": {
			"39.97.63.215:80": 1
		}
	}
}'
```

## 14、limit-conn

limit-conn插件用于限制客户端对单个服务的并发请求数。当客户端对路由的并发请求数达到限制时，可以返回自定义的状态码和响应信息。

## 15、limit-count

limit-count插件使用`固定的窗口算法`，主要用于限制单个客户端在指定时间范围内对服务的总请求数，并且会在HTTP响应头中返回剩余可以请求的个数。

## 16、synchronizing Kubernetes resources to APISIX

```
kubectl get cm -n ingress-apisix apisix-configmap -o yaml|sed -e 's/6h/30s/g'|kubectl apply -f -
kubectl get pod -n ingress-apisix -o name|grep controller|xargs kubectl delete -n ingress-apisix
kubectl get cm -n ingress-apisix apisix-configmap -o yaml|sed -e 's/30s/6h/g'|kubectl apply -f -
```

# 七、通过 lua 实现限速 limit-rate

[https://www.daxuxu.info/blog/post/nginx-lua-jie-shao/](https://www.daxuxu.info/blog/post/nginx-lua-jie-shao/)

[https://blog.donatas.net/blog/2017/07/25/limit-bandwidth-openresty/](https://blog.donatas.net/blog/2017/07/25/limit-bandwidth-openresty/)

[https://developer.moduyun.com/article/c65f7549-169e-4544-b4d9-654bd9389bed.html](https://developer.moduyun.com/article/c65f7549-169e-4544-b4d9-654bd9389bed.html)

[https://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate](https://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate)

[https://github.com/apache/apisix/blob/master/example/apisix/plugins/3rd-party.lua](https://github.com/apache/apisix/blob/master/example/apisix/plugins/3rd-party.lua)

```
local core = require("apisix.core")
local ngx  = ngx
local type = type

local schema = {
  type = "object",
  properties = {
    limit_rate_after = {type ="string"},
    limit_rate = {type ="string"}
  },
  required = {"limit_rate_after","limit_rate"},
}

local plugin_name = "limit_rate"

local _M={
    version = 0.1,
    priority = 99,
    name = plugin_name,
    schema = schema
}

function _M.check_schema(conf)
  return core.schema.check(schema, conf)
end

function _M.access(conf,ctx)
    ngx.var.limit_rate_after = conf.limit_rate_after
    ngx.var.limit_rate = conf.limit_rate
    --return 203, conf.limit_rate_after
end

function _M.header_filter(ctx)
    core.response.add_header("X-Custom-Header", "hlyani")
end

--function _M.body_filter(ctx)
--    core.log.warn("hit body_filter phase")
--end

function _M.log(conf, ctx)
    core.log.warn("limit_rate_after: ", conf.limit_rate_after, ", limit_rate: ", conf.limit_rate)
end

-- 注册插件
return _M
```

```
kubectl cp limit-rate.lua -n ingress-apisix apisix-649cb68c96-4d8cr:/usr/local/apisix/apisix/plugins/ai.lua
```

```
kubectl exec -it -n ingress-apisix apisix-649cb68c96-4d8cr apisix reload
```

```
curl -s http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/global_rules -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
   "id":"1",
   "plugins":{
      "limit-rate":{
        "limit_rate": "2k",
        "limit_rate_after": "500k"
      }
   }
}'
```

```
curl -s http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/global_rules -H 'X-API-Key: edd1c9f034335f136f87ad84b625c8f1'|jq
```

```
curl -s http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/plugins/list -H 'X-API-Key: edd1c9f034335f136f87ad84b625c8f1'|jq
```

```
curl -s http://$(kubectl get svc -n ingress-apisix apisix-admin -o jsonpath="{.spec.clusterIP}"):9180/apisix/admin/plugins/list -H 'X-API-Key: edd1c9f034335f136f87ad84b625c8f1'|jq|grep ai
```

```
kubectl edit cm -n ingress-apisix  apisix
     
      http_server_location_configuration_snippet:      |
        set $limit_rate 0;
        set $limit_rate_after 0;
```

```
curl apisix.ingress.org:32060/headers -v
```

```
curl -o /dev/null  apisix.ingress.org:32060/image/jpeg
```

```
apiVersion: apisix.apache.org/v2
kind: ApisixPluginConfig
metadata:
  name: limit-rate
spec:
  plugins:
    - name: limit-rate
      enable: true
      config:
        limit_rate: 20k
        limit_rate_after: 500k
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-httpbin-ingress
  annotations:
    k8s.apisix.apache.org/plugin-config-name: limit-rate
spec:
  ingressClassName: nginx
  rules:
  - host: apisix.ingress.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: test-httpbin-svc
            port:
              number: 80
```

```
apiVersion: apisix.apache.org/v2
kind: ApisixRoute
metadata:
  name: test-limit-rate
spec:
  http:
    - name: route
      match:
        hosts:
          - apisix.ingress.org
        paths:
          - /
      backends:
        - serviceName: test-httpbin-svc
          servicePort: 80
      plugins:
        - name: limit-rate
          enable: true
          config:
            limit_rate: 20k
            limit_rate_after: 500k
```

## 修改 chart 包

```
vim apisix/templates/limit-rate.yaml

{{- if .Values.apisix.customPlugins.enabled }}
kind: ConfigMap
apiVersion: v1
metadata:
  name: limit-rate
data:
  limit-rate.lua: |
    local core = require("apisix.core")
    local ngx  = ngx
    local type = type

    local schema = {
      type = "object",
      properties = {
        limit_rate_after = {type ="string"},
        limit_rate = {type ="string"}
      },
      required = {"limit_rate_after","limit_rate"},
    }

    local plugin_name = "limit_rate"

    local _M={
        version = 0.1,
        priority = 1004,
        name = plugin_name,
        schema = schema
    }

    function _M.check_schema(conf)
      return core.schema.check(schema, conf)
    end

    function _M.access(conf,ctx)
        ngx.var.limit_rate_after = conf.limit_rate_after
        ngx.var.limit_rate = conf.limit_rate
    end

    function _M.log(conf, ctx)
        core.log.warn("limit_rate_after: ", conf.limit_rate_after, ", limit_rate: ", conf.limit_rate)
    end

    return _M
{{- end }}
```

> luaPath: 因为已经放在了默认插件目录，所以可以不需要配置
>
> luaPath 的路径会默认添加 “/apisix/plugins”

> chart comfigmap 逻辑  如果 plugins 不为空，会自动加载 .Values.apisix.customPlugins.plugins，所以 apisix.plugins 不用添加新加插件

```
vim apisix/values.yaml

apisix:
  plugins:
    - 略。。。
  customPlugins:
    enabled: true
    #luaPath: "/usr/local/apisix/?.lua" # ->  /usr/local/apisix/apisix/plugins/?.lua
    luaPath: "/opt/custom_plugins/?.lua"
    plugins:
      - name: "limit-rate"
        attrs: {}
        configMap
          name: "limit-rate"
          mounts:
            - key: "limit-rate.lua"
              path: "/usr/local/apisix/apisix/plugins/limit-rate.lua"
```

# 八、wrk 压力测试

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

# 九、FAQ

[https://apisix.apache.org/zh/docs/apisix/FAQ/](https://apisix.apache.org/zh/docs/apisix/FAQ/)

# 十、Nginx参数优化

[https://gist.github.com/denji/8359866](https://gist.github.com/denji/8359866)

[https://www.cloudpanel.io/blog/nginx-performance/](https://www.cloudpanel.io/blog/nginx-performance/)

[https://nginx.org/en/docs/](https://nginx.org/en/docs/)

```
events {
        use epoll;
        worker_connections 6000;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    access_log /usr/local/nginx/logs/access.log;
    error_log  /usr/local/nginx/logs/error.log;
    proxy_connect_timeout    900;
    proxy_read_timeout       900;
    proxy_send_timeout       900;
    proxy_cache_path /var/cache/nginx keys_zone=a_cache:10m inactive=10m max_size=10g;
    gzip on;
    gzip_static on;
    gzip_buffers 40 4K;
    gzip_comp_level 7;
    gzip_min_length 1k;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png application/vnd.ms-fontobject font/ttf font/opentype font/x-woff image/svg+xml;
    gzip_disable "MSIE [1-6]\.";
    gzip_vary on;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 500M;
}

 location  / {
    proxy_http_version 1.1;
    proxy_set_header Host $host:$server_port;
    proxy_set_header X-Referer $http_referer;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header Origin "";
    client_body_buffer_size 128k;
    client_max_body_size 500M;
    proxy_connect_timeout 90;
    proxy_send_timeout 90;
    proxy_read_timeout 90;
    proxy_buffer_size 4k;
    proxy_buffers 32 4k;
    proxy_busy_buffers_size 64k;
}
```

```
{
    "client-max-body-size": "500m",
    "default-server-return": "https://192.168.0.127/error",
    "keepalive-requests": "3000",
    "keepalive-timeout": "65s",
    "location-snippets": "proxy_set_header Origin \"\";\ngzip_static on;\nproxy_set_header Accept-Encoding gzip;\n",
    "proxy-connect-timeout": "900s",
    "proxy-read-timeout": "900s",
    "proxy_send_timeout": "900s",
    "worker-connections": "3000"
}
```

```
  nginx:
    workerRlimitNofile: "204800"
    workerConnections: "204800"
    workerProcesses: auto
    enableCPUAffinity: true
    keepaliveTimeout: 30
    clientMaxBodySize: 500M
    logs:
      enableAccessLog: false
    configurationSnippet:
      httpStart: |
        access_log off;
        open_file_cache_valid 30s;
        open_file_cache_min_uses 2;
        open_file_cache_errors on;
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        gzip on;
        gzip_static on;
        gzip_min_length 10240;
        gzip_comp_level 5;
        gzip_vary on;
        gzip_disable msie6;
        #gzip_proxied expired no-cache no-store private auth;
        gzip_proxied any;
        gzip_types
          text/css
          text/javascript
          text/xml
          text/plain
          text/x-component
          application/javascript
          application/x-javascript
          application/json
          application/xml
          application/rss+xml
          application/atom+xml
          application/vnd.ms-fontobject
          font/truetype
          font/opentype
          image/svg+xml;
        reset_timedout_connection on;
        keepalive_requests 100000;
        client_body_buffer_size 128k;
        client_header_buffer_size 3m;
      httpSrvLocation: |
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
```

final

```
  config.yaml: |-
    #
    # Licensed to the Apache Software Foundation (ASF) under one or more
    # contributor license agreements.  See the NOTICE file distributed with
    # this work for additional information regarding copyright ownership.
    # The ASF licenses this file to You under the Apache License, Version 2.0
    # (the "License"); you may not use this file except in compliance with
    # the License.  You may obtain a copy of the License at
    #
    #     http://www.apache.org/licenses/LICENSE-2.0
    #
    # Unless required by applicable law or agreed to in writing, software
    # distributed under the License is distributed on an "AS IS" BASIS,
    # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    # See the License for the specific language governing permissions and
    # limitations under the License.
    #
    apisix:    # universal configurations
      events:                             # Event distribution module configuration
        module: lua-resty-events          # Sets the name of the events module used.
                                          # Supported module: lua-resty-worker-events and lua-resty-events
      node_listen:    # APISIX listening port
        - 9080
      enable_heartbeat: true
      enable_admin: true
      enable_admin_cors: true
      enable_debug: false
      extra_lua_path: /opts/custom_plugins/?.lua;

      enable_control: true
      control:
        ip: 127.0.0.1
        port: 9090

      enable_dev_mode: false                       # Sets nginx worker_processes to 1 if set to true
      enable_reuseport: true                       # Enable nginx SO_REUSEPORT switch if set to true.
      enable_ipv6: true # Enable nginx IPv6 resolver
      enable_http2: true
      enable_server_tokens: true # Whether the APISIX version number should be shown in Server header

      # proxy_protocol:                   # Proxy Protocol configuration
      #   listen_http_port: 9181          # The port with proxy protocol for http, it differs from node_listen and admin_listen.
      #                                   # This port can only receive http request with proxy protocol, but node_listen & admin_listen
      #                                   # can only receive http request. If you enable proxy protocol, you must use this port to
      #                                   # receive http request with proxy protocol
      #   listen_https_port: 9182         # The port with proxy protocol for https
      #   enable_tcp_pp: true             # Enable the proxy protocol for tcp proxy, it works for stream_proxy.tcp option
      #   enable_tcp_pp_to_upstream: true # Enables the proxy protocol to the upstream server

      proxy_cache:                         # Proxy Caching configuration
        cache_ttl: 10s                     # The default caching time if the upstream does not specify the cache time
        zones:                             # The parameters of a cache
        - name: disk_cache_one             # The name of the cache, administrator can be specify
                                           # which cache to use by name in the admin api
          memory_size: 50m                 # The size of shared memory, it's used to store the cache index
          disk_size: 1G                    # The size of disk, it's used to store the cache data
          disk_path: "/tmp/disk_cache_one" # The path to store the cache data
          cache_levels: "1:2"              # The hierarchy levels of a cache
      #  - name: disk_cache_two
      #    memory_size: 50m
      #    disk_size: 1G
      #    disk_path: "/tmp/disk_cache_two"
      #    cache_levels: "1:2"

      router:
        http: radixtree_host_uri  # radixtree_uri: match route by uri(base on radixtree)
                                    # radixtree_host_uri: match route by host + uri(base on radixtree)
                                    # radixtree_uri_with_parameter: match route by uri with parameters
        ssl: 'radixtree_sni'        # radixtree_sni: match route by SNI(base on radixtree)

      proxy_mode: http
      stream_proxy:                 # TCP/UDP proxy
        tcp:                        # TCP proxy port list
          - 9100
        udp:                        # UDP proxy port list
          - 9200
      # dns_resolver:
      #
      #   - 127.0.0.1
      #
      #   - 172.20.0.10
      #
      #   - 114.114.114.114
      #
      #   - 223.5.5.5
      #
      #   - 1.1.1.1
      #
      #   - 8.8.8.8
      #
      dns_resolver_valid: 30
      resolver_timeout: 5
      ssl:
        enable: false
        listen:
          - port: 9443
            enable_http3: false
        ssl_protocols: "TLSv1.2 TLSv1.3"
        ssl_ciphers: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA256:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA"

    nginx_config:    # config for render the template to genarate nginx.conf
      error_log: "/dev/stderr"
      error_log_level: "warn"    # warn,error
      worker_processes: "10"
      enable_cpu_affinity: true
      worker_rlimit_nofile: 8192  # the number of files a worker process can open, should be larger than worker_connections
      worker_shutdown_timeout: 240s
      max_pending_timers: 16384
      max_running_timers: 4096
      event:
        worker_connections: 1024
      http:
        lua_shared_dict: # Nginx Lua shared memory zone. Size units are m or k.
          lrucache-lock: 100m
          prometheus-metrics: 200m
          worker-events: 100m
        enable_access_log: true
        access_log: "/dev/stdout"
        access_log_format: '$remote_addr - $remote_user [$time_local] $http_host \"$request\" $status $body_bytes_sent $request_time \"$http_referer\" \"$http_user_agent\" $upstream_addr $upstream_status $upstream_response_time \"$upstream_scheme://$upstream_host$upstream_uri\"'
        access_log_format_escape: default
        keepalive_timeout: "60s"
        client_max_body_size: 500M
        client_header_timeout: 60s     # timeout for reading client request header, then 408 (Request Time-out) error is returned to the client
        client_body_timeout: 60s       # timeout for reading client request body, then 408 (Request Time-out) error is returned to the client
        send_timeout: 30s              # timeout for transmitting a response to the client.then the connection is closed
        underscores_in_headers: "on"   # default enables the use of underscores in client request header fields
        real_ip_header: "X-Real-IP"    # http://nginx.org/en/docs/http/ngx_http_realip_module.html#real_ip_header
        real_ip_from:                  # http://nginx.org/en/docs/http/ngx_http_realip_module.html#set_real_ip_from
          - 127.0.0.1
          - 'unix:'
        upstream:
          keepalive: 320
          keepalive_requests: 1000
          keepalive_timeout: 60s
      http_configuration_snippet:      |
        access_log on;
        proxy_connect_timeout 900;
        proxy_read_timeout 900;
        proxy_send_timeout 900;
        proxy_buffers 16 32k;
        proxy_buffer_size 64k;
        open_file_cache_valid 30s;
        open_file_cache_min_uses 2;
        open_file_cache_errors on;
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        gzip on;
        gzip_static on;
        gzip_min_length 1k;
        gzip_buffers 16 8k;
        gzip_comp_level 7;
        gzip_vary on;
        gzip_disable msie6;
        gzip_proxied any;
        gzip_types
          text/css
          text/javascript
          text/xml
          text/plain
          text/x-component
          application/javascript
          pplication/x-javascript
          application/json
          application/xml
          application/rss+xml
          application/atom+xml
          application/vnd.ms-fontobject
          font/truetype
          font/opentype
          image/svg+xml;
        reset_timedout_connection on;
        keepalive_requests 1000;
        client_body_buffer_size 128k;
        client_header_buffer_size 3m;
      http_server_location_configuration_snippet:      |
        proxy_connect_timeout 900;
        proxy_read_timeout 900;
        proxy_send_timeout 900;
        proxy_buffers 16 32k;
        proxy_buffer_size 64k;
        set $limit_rate 0;
        set $limit_rate_after 0;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_intercept_errors on;
        error_page 400 401 402 403 404 405 406 407 408 409 410 411 412 413 414 415 416 417 418 421 422 423 424 425 426 428 429 431 451 500 501 502 503 504 505 506 507 508 510 511 = https://10.15.200.50:58043/notebook/error;
    plugins:    # plugin list
      - cors
      - proxy-rewrite
      - limit-rate
      - limit-conn
      - limit-count
      - limit-req
      - gzip
      - redirect
      - response-rewrite
      - prometheus
      - limit-rate
    plugin_attr:
      prometheus:
        export_addr:
          ip: 0.0.0.0
          port: 9091
        export_uri: /apisix/prometheus/metrics
        metric_prefix: apisix_

    deployment:
      role: traditional
      role_traditional:
        config_provider: etcd
      admin:
        allow_admin:    # http://nginx.org/en/docs/http/ngx_http_access_module.html#allow
          - 127.0.0.1/24
          - 0.0.0.0/0
        #   - "::/64"
        admin_listen:
          ip: 0.0.0.0
          port: 9180
        # Default token when use API to call for Admin API.
        # *NOTE*: Highly recommended to modify this value to protect APISIX's Admin API.
        # Disabling this configuration item means that the Admin API does not
        # require any authentication.
        admin_key:
          # admin: can everything for configuration data
          - name: "admin"
            key: edd1c9f034335f136f87ad84b625c8f1
            role: admin
          # viewer: only can view configuration data
          - name: "viewer"
            key: 4054f7cf07e344346cd3f287985e76a2
            role: viewer
      etcd:
        host:                          # it's possible to define multiple etcd hosts addresses of the same etcd cluster.
          - "http://192.168.0.127:2379"             # multiple etcd address
        timeout: 30
        watch_timeout: 50
        resync_delay: 5
        health_check_timeout: 10
        startup_retry: 2
        prefix: "/apisix"    # configuration prefix in etcd
        timeout: 30    # 30 seconds
```

# 十一、default error return

```
      httpSrv: |
        location = /default-server-return {
           proxy_pass https://192.168.0.127/error;
        }

        location /error-forward/ {
          proxy_pass https://10.15.200.50:58043;
        }
        
      httpSrvLocation: |
        proxy_intercept_errors on;
        error_page 400 401 402 403 404 405 406 407 408 409 410 411 412 413 414 415 416 417 418 421 422 423 424 425 426 428 429 431 451 500 501 502 503 504 505 506 507 508 510 511 /default-server-return;

```

[https://github.com/apache/apisix/issues/7987](https://github.com/apache/apisix/issues/7987)

```
  configurationSnippet:
    # Based on
    # - https://blog.adriaan.io/one-nginx-error-page-to-rule-them-all.html
    # - https://gist.github.com/lextoumbourou/d6221deb818da4f342ea
    httpStart: |
      more_clear_headers Server;
      
      map $status $status_text {
        400 'Bad Request';
        401 'Unauthorized';
        402 'Payment Required';
        403 'Forbidden';
        404 'Not Found';
        405 'Method Not Allowed';
        406 'Not Acceptable';
        407 'Proxy Authentication Required';
        408 'Request Timeout';
        409 'Conflict';
        410 'Gone';
        411 'Length Required';
        412 'Precondition Failed';
        413 'Payload Too Large';
        414 'URI Too Long';
        415 'Unsupported Media Type';
        416 'Range Not Satisfiable';
        417 'Expectation Failed';
        418 'I\'m a teapot';
        421 'Misdirected Request';
        422 'Unprocessable Entity';
        423 'Locked';
        424 'Failed Dependency';
        425 'Too Early';
        426 'Upgrade Required';
        428 'Precondition Required';
        429 'Too Many Requests';
        431 'Request Header Fields Too Large';
        451 'Unavailable For Legal Reasons';
        500 'Internal Server Error';
        501 'Not Implemented';
        502 'Bad Gateway';
        503 'Service Unavailable';
        504 'Gateway Timeout';
        505 'HTTP Version Not Supported';
        506 'Variant Also Negotiates';
        507 'Insufficient Storage';
        508 'Loop Detected';
        510 'Not Extended';
        511 'Network Authentication Required';
        default 'Something is wrong';
      }
      map $http_accept $extension {
        default html;
        ~*application/json json;
      }
    httpSrv: |
      error_page 400 401 402 403 404 405 406 407 408 409 410 411 412 413 414 415 416 417 418 421 422 423 424 425 426 428 429 431 451 500 501 502 503 504 505 506 507 508 510 511 @error_$extension;

      location @error_json {
        types { } default_type "application/json; charset=utf-8";
        echo '{"error_msg": "$status_text"}';
      }
      location @error_html {
        types { } default_type "text/html; charset=utf-8";
        echo '<html><head><title>$status $status_text</title></head><body><center><h1>$status $status_text</h1></center></html>';
      }
```

```
      httpSrvLocation: |
        proxy_intercept_errors on;
        error_page 400 401 402 403 404 405 406 407 408 409 410 411 412 413 414 415 416 417 418 421 422 423 424 425 426 428 429 431 451 500 501 502 503 504 505 506 507 508 510 511 = https://192.168.0.127/error;
```

