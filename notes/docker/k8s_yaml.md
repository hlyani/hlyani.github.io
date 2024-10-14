# K8S Yaml

# 一、deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment
  namespace: test-deployment
  labels:
    key: val
spec:
  replicas: 1
  selector:
    matchLabels:
     app: test-deployment
  template:
    metadata:
      labels:
        app: test-deployment
    spec:
      hostNetwork: true
      schedulerName: default-scheduler
      nodeSelector:
        resourceGroup: test
      restartPolicy: Always   # Never
      terminationGracePeriodSeconds: 30
      containers:
      - name: test-deployment
        image: docker.io/library/nginx:1.21.3
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 22
          name: ssh
          protocol: TCP
        securityContext:
          runAsUser: 0
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        env:
        - name: http_proxy
          value: http://xxxx:3120
        command:
        - nginx -g daemon off;
        resources:
          requests:
            cpu: 200m
            memory: 128Mi
          limits:
            cpu: "8"
            memory: 20Gi
            nvidia.com/gpu: "1"
        livenessProbe:
          httpGet:
            path: /healthz
            port: 80
            scheme: HTTPS
        volumeMounts:
        - name: config-volume
          mountPath: /etc/kubernetes
      serviceAccountName: service-account-sa
      volumes:
      - name: config-volume
        configMap:
          name: config-demo
```

# Job

```
apiVersion: batch/v1  
kind: Job  
metadata:  
  name: hello-job  
spec:
  activeDeadlineSeconds: 60 # 设置Job的超时时间为60秒  
  template:  
    spec:  
      containers:  
      - name: hello-job-container  
        image: busybox  
        command: ["echo", "Hello, Kubernetes Job!"]  
        # 如果你想让容器保持运行一段时间（比如模拟长时间运行的任务），  
        # 可以使用下面的命令替换上面的echo命令。但请注意，这通常不是Job的预期用途。  
        # command: ["sh", "-c", "echo Hello, Kubernetes Job! && sleep 30"]  
      restartPolicy: Never  
  backoffLimit: 0 # 设置为0表示不允许重试，根据你的需求调整
```

# CronJob

```
apiVersion: batch/v1beta1  
kind: CronJob  
metadata:  
  name: my-cronjob  
spec:  
  schedule: "*/1 * * * *" # 每分钟的每一秒执行一次（注意：CronJob的精度通常到分钟，这里仅为示例）  
  jobTemplate:  
    spec:  
      template:  
        spec:  
          containers:  
          - name: my-container  
            image: busybox  
            command: ["echo", "$(date) Hello from the Kubernetes cron job"]  
          restartPolicy: OnFailure  
  successfulJobsHistoryLimit: 3  
  failedJobsHistoryLimit: 1
```

# ConfigMap

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-demo
  namespace: kube-system
data:
  scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    profiles:
    - schedulerName: d-scheduler
    - schedulerName: image-locality-scheduler
```

# ClusterRoleBinding

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-role-crb
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-role-cr
subjects:
  - kind: ServiceAccount
    name: service-account-sa
    namespace: kube-system
```

# ClusterRole

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-role-cr
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
```

# ServiceAccount

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: service-account-sa
  namespace: kube-system
```

# daemonset

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: dnsmasq
  namespace: kube-system
spec:
  selector:
    matchLabels:
      dns: dnsmasq
  template:
    metadata:
      labels:
        dns: dnsmasq
    spec:
      tolerations:
      - key: "CriticalAddonsOnly"
        operator: "Exists"
      - operator: "Exists"
        effect: "NoSchedule"
      - operator: "Exists"
        effect: "NoExecute"
      containers:
      - name: dnsmasq
        image: easzlab.io.local:5000/busybox:1.28
        command: ["/bin/sh", "-c"]
        args:
        - |
          cat /opt/resolv.conf > /etc/resolv.conf
          tail -f /dev/null
        volumeMounts:
        - name: resolv
          mountPath: /etc/resolv.conf
        - name: dnsmasq-config
          mountPath: /opt
      volumes:
      - name: resolv
        hostPath:
          path: /etc/resolv.conf
      - name: dnsmasq-config
        configMap:
          name: dnsmasq-config
```

# Service

```
apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: ingress-apisix
    meta.helm.sh/release-namespace: ingress-apisix
  labels:
    app.kubernetes.io/component: etcd
    app.kubernetes.io/instance: ingress-apisix
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: etcd
    app.kubernetes.io/version: 3.5.10
    helm.sh/chart: etcd-9.7.3
  name: apisix-etcd
  namespace: ingress-apisix
spec:
  clusterIP: 10.127.15.211
  clusterIPs:
  - 10.127.15.211
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: client
    port: 2379
    protocol: TCP
    targetPort: client
  - name: peer
    port: 2380
    protocol: TCP
    targetPort: peer
  selector:
    app.kubernetes.io/component: etcd
    app.kubernetes.io/instance: ingress-apisix
    app.kubernetes.io/name: etcd
  sessionAffinity: None
  type: ClusterIP
```

# pod

```
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  schedulerName: default-scheduler
  containers:
  - name: test-pod
    image: docker.io/library/nginx:1.21.3
```

# ServiceMonitor

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  annotations:
    meta.helm.sh/release-name: ingress-apisix
    meta.helm.sh/release-namespace: ingress-apisix
  labels:
    app.kubernetes.io/instance: ingress-apisix
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: apisix
    app.kubernetes.io/version: 3.9.1
    helm.sh/chart: apisix-2.8.1
    release: ai-kube-prometheus-stack
  name: apisix
  namespace: ingress-apisix
spec:
  endpoints:
  - interval: 15s
    path: /apisix/prometheus/metrics
    scheme: http
    targetPort: prometheus
  namespaceSelector:
    matchNames:
    - ingress-apisix
  selector:
    matchLabels:
      app.kubernetes.io/instance: ingress-apisix
      app.kubernetes.io/managed-by: Helm
      app.kubernetes.io/name: apisix
      app.kubernetes.io/service: apisix-prometheus-metrics
      app.kubernetes.io/version: 3.9.1
      helm.sh/chart: apisix-2.8.1
```

# PodMonitor

```
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  labels:
    app: apisix-etcd
    app.kubernetes.io/instance: ingress-apisix
    app.kubernetes.io/name: etcd
    app.kubernetes.io/version: 3.5.10
    release: ai-kube-prometheus-stack
  name: apisix-etcd
  namespace: monitoring
spec:
  podMetricsEndpoints:
  - interval: 15s
    path: /metrics
    scheme: http
    port: client
  namespaceSelector:
    matchNames:
    - ingress-apisix
  selector:
    matchLabels:
      app.kubernetes.io/instance: ingress-apisix
      app.kubernetes.io/name: etcd
      app.kubernetes.io/component: etcd
```

# api-resources

```
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
persistentvolumeclaims            pvc          v1                                     true         PersistentVolumeClaim
persistentvolumes                 pv           v1                                     false        PersistentVolume
pods                              po           v1                                     true         Pod
podtemplates                                   v1                                     true         PodTemplate
replicationcontrollers            rc           v1                                     true         ReplicationController
resourcequotas                    quota        v1                                     true         ResourceQuota
secrets                                        v1                                     true         Secret
serviceaccounts                   sa           v1                                     true         ServiceAccount
services                          svc          v1                                     true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1        false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io/v1        false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1                false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io/v1              false        APIService
apisixclusterconfigs              acc          apisix.apache.org/v2                   false        ApisixClusterConfig
apisixconsumers                   ac           apisix.apache.org/v2                   true         ApisixConsumer
apisixglobalrules                 agr          apisix.apache.org/v2                   true         ApisixGlobalRule
apisixpluginconfigs               apc          apisix.apache.org/v2                   true         ApisixPluginConfig
apisixroutes                      ar           apisix.apache.org/v2                   true         ApisixRoute
apisixtlses                       atls         apisix.apache.org/v2                   true         ApisixTls
apisixupstreams                   au           apisix.apache.org/v2                   true         ApisixUpstream
controllerrevisions                            apps/v1                                true         ControllerRevision
daemonsets                        ds           apps/v1                                true         DaemonSet
deployments                       deploy       apps/v1                                true         Deployment
replicasets                       rs           apps/v1                                true         ReplicaSet
statefulsets                      sts          apps/v1                                true         StatefulSet
tokenreviews                                   authentication.k8s.io/v1               false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io/v1                true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io/v1                false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io/v1                false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io/v1                false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling/v2                         true         HorizontalPodAutoscaler
cronjobs                          cj           batch/v1                               true         CronJob
jobs                                           batch/v1                               true         Job
certificatesigningrequests        csr          certificates.k8s.io/v1                 false        CertificateSigningRequest
leases                                         coordination.k8s.io/v1                 true         Lease
endpointslices                                 discovery.k8s.io/v1                    true         EndpointSlice
events                            ev           events.k8s.io/v1                       true         Event
flowschemas                                    flowcontrol.apiserver.k8s.io/v1beta3   false        FlowSchema
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta3   false        PriorityLevelConfiguration
nodes                                          metrics.k8s.io/v1beta1                 false        NodeMetrics
pods                                           metrics.k8s.io/v1beta1                 true         PodMetrics
alertmanagerconfigs               amcfg        monitoring.coreos.com/v1alpha1         true         AlertmanagerConfig
alertmanagers                     am           monitoring.coreos.com/v1               true         Alertmanager
podmonitors                       pmon         monitoring.coreos.com/v1               true         PodMonitor
probes                            prb          monitoring.coreos.com/v1               true         Probe
prometheusagents                  promagent    monitoring.coreos.com/v1alpha1         true         PrometheusAgent
prometheuses                      prom         monitoring.coreos.com/v1               true         Prometheus
prometheusrules                   promrule     monitoring.coreos.com/v1               true         PrometheusRule
scrapeconfigs                     scfg         monitoring.coreos.com/v1alpha1         true         ScrapeConfig
servicemonitors                   smon         monitoring.coreos.com/v1               true         ServiceMonitor
thanosrulers                      ruler        monitoring.coreos.com/v1               true         ThanosRuler
ingressclasses                                 networking.k8s.io/v1                   false        IngressClass
ingresses                         ing          networking.k8s.io/v1                   true         Ingress
networkpolicies                   netpol       networking.k8s.io/v1                   true         NetworkPolicy
runtimeclasses                                 node.k8s.io/v1                         false        RuntimeClass
poddisruptionbudgets              pdb          policy/v1                              true         PodDisruptionBudget
clusterrolebindings                            rbac.authorization.k8s.io/v1           false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io/v1           false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io/v1           true         RoleBinding
roles                                          rbac.authorization.k8s.io/v1           true         Role
priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass
csidrivers                                     storage.k8s.io/v1                      false        CSIDriver
csinodes                                       storage.k8s.io/v1                      false        CSINode
csistoragecapacities                           storage.k8s.io/v1                      true         CSIStorageCapacity
storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment
```





# Demo

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: apisix-etcd-monitor
  namespace: ingress-apisix
spec:
  selector:
    matchLabels:
      app: apisix-etcd-monitor
  template:
    metadata:
      labels:
        app: apisix-etcd-monitor
    spec:
      hostNetwork: true
      nodeSelector:
        monitor: apisix-etcd
      containers:
      - name: apisix-etcd-monitor
        image: easzlab.io.local:5000/alpine:3.10.4
        ports:
        - name: metrics
          containerPort: 3379
          hostPort: 3379
        command:
        - sh
        - -c
        - tail -f /dev/null
---
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  labels:
    app: apisix-etcd-monitor
    release: ai-kube-prometheus-stack
  name: apisix-etcd-monitor
  namespace: monitoring
spec:
  podMetricsEndpoints:
  - interval: 15s
    path: /metrics
    scheme: http
    port: metrics
  namespaceSelector:
    matchNames:
    - ingress-apisix
  selector:
    matchLabels:
      app: apisix-etcd-monitor
```

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: apisix-etcd-monitor
  namespace: ingress-apisix
spec:
  selector:
    matchLabels:
      app: apisix-etcd-monitor
  template:
    metadata:
      labels:
        app: apisix-etcd-monitor
    spec:
      hostNetwork: true
      nodeSelector:
        monitor: apisix-etcd
      containers:
      - name: apisix-etcd-monitor
        image: easzlab.io.local:5000/alpine:3.10.4
        ports:
        - name: metrics
          containerPort: 3379
          hostPort: 3379
        command:
        - sh
        - -c
        - tail -f /dev/null
---
apiVersion: v1
kind: Service
metadata:
  name: apisix-etcd-monitor
  namespace: ingress-apisix
  labels:
    app: apisix-etcd-monitor-svc
spec:
  selector:
    app: apisix-etcd-monitor
  ports:
  - name: metrics
    protocol: TCP
    port: 3379
    targetPort: metrics
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    release: ai-kube-prometheus-stack
  name: apisix-etcd-monitor
  namespace: ingress-apisix
spec:
  endpoints:
  - interval: 15s
    path: /metrics
    scheme: http
    targetPort: metrics
  namespaceSelector:
    matchNames:
    - ingress-apisix
  selector:
    matchLabels:
      app: apisix-etcd-monitor-svc
```

