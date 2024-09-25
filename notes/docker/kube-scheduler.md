# kube-scheduler

[https://kubernetes.io/docs/reference/scheduling/config/](https://kubernetes.io/docs/reference/scheduling/config/)

[https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/](https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/)

[https://tanjunchen.github.io/post/2024-04-08-scheduler-framework-03/](https://tanjunchen.github.io/post/2024-04-08-scheduler-framework-03/)

# 插件

- ImageLocality: score
- TaintToleration: filter, preScore, score
- NodeName: filter
- NodePorts: preFilter, filter
- NodeAffinity: filter, score
- PodTopologySpread: preFilter, filter, preScore, score
- NodeUnschedulable: filter
- NodeResourcesFit：LeastAllocated、MostAllocated、RequestedToCapacityRatio : preFilter, filter, score
- NodeResourcesBalancedAllocation: score
- VolumeBinding:  preFilter, filter, reserve, preBind, score
- VolumeRestrictions: filter
- VolumeZone: filter
- NodeVolumeLimits: filter
- EBSLimits: filter
- GCEPDLimits: filter
- AzureDiskLimits: filter
- InterPodAffinity: preFilter, filter, preScore, score
- PrioritySort: queueSort
- DefaultBinder: bind
- DefaultPreemption: postFilter
- CinderLimits: filter

# 扩展点

- queueSort
- preFilter
- filter
- postFilter
- preScore
- score
- reserve
- permit
- preBind
- bind
- postBind
- multiPoint

# 一、调度插件

## 1.DefaultPreemption

默认抢占机制，当调度器发现当前急群中的节点无法满足新Pod的资源需求时，可能会通过抢占低优先级的Pod，腾出足够的资源来满足高优先级Pod的需求。

```
spec.PreemptionPolicy
PriorityClass
```

> 默认为 PreemptLowerPriority，即该 Pod 可以抢占低优先级的 Pod

## 2.InterPodAffinity

允许用户指定两个或多个 Pod 之间的亲和性要求，确保它们在特定条件下可以调度到相同或相关的节点上。

```
Affinity
AntiAffinity
```

* requiredDuringSchedulingIgnoredDuringExecution：这是一个强制性的亲和性规则，表示调度时必须满足这些条件。

* preferredDuringSchedulingIgnoredDuringExecution：表示这是一个软要求，调度器会尽量满足，但如果无法满足，也不会阻止调度。

## 3.**NodeAffinity** 

允许用户基于节点的标签来控制 Pod 的调度位置。

```
nodeSelector
```

## 4.NodeResourcesBalancedAllocation

在多个节点中选择资源分配更加均衡的节点来调度 Pod，通过评估节点的 CPU 和内存利用率，确保 Pod 不会集中在资源过度的使用的节点上，也不会导致某些节点的资源闲置不被利用。主要目的是在节点的 CPU 和内存使用之前取得平衡。

1.CPU 和内存使用的均衡性：根据资源利用率，优先选择资源更为均衡的节点。

2.打分机制：节点的打分介于0到100分之间，得分越高表示资源分配越均衡。

## 5.NodeResourcesFit

根据接的可用资源（如 CPU 和内存）判断 Pod 是否可以调度该节点。确保每个 Pod 都能分配到满足其资源请求的节点。主要依据 requests 和节点的资源可用性来进行调度决策。

## 6.PodTopologySpread

确保 Pods 在指定的拓扑域（例如可用区、区域或其他标签）之间均匀分布。该功能对与增强应用程序的弹性和可用性非常有用，因为可以防止  Pods 集中在单个故障域中。

1.拓扑域：pods 应该分布的区域。包括不同的节点、可用区或区域。

2.分布约束：可以定义约束，指定在每个拓扑域中可以存在多少 Pods。例如 ，可能希望在每个可用区中至少有一个 Pod。

3.权重：可以为不同的拓扑域分配权重，从而允许优先考虑在某些域之间的分布。



# 示例

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: multipoint-scheduler-sa
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: multipoint-scheduler-cr
rules:
  - apiGroups: [""]
    resources:
      - pods
      - pods/logs
      - pods/status
      - pods/binding
      - bindings
      - nodes
      - events
      - services
      - namespaces
      - configmaps
      - secrets
      - serviceaccounts
      - resourcequotas
      - replicationcontrollers
      - persistentvolumes
      - persistentvolumeclaims
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete
  - apiGroups: ["apps"]
    resources:
      - replicasets
      - statefulsets
      - deployments
      - daemonsets
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete
  - apiGroups: ["storage.k8s.io"]
    resources:
      - storageclasses
      - volumeattachments
      - csinodes
      - csidrivers
      - csistoragecapacities
    verbs:
      - get
      - list
      - watch
  - apiGroups: ["policy"]
    resources:
      - poddisruptionbudgets
    verbs:
      - get
      - list
      - watch
  - apiGroups: ["k8s.io", "events.k8s.io"]
    resources:
      - priorityclasses
      - events
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete
  - apiGroups: ["node"]
    resources:
      - runtimeclasses
    verbs:
      - get
      - list
      - watch
  - apiGroups: ["coordination.k8s.io"]
    resources:
      - leases
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: multipoint-scheduler-crb
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: multipoint-scheduler-cr
subjects:
  - kind: ServiceAccount
    name: multipoint-scheduler-sa
    namespace: kube-system
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: multipoint-scheduler-config
  namespace: kube-system
data:
  scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    profiles:
    - schedulerName: d-scheduler
    - pluginConfig:
      - args:
          apiVersion: kubescheduler.config.k8s.io/v1
          hardPodAffinityWeight: 1
          kind: InterPodAffinityArgs
        name: InterPodAffinity
      - args:
          apiVersion: kubescheduler.config.k8s.io/v1
          kind: NodeAffinityArgs
        name: NodeAffinity
      - args:
          apiVersion: kubescheduler.config.k8s.io/v1
          kind: NodeResourcesFitArgs
          scoringStrategy:
            resources:
            - name: cpu
              weight: 1
            - name: memory
              weight: 1
            type: MostAllocated
        name: NodeResourcesFit
      plugins:
        multiPoint:
          enabled:
          - name: TaintToleration
            weight: 2
          - name: NodeAffinity
            weight: 2
          - name: NodeResourcesFit
            weight: 2
          - name: InterPodAffinity
            weight: 2
          - name: ImageLocality
            weight: 25
          disabled:
          - name: NodeResourcesBalancedAllocation
          - name: PodTopologySpread
      schedulerName: image-locality-scheduler
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multipoint-scheduler
  namespace: kube-system
  labels:
    component: multipoint-scheduler
spec:
  replicas: 2
  selector:
    matchLabels:
      component: multipoint-scheduler
  template:
    metadata:
      labels:
        component: multipoint-scheduler
        name: multipoint-scheduler
        tier: control-plane
    spec:
      containers:
      - name: multipoint-scheduler
        image: easzlab.io.local:5000/k8s.gcr.io/kube-scheduler:v1.26.8
        imagePullPolicy: IfNotPresent
        command:
        - kube-scheduler
        - --config=/etc/kubernetes/scheduler-config.yaml
        - --leader-elect=true
        - --leader-elect-resource-name=multipoint-scheduler
        - --logging-format=text
        - --v=6
        resources:
          requests:
            cpu: 200m
            memory: 128Mi
          limits:
            memory: 128Mi
        livenessProbe:
          httpGet:
            path: /healthz
            port: 10259
            scheme: HTTPS
        volumeMounts:
        - name: config-volume
          mountPath: /etc/kubernetes
      serviceAccountName: multipoint-scheduler-sa
      volumes:
      - name: config-volume
        configMap:
          name: multipoint-scheduler-config
```

[https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)

>         - --logging-format=json
>         - --v=10

>         - --logging-format=text
>         - --v=6

>         - --leader-elect=false
>       
>         - --leader-elect=true
>         - --leader-elect-resource-name=multipoint-scheduler

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: yani
spec:
  replicas: 1
  selector:
    matchLabels:
     app: yani
  template:
    metadata:
      labels:
        app: yani
    spec:
      schedulerName: image-locality-scheduler
      containers:
      - name: yani
        image: docker.io/library/nginx:1.21.3
        imagePullPolicy: IfNotPresent
```

