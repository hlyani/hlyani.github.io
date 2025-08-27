# K8S HPA

# 一、概述

K8S HPA（HorizontalPodAutoscaler），主要适用于 Deployment 或 StatefulSet 对象。

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

## 1、扩缩比例算法

```
期望副本数 = ceil[当前副本数 * (当前指标 / 期望指标)]
```

```
currentUtilization = metricsTotal * 100 / requestedTotal

usageRatio = currentUtilization / targetUtilization

if (1 - tolerance) <= usageRatio <= (1 + tolerance): keep current replica amount

newReplicas = ceil(currentReplicas * usageRatio)
```

```
当前指标 = sum(所有 Pod 当前指标值) / Pod 数  # 当前指标或平均指标
```

## 2、计算过程示例

如果当前指标值为 `200m`，而期望值为 `100m`，则副本数将加倍， 因为 `200.0 / 100.0 == 2.0` 如果当前值为 `50m`，则副本数将减半， 因为 `50.0 / 100.0 == 0.5`。如果比率足够接近 1.0（在全局可配置的容差范围内，默认为 0.1）， 则控制平面会跳过扩缩操作。

## 3、排除计算 Pod

排除以下异常 Pod，其他所有 Pod 参与指标计算：

1. 非就绪状态的 Pod
2. 正在被终止的 Pod
3. 尚未达到指标采集时间的 Pod
4. 未暴露指标的 Pod
5. 未满足健康检查的 Pod
6. 超过指标失效时间的 Pod，可能是指标采集延迟或网络问题导致的。

## 4、资源指标

 资源指标是指 Kubernetes 内置的 CPU 和内存指标。

 支持的目标类型：

- `Utilization`：按百分比表示的指标利用率（默认目标为 Pod 请求的百分比）。
- `Value`：指标的绝对值（例如，500m 表示 0.5 CPU 核心）。
- `AverageValue`：所有 Pod 指标的平均值（例如，每个 Pod 的平均内存使用量）。

## 5、多指标联合策略

[多指标联合策略示例](https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics)

可以同时监控多个指标，并根据定义的规则来扩展Pod数量。例如，基于CPU和内存同时进行扩展。

HPA 支持定义多个指标，并根据所有指标计算出最终的扩缩容决策。

**联合规则**：

- 如果多个指标有不同的扩缩容建议值
  - **扩容**时会选择 **Pod 数量最多的值**。这是因为扩容是为了确保工作负载不会超载并保持服务的可用性。max
  - **缩容**时会选择 **Pod 数量最少的值**。但必须满足 **`minReplicas`** 限制，确保副本数不会小于最小值。这是为了避免 Pod 数量过少，导致服务不稳定。min
  - 为保证服务的可用性，**扩容优先于缩容**。

## 6、kube-controller 中配置

```YAML
spec:
  containers:
  - command:
    - kube-controller-manager
    - --horizontal-pod-autoscaler-sync-period=15
      # HPA 的同步间隔时间（单位：秒）。
      # 表示 HPA 控制器多长时间检查一次指标并决定是否调整 Pod 数量。
      # 默认值为 15 秒，这里显式设置为 15。
    - --horizontal-pod-autoscaler-tolerance=0.1
      # HPA 的容差范围（默认值：0.1，即 10%）。
      # 当当前指标值与目标值的比率在 (1 ± 容差范围) 内（如 0.9 到 1.1）时，HPA 不会触发扩缩容。
      # 只有超出此范围时才会触发扩缩容操作。
    - --horizontal-pod-autoscaler-initial-readiness-delay=30
      # 初始化延迟时间（单位：秒）。
      # 在 Pod 创建时，HPA 会等待这么长时间后才检查 Pod 的就绪状态。
      # 避免刚启动的 Pod 被误判为负载不足或负载过高。
      # 默认值为 30 秒。
    - --horizontal-pod-autoscaler-cpu-initialization-period=300
      # CPU 初始化周期（单位：秒）。
      # 在 Pod 创建后的指定时间内，HPA 会忽略 CPU 使用率指标，以防止初始化期间的波动影响扩缩容决策。
      # 默认值为 300 秒（5 分钟）。
    - --horizontal-pod-autoscaler-downscale-stabilization=300
      # 缩容稳定化时间（单位：秒）。
      # 表示在决定缩容之前，HPA 会等待指定时间，以确保指标值持续下降而不是短期波动。
      # 默认值为 300 秒（5 分钟），避免频繁缩容导致的不稳定。
```

# 三、keda

https://keda.sh/docs/2.16/

## 1、示例

```
kubectl apply -f -<<EOF
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: prometheus-scaledobject
  namespace: hpa
spec:
  scaleTargetRef: # 必选，通常是 Deployment、StatefulSet、ReplicaSet、Job
    name: hpa-test
  pollingInterval: 30  # 每隔 30 秒检查一次触发条件，默认 30s，可设置范围为 1-3600 秒
  cooldownPeriod: 300  # 扩缩容后冷却时间，单位为秒，默认 300s，可设置范围为 1-3600 秒
  minReplicaCount: 1   # 最小副本数
  maxReplicaCount: 10  # 最大副本数
  idleReplicaCount: 1 # 在无负载情况下缩容至的副本数, 默认未启用
  triggers: # 定义多个触发器来控制扩缩容逻辑
    - type: cpu
      metadata:
        value: "60" # CPU 使用率阈值，单位百分比
    - type: prometheus
      metadata:
        # 必选
        serverAddress: http://prometheus.monitoring.svc.cluster.local:9090 # Prometheus 服务器的地址。
        query: sum(rate(nginx_ingress_controller_requests[1m])) # Prometheus 查询表达式，用于获取扩缩容指标值。
        threshold: "50" # 触发扩容的指标阈值。
        # 可选
        activationThreshold: "10" # 缩容触发的指标下限（避免缩容到最小副本）。
        metricName: custom_requests_metric # 自定义指标名称。默认会使用 Prometheus 查询返回的名称。
        aggregationType: "Sum" # 指定聚合类型（如 Average、Sum、Max），默认是 Average。
        queryOffset: "1m" # 设置 Prometheus 查询偏移时间。
        mode: "Value" # 触发模式，支持 Value（直接数值对比）或 Rate（增量变化率）。
        scalingBehavior: "ScaleUpOnly" # 配置扩容行为，如 ScaleUpOnly 或 ScaleDownOnly。
        step: "30s" # Prometheus 查询时间步长
        disableScaleToZero: "true" # 禁止缩容到 0 副本
  advanced: # 扩缩容行为高级配置
    horizontalPodAutoscalerConfig:
      behavior:
        scaleUp:
          stabilizationWindowSeconds: 30 # 扩容的稳定时间窗口（秒），避免频繁扩容。
          selectPolicy: Max # 指定扩容策略, Max: 选择最大扩容步数。Min: 选择最小扩容步数。Disabled: 禁用扩容。
          policies: # 定义扩容策略及其步数规则。
            - type: Percent # 扩容步数百分比
              value: 50 # 每次扩容最多增加 50% 的副本数
              periodSeconds: 60 # 检查周期（秒）
        scaleDown:
          stabilizationWindowSeconds: 60 # 缩容的稳定时间窗口（秒），避免频繁缩容。
          selectPolicy: Min # 缩容策略选择
          policies: # 定义扩容策略及其步数规则。
            - type: Pods # 按副本数量缩容
              value: 2 # 每次缩容最多减少 2 个副本
              periodSeconds: 60 # 检查周期（秒）
  fallback:
    failureThreshold: 3  # 失败次数阈值，连续失败 3 次触发回退策略
    replicas: 2  # 在失败情况下保持的副本数
EOF
```

## 2、部署

```
git clone -b master --depth 1 http://gitlab.hpc.sugon.com/ai/ske-chart.git
helm install -n ske keda ske-chart/keda
```

## 3、自动扩缩容

### 1.主要参数解释

```
1. 最小实例数
spec.minReplicaCount
2. 最大实例数
spec.maxReplicaCount
3. 扩容生效时长
advanced.horizontalPodAutoscalerConfig.behavior.scaleUp.stabilizationWindowSeconds
4. 缩容生效时长
advanced.horizontalPodAutoscalerConfig.behavior.scaleDown.stabilizationWindowSeconds
5. 缩容触发值
spec.triggers.type.activationThreshold
6. 扩容触发值
spec.triggers.type.threshold

特别注意： k8s 默认有10%的容差，设置扩缩容指标时序需考虑
  - --horizontal-pod-autoscaler-tolerance=0.1
      # HPA 的容差范围（默认值：0.1，即 10%）。
      # 当当前指标值与目标值的比率在 (1 ± 容差范围) 内（如 0.9 到 1.1）时，HPA 不会触发扩缩容。
      # 只有超出此范围时才会触发扩缩容操作。
```

### 2.CPU 利用率

```
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: HPA_NAME
  namespace: default
spec:
  scaleTargetRef:
    name: DEPLOY_NAME
  pollingInterval: 30
  cooldownPeriod: 300
  minReplicaCount: 1
  maxReplicaCount: 10
  advanced:
    horizontalPodAutoscalerConfig:
      behavior:
        scaleUp:
          stabilizationWindowSeconds: 30
        scaleDown:
          stabilizationWindowSeconds: 300
  triggers:
  - type: prometheus
    metricType: "Value"
    metadata:
      serverAddress: http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090
      query: |
        clamp_min(
                sum(
                  rate(container_cpu_usage_seconds_total{container!="",namespace="default",pod=~".*test.*"}[1m])
                )
              /
                sum(
                  kube_pod_container_resource_limits{container!="",namespace="default",pod=~".*test.*",resource="cpu"}
                )
            *
              100
          or
            vector(0),
          0
        )
      threshold: "77.27"
      activationThreshold: "66.666"
```

### 3.GPU 利用率

```
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: HPA_NAME
  namespace: default
spec:
  scaleTargetRef:
    name: DEPLOY_NAME
  pollingInterval: 30
  cooldownPeriod: 300
  minReplicaCount: 1
  maxReplicaCount: 10
  advanced:
    horizontalPodAutoscalerConfig:
      behavior:
        scaleUp:
          stabilizationWindowSeconds: 30
        scaleDown:
          stabilizationWindowSeconds: 300
  triggers:
  - type: prometheus
    metricType: "Value"
    metadata:
      serverAddress: http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090
      query: |
        clamp_min(
              sum(
                DCGM_FI_DEV_GPU_UTIL{exported_container!="",exported_namespace="default",exported_pod=~".*notebook-xxx.*"}
              )
            /
              sum(
                label_replace(
                  kube_pod_container_resource_limits{namespace="default",pod=~".*notebook-xxxx.*",resource="nvidia_com_gpu"},
                  "exported_pod",
                  "$1",
                  "pod",
                  "(.*)"
                )
              )
          or
            vector(0),
          0
        )
      threshold: "77.27"
      activationThreshold: "66.666"
```

### 4.QPM

```
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: HPA_NAME
  namespace: default
spec:
  scaleTargetRef:
    name: DEPLOY_NAME
  pollingInterval: 30
  cooldownPeriod: 300
  minReplicaCount: 1
  maxReplicaCount: 10
  advanced:
    horizontalPodAutoscalerConfig:
      behavior:
        scaleUp:
          stabilizationWindowSeconds: 30
        scaleDown:
          stabilizationWindowSeconds: 300
  triggers:
  - type: prometheus
    metricType: "Value"
    metadata:
      serverAddress: http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090
      query: |
          ceil(
              sum(rate(nginx_ingress_controller_requests{exported_namespace="default",host=~".*test.*"}[1m]))
            *
              60
          )
        or
          vector(0)
      threshold: "77.27"
      activationThreshold: "66.666"
```

# 四、示例

## 1、部署 deploy

```Bash
kubectl apply -f -<<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: hpa
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-test
  namespace: hpa
spec:
  selector:
    matchLabels:
      run: hpa-test
  template:
    metadata:
      labels:
        run: hpa-test
    spec:
      containers:
      - name: hpa-test
        image: hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: hpa-test
  namespace: hpa
  labels:
    run: hpa-test
spec:
  ports:
  - port: 80
  selector:
    run: hpa-test
EOF
```

## 2、配置 hpa

```Bash
kubectl autoscale deployment -n hpa hpa-test --cpu-percent=50 --min=1 --max=10
```

或

```Bash
kubectl apply -f -<<EOF
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-test
  namespace: hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-test
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
EOF
```

## 3、查看 hpa

```Bash
kubectl get hpa -n hpa
NAME       REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-test   Deployment/hpa-test   0%/50%    1         10        1          18s
```

## 4、增加负载

```Bash
kubectl run -it load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://hpa-test.hpa.svc.cluster.local; done"
```

## 5、查看 hpa

```Bash
kubectl get hpa -n hpa hpa-test --watch

kubectl get hpa -n hpa hpa-test -w
NAME       REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-test   Deployment/hpa-test   0%/50%    1         10        1          10m
hpa-test   Deployment/hpa-test   92%/50%   1         10        1          10m
hpa-test   Deployment/hpa-test   249%/50%   1         10        2          11m
hpa-test   Deployment/hpa-test   250%/50%   1         10        4          11m
hpa-test   Deployment/hpa-test   250%/50%   1         10        5          11m
hpa-test   Deployment/hpa-test   166%/50%   1         10        5          11m
hpa-test   Deployment/hpa-test   69%/50%    1         10        5          12m
hpa-test   Deployment/hpa-test   56%/50%    1         10        5          12m
hpa-test   Deployment/hpa-test   58%/50%    1         10        6          12m
hpa-test   Deployment/hpa-test   53%/50%    1         10        6          12m
hpa-test   Deployment/hpa-test   47%/50%    1         10        6          13m
hpa-test   Deployment/hpa-test   47%/50%    1         10        6          13m

kubectl get po -n hpa -w
NAME                        READY   STATUS              RESTARTS   AGE
hpa-test-6c7675c585-28dsd   0/1     ContainerCreating   0          7s
hpa-test-6c7675c585-k8hx7   1/1     Running             0          15m
hpa-test-6c7675c585-rqtqj   0/1     ContainerCreating   0          22s
hpa-test-6c7675c585-vkf7h   0/1     ContainerCreating   0          7s
hpa-test-6c7675c585-lzgb2   0/1     Pending             0          0s
hpa-test-6c7675c585-lzgb2   0/1     Pending             0          0s
hpa-test-6c7675c585-lzgb2   0/1     ContainerCreating   0          0s
hpa-test-6c7675c585-rqtqj   1/1     Running             0          36s
hpa-test-6c7675c585-vkf7h   1/1     Running             0          34s
hpa-test-6c7675c585-lzgb2   1/1     Running             0          25s
hpa-test-6c7675c585-28dsd   1/1     Running             0          43s
hpa-test-6c7675c585-gjc58   0/1     Pending             0          0s
hpa-test-6c7675c585-gjc58   0/1     Pending             0          0s
hpa-test-6c7675c585-gjc58   0/1     ContainerCreating   0          0s
hpa-test-6c7675c585-gjc58   1/1     Running             0          5s

kubectl get po -n hpa 
NAME                        READY   STATUS    RESTARTS   AGE
hpa-test-6c7675c585-28dsd   1/1     Running   0          2m
hpa-test-6c7675c585-gjc58   1/1     Running   0          44s
hpa-test-6c7675c585-k8hx7   1/1     Running   0          17m
hpa-test-6c7675c585-lzgb2   1/1     Running   0          105s
hpa-test-6c7675c585-rqtqj   1/1     Running   0          2m15s
hpa-test-6c7675c585-vkf7h   1/1     Running   0          2m
```
