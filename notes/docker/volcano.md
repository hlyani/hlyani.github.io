# Volcano

[https://github.com/volcano-sh/volcano](https://github.com/volcano-sh/volcano)

[https://docs.otc.t-systems.com/cloud-container-engine/umn/add-ons/volcano_scheduler.html](https://docs.otc.t-systems.com/cloud-container-engine/umn/add-ons/volcano_scheduler.html)

# 一、安装

##### 1、helm

```
helm repo add volcano-sh https://volcano-sh.github.io/helm-charts
helm repo update
helm pull volcano-sh/volcano
helm install volcano volcano-sh/volcano -n volcano-system --create-namespace
```

##### 2、镜像

```
volcanosh/vc-controller-manager:v1.9.0
volcanosh/vc-webhook-manager:v1.9.0
volcanosh/vc-scheduler:v1.9.0
```

# 二、binpack

> priority，优先调度，逻辑是打分机制，默认打分机制是0-100，根据资源情况使用情况，使用越高，分数越低。
>
> binpack 刚好相反，资源使用越低，分数越低。这种调度算法能够尽可能减小节点内的碎片，在空闲的机器上为申请了更大资源请求的Pod预留足够的资源空间，使集群下空闲资源得到最大化的利用。LeastRequestedPriority

## 1、queue

```
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: default
spec:
  reclaimable: true          # 表示该queue在资源使用量超过该queue所应得的资源份额时，是否允许其他queue回收该queue使用超额的资源，默认值为true。（需要开启reclaim action）
  weight: 1                  # 软限制  表示该queue在集群资源划分中所占的相对比重，该queue应得资源总量为 (weight/total-weight) * total-resource。当queue中任务较多时可以超过weight限制，去借用其他空闲queue的部分资源。
  capability:                # 硬限制  表示该queue内所有podgroup使用资源量之和的上限。
    cpu: 4
    memory: 4096Mi
  quarantee：                # 硬限制  表示资源预留配置，即使queue空闲，也为其保留这些计算资源，这部分不许被其他queue共享。
    resource:
      cpu: 500m
      memory: 1024Mi
```



```
cat queue.yaml 
# create test queue and then run deployment.
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: queue
spec:
  weight: 1
  reclaimable: false
  capability:
    cpu: 1
```

```
cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-with-volcano
  labels:
    app: nginx
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        # create test queue and use this annotation
        # to make deployment be scheduled into test queue
        scheduling.volcano.sh/queue-name: queue
    spec:
      # set spec.schedulerName to 'volcano' instead of
      # 'default-scheduler' for deployment.
      schedulerName: volcano
      containers:
        - name: nginx
          image: easzlab.io.local:5000/nginx:1.21.3
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
```



```
cat volcano-scheduler-binpack.conf 
actions: "enqueue, allocate, backfill"
tiers:
- plugins:
  - name: binpack
  - name: gang
    enablePreemptable: false
  - name: conformance
- plugins:
  - name: overcommit
    enablePreemptable: false
  - name: predicates
```

```
cat volcano-scheduler-default.conf 
actions: "enqueue, allocate, backfill"
tiers:
- plugins:
  - name: priority
  - name: gang
    enablePreemptable: false
  - name: conformance
- plugins:
  - name: overcommit
  - name: drf
    enablePreemptable: false
  - name: predicates
  - name: proportion
  - name: nodeorder
  - name: binpack
```





```
actions: "enqueue, allocate, backfill"
tiers:
- plugins:
  - name: gang                          # gang插件没有参数，开启即可。 podgroup的minMember依赖于gang插件
  - name: predicates
    arguments:
      predicate.ProportionalEnable: true
      predicate.resources: nvidia.com/gpu
      predicate.resources.nvidia.com/gpu.cpu: 8
      predicate.resources.nvidia.com/gpu.memory: 8
  - name: nodeorder
    arguments:
      leastrequested.weight: 1
      mostrequested.weight: 0
      nodeaffinity.weight: 1
      podaffinity.weight: 1
      balancedresource.weight: 1
      tainttoleration.weight: 1
      imagelocality.weight: 2
  - plugins:
    - name: binpack
      arguments:
        binpack.weight: 10              # binpack插件的权重（allocate action有许多计算插件，可以把某个插件权重调大，作为主要计算标准）
        binpack.cpu: 2                  # binpack计算时，cpu所占权重较高
        binpack.memory: 1               # binpack计算时，mem所占权重较低
```



## volcano scheduler 

```
colocation_enable: ''
default_scheduler_conf:
  actions: 'allocate, backfill, preempt'
  tiers:
    - plugins:
        - name: 'priority'
        - name: 'gang'
        - name: 'conformance'
        - name: 'lifecycle'
          arguments:
            lifecycle.MaxGrade: 10
            lifecycle.MaxScore: 200.0
            lifecycle.SaturatedTresh: 1.0
            lifecycle.WindowSize: 10
    - plugins:
        - name: 'drf'
        - name: 'predicates'
        - name: 'nodeorder'
    - plugins:
        - name: 'cce-gpu-topology-predicate'
        - name: 'cce-gpu-topology-priority'
        - name: 'cce-gpu'
    - plugins:
        - name: 'nodelocalvolume'
        - name: 'nodeemptydirvolume'
        - name: 'nodeCSIscheduling'
        - name: 'networkresource'
tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 60
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 60
```

### actions

The following options are supported:

- **enqueue**: uses a series of filtering algorithms to filter out tasks to be scheduled and sends them to the queue to wait for scheduling. After this action, the task status changes from **pending** to **inqueue**.
- **allocate**: selects the most suitable node based on a series of pre-selection and selection algorithms.
- **preempt**: performs preemption scheduling for tasks with higher priorities in the same queue based on priority rules.
- **backfill**: schedules pending tasks as much as possible to maximize the utilization of node resources.



### plugins

#### 1.binpack

```
- plugins:
  - name: binpack
    arguments:
      binpack.weight: 10
      binpack.cpu: 1
      binpack.memory: 1
      binpack.resources: nvidia.com/gpu, example.com/foo
      binpack.resources.nvidia.com/gpu: 2
      binpack.resources.example.com/foo: 3
```

#### 2.conformance

```
- plugins:
  - name: 'priority'
  - name: 'gang'
    enablePreemptable: false
  - name: 'conformance'
```

#### 3.lifecycle

```
- plugins:
  - name: priority
  - name: gang
    enablePreemptable: false
  - name: conformance
  - name: lifecycle
    arguments:
      lifecycle.MaxGrade: 10
      lifecycle.MaxScore: 200.0
      lifecycle.SaturatedTresh: 1.0
      lifecycle.WindowSize: 10
```

#### 4.Gang

```
- plugins:
  - name: priority
  - name: gang
    enablePreemptable: false
    enableJobStarving: false
  - name: conformance
```

#### 5.priority

```
- plugins:
  - name: priority
  - name: gang
    enablePreemptable: false
  - name: conformance
```

#### 6.overcommit

```
- plugins:
  - name: overcommit
    arguments:
      overcommit-factor: 2.0
```

#### 7.drf

```
- plugins:
  - name: 'drf'
  - name: 'predicates'
  - name: 'nodeorder'
```

#### 8.predicates

```
- plugins:
  - name: 'drf'
  - name: 'predicates'
  - name: 'nodeorder'
```

#### 9.nodeorder

```
- plugins:
  - name: nodeorder
    arguments:
      leastrequested.weight: 1
      mostrequested.weight: 0
      nodeaffinity.weight: 2
      podaffinity.weight: 2
      balancedresource.weight: 1
      tainttoleration.weight: 3
      imagelocality.weight: 1
      podtopologyspread.weight: 2
```

# 三、架构

Volcano 核心组件主要包含三个：Admission、ControllerManager、Scheduler

* Admission 对 Volcano CRD API 提供校验能力
* ControllerManager 负责对 Volcano CRD 进行资源管理
* Scheduler 对任务提供丰富的调度能力

## 1.PodGroup

定义一组强关联的 Pod

```
apiVersion: scheduling.volcano.sh/v1beta1
kind: PodGroup
metadata:
  name: test
  namespace: default
spec:
  minMember: 1               # 被gang插件使用，表示该podgroup下最少需要运行的pod数量。如果集群资源不满足miniMember数量的pod运行，调度器将不会调度任何一个该podgroup内的pod。
  minResources:              # minResources表示运行该podgroup所需要的最少资源。当集群可分配资源不满足minResources时，调度器将不会调度任何一个该podgroup内的pod。
    cpu: "3"
    memory: "2048Mi"
  priorityClassName: high-prority
  queue: default             # queue表示该podgroup所属的queue。queue必须提前已创建且状态为open。
```

## 2.Queue

划分集群中的计算资源，也是容纳PodGroup的队列

```
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: default
spec:
  reclaimable: true          # 表示该queue在资源使用量超过该queue所应得的资源份额时，是否允许其他queue回收该queue使用超额的资源，默认值为true。（需要开启reclaim action）
  weight: 1                  # 软限制  表示该queue在集群资源划分中所占的相对比重，该queue应得资源总量为 (weight/total-weight) * total-resource。当queue中任务较多时可以超过weight限制，去借用其他空闲queue的部分资源。
  capability:                # 硬限制  表示该queue内所有podgroup使用资源量之和的上限。
    cpu: 4
    memory: 4096Mi
  quarantee：                # 硬限制  表示资源预留配置，即使queue空闲，也为其保留这些计算资源，这部分不许被其他queue共享。
    resource:
      cpu: 500m
      memory: 1024Mi
```

## 3.Volcano Job

vcjob，是 volcano 自定义的 job 资源类型。区别于 k8s job，vcjob 提供了更多高级功能，如可指定调度器、支持最小运行 pod 数、支持 task、支持生命周期管理、支持指定队列、支持优先级调度等。

```
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: volcano-sample-job
spec:
  minAvailable: 2  # Gang Scheduling：至少 2 个任务必须可用，任务才会被调度
  schedulerName: volcano
  tasks:
    - replicas: 2  # 任务的副本数
      name: "task1"
      template:
        spec:
          containers:
            - image: nginx
              name: nginx
              command: ["/bin/sh", "-c", "sleep 30"]
          restartPolicy: Never
    - replicas: 1  # 另一个任务
      name: "task2"
      policies:
        - event: TaskCompleted
          action: CompleteJob
      template:
        spec:
          containers:
            - image: busybox
              name: busybox
              command: ["/bin/sh", "-c", "echo 'Task 2 completed'"]
          restartPolicy: Never
```

## 4.Queue、PodGroup、VolcanoJob 关系

Queue 是一个 PodGroup 队列，PodGroup 是一组强关联的 Pod 集合。

vcjob 对应的下一级资源是 PodGroup。

## 5.插件

```
actions: "enqueue, allocate, backfill"
tiers:
- plugins:
  - name: gang                          ## gang插件没有参数，开启即可。 podgroup的minMember依赖于gang插件
  - name: predicates
    arguments:
      predicate.ProportionalEnable: true
      predicate.resources: nvidia.com/gpu
      predicate.resources.nvidia.com/gpu.cpu: 8
      predicate.resources.nvidia.com/gpu.memory: 8
  - name: nodeorder
    arguments:
      leastrequested.weight: 1
      mostrequested.weight: 0
      nodeaffinity.weight: 1
      podaffinity.weight: 1
      balancedresource.weight: 1
      tainttoleration.weight: 1
      imagelocality.weight: 2
  - plugins:
    - name: binpack
      arguments:
        binpack.weight: 10              ## binpack插件的权重（allocate action有许多计算插件，可以把某个插件权重调大，作为主要计算标准）
        binpack.cpu: 2                  ## binpack计算时，cpu所占权重较高
        binpack.memory: 1               ## binpack计算时，mem所占权重较低
```

