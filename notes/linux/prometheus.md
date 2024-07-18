# Prometheus

# 一、PromQL

## 1、选择器

* =    与字符串匹配
* !=   与字符串不匹配
* =~  与字符串正则匹配
* !~   与字符串正则不匹配

```
http_requests_total{handler=~"/api/v1/.*"}
```

## 2、范围查询

> ms, s, m, h, d, w, y

```
http_requests_total[5m]
```

## 3、时间位移

> 查询的时间范围分别前移 5分钟

```
node_filesystem_free_bytes{mountpoint="/data00"} offset 5m
```

## 4、操作符

[https://prometheus.io/docs/prometheus/latest/querying/operators/](https://prometheus.io/docs/prometheus/latest/querying/operators/)

> +, -, *, /, %, ^

> ==, !=, >, <, >=, <=

> and, or, unless

> on, ignoring

> roup_left, roup_right

## 5、聚合操作

|     函数     |                  说明                  |
| :----------: | :------------------------------------: |
|     sum      |                  求和                  |
|    count     |                  计数                  |
| count_values |              对value计数               |
|     min      |                 最小值                 |
|     max      |                 最大值                 |
|     avg      |                 平均值                 |
|    stddev    |                 标准差                 |
|    stdvar    |                标准方差                |
|   bottomk    |               后n条时序                |
|     topk     |               前n条时序                |
|   quantile   |                 分位数                 |
|      @       | 更改查询中各个即时和范围向量的计算时间 |
|    atan2     |                弧度计算                |

> without用于从计算结果中移除列举的标签，而保留其它标签。
>
> by则正好相反，结果向量中只保留列出的标签，其余标签则移除。

```
sum(http_requests_total) without (instance)
等价于
sum(http_requests_total) by (code,handler,job,method)
```

```
# 获取HTTP请求数前5位的时序样本数据
topk(5, http_requests_total)
```

```
# quantile用于计算当前样本数据值的分布情况quantile(φ, express)其中0 ≤ φ ≤ 1。
例如，当φ为0.5时，即表示找到当前样本数据中的中位数：
quantile(0.5, http_requests_total)
```

```
count by (namespace) (kube_pod_container_resource_limits: {ressource="cpu"})
sum by (node) (node_memory_MemTotal_bytes — node_memory_MemAvailable_bytes)
```

```
sum(http_requests_total{method="GET"} @ 1609746000) 
```

```
rate(http_requests_total[5m])[30m:1m]
```

```
sum(apisix_http_requests_total)[1h:]
```

## 6、常用方法

[https://prometheus.io/docs/prometheus/latest/querying/functions/](https://prometheus.io/docs/prometheus/latest/querying/functions/)

### 1.increase

> 获取区间向量中第一个样本和最后一个样本，并返回其增长量。

获取节点存储5分钟内的变化量

```
increase(node_filesystem_free_bytes{mountpoint="/data"}[5m])
```

### 2.rate、irate

> rate 计算区间向量在时间窗口内平均增长速率，会在单调性发生变化时自动中断。
>
> irate 计算区间向量的增长率，但其反应出的是瞬时增长率。
>
> rate 与 irate 函数仅适用于 Counter 类型的 Metrics。

获取 HTTP Request 请求5 分钟内的变化率。

```
rate(http_request_total{status="200", method="GET"}[5m])

irate(http_request_total{status="200", method="GET"}[5m])
```

### 3.delta、idelta

> delta 计算一个区间向量的第一个元素和最后一个元素之间的差值。
>
> idelta与delta函数类似，不同的是它计算最新的2个样本之间的差值。

获取最近两个小时 CPU 的温度差值。

```
delta(cpu_temp_celsius{host="zeus"}[2h])
```

### 4.histogram_quantile

> 计算分位数

计算延迟的 P99 值。

```
histogram_quantile(0.99 , rate(prometheus_tsdb_compaction_chunk_range_bucket[5m]))
```

### 5.absent

> 一般用于验证 样本是否存在，如果存在则返回空，如果不存在，则返回 1。

确认节点中 node exporter 是否存在。

```
absent(up{job="node-exporter", instance="127.0.0.1:9100"})
```

### 6.abs、ceil、floor

> abs() 绝对值
>
> ceil() 向上取整
>
> floor() 向下取整

### 7.label_replace

> 动态标签替换

```
label_replace(node_boot_time_seconds{instance="10.13.1.10:9100"},"node","$1","instance","(.*):9100")
```

```
label_replace(up, "host", "$1", "instance",  "(.*):.*")
```

```
label_replace(apisix_http_status,"path","$0","matched_uri",".*")
```

### 8.label_join

```
label_join(up{job="api-server",src1="a",src2="b",src3="c"}, "foo", ",", "src1", "src2", "src3")
```

### 9.time

> 当前时间戳

```
(time()-node_boot_time_seconds)/60/60/24
```

### 10.group_right/group_left

```
a * on (foo, bar) b
```

```
a * ignoring (baz) b
```

```
a * on (foo, bar) group_left(baz) b
```

```
kube_node_info * on (node) group_right() kube_node_status_condition{condition="Ready",status="true"} * on (node) group_right() label_replace(node_boot_time_seconds,"node","$1","instance","(.*):9100")
```

```
(sum(label_replace(label_replace(apisix_http_status,"host","$0","matched_host",".*"),"path","$0","matched_uri",".*")) by (code,path,host)) * on (host,path) group_left(service_name,ingress,namespace,service_port) (kube_ingress_path)
```

```
(sum(label_replace(label_replace(apisix_http_status,"host","$0","matched_host",".*"),"path","$1","matched_uri","(/[^/.]*).*")) by (code,path,host)) * on (host,path) group_left(service_name,ingress,namespace,service_port) (kube_ingress_path)
```

# 二、HTTP API

[https://prometheus.io/docs/prometheus/latest/querying/api/](https://prometheus.io/docs/prometheus/latest/querying/api/)

```
curl -G 'http://192.168.0.127:32070/api/v1/query_range' \
--data-urlencode 'query=sum(apisix_http_requests_total)' \
--data-urlencode 'start=2024-07-15T20:10:30.781Z' \
--data-urlencode 'end=2024-07-15T20:10:30.781Z' \
--data-urlencode 'step=15s'
```

# 三、自定义metric

```
go get github.com/prometheus/client_golang/prometheus
go get github.com/prometheus/client_golang/prometheus/promhttp
```

```
package main
 
import (
    "net/http"
 
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)
 
// 定义计数器
var requestTotal = prometheus.NewCounterVec(
    prometheus.CounterOpts{
        Name: "http_requests_total",
        Help: "Number of get requests.",
    },
    []string{"code"},
)
 
func main() {
    // 注册指标
    prometheus.MustRegister(requestTotal)
 
    // 使用http.HandleFunc来处理metrics请求
    http.Handle("/metrics", promhttp.Handler())
 
    // 你的业务逻辑
    // ...
 
    // 启动HTTP服务器
    http.ListenAndServe(":8080", nil)
}
```

```
vim prometheus-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: 'my-custom-metrics'
        static_configs:
          - targets: ['localhost:8080']
```

# 四、kube-state-metrics

[https://github.com/kubernetes/kube-state-metrics/blob/main/docs/metrics/cluster/node-metrics.md](https://github.com/kubernetes/kube-state-metrics/blob/main/docs/metrics/cluster/node-metrics.md)

## 1、允许labels

```
vim kubeasz/roles/kube-prometheus-stack/files/kube-prometheus-stack/charts/kube-state-metrics/values.yaml
...
extraArgs: ["--metric-labels-allowlist=nodes=[*]", "--metric-annotations-allowlist=nodes=[*]"]
...
```

```
--metric-labels-allowlist=pods=[*]
```

```
--metric-labels-allowlist=nodes=[*],pods=[*],persistentvolumeclaims=[*],deployments=[*],statefulsets=[*],configmaps=[*],secrets=[*],services=[*],replicasets=[*]
```

```
--metric-labels-allowlist=*=[*]
```

```
--metric-annotations-allowlist=pods=[*]
```

```
--metric-labels-allowlist=pods=[*],nodes=[node,failure-domain.beta.kubernetes.io/zone,topology.kubernetes.io/zone]
```

## 2、查询指标

```
curl 'http://127.0.0.1:9090/metrics'
```

```
curl -G 'http://127.0.0.1:9090/api/v1/query' \
--data-urlencode 'query=count(kube_node_status_condition{condition="Ready", status="false"})/count(kubelet_node_name)*100'
```

## 3、其他

```
--metric-denylist=kube_deployment_spec_.* 
```

