# K8S Yaml

# test

```
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
--- 
apiVersion: v1
kind: Service
metadata:
  name: test-service
spec:
  selector: 
    app: nginx
  type: NodePort  
  ports:
    - port: 80
      targetPort: 80
      nodePort: 32000
EOF
```

# Deployment

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
        test: "true"
    spec:
      hostAliases:
      - hostnames:
        - test.aa.com
        - test.bb.com
        ip: 192.168.0.127
      - hostnames:
        - test.cc.com
        ip: 192.168.0.128
      hostNetwork: true
      tolerations:
      - key: "CriticalAddonsOnly"
        operator: "Exists"
      - operator: "Exists"
        effect: "NoSchedule"
      - operator: "Exists"
        effect: "NoExecute"
      schedulerName: default-scheduler
      nodeSelector:
        resourceGroup: test
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                  - key: "test"
                    operator: In
                    values:
                      - "true"
              topologyKey: "kubernetes.io/hostname"
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
        livenessProbe:
          tcpSocket:
            port: 53
          initialDelaySeconds: 5
          periodSeconds: 10
        volumeMounts:
        - name: config-volume
          mountPath: /etc/kubernetes
        - name: dnsmasq-config
          mountPath: /etc/dnsmasq.conf
          subPath: dnsmasq.conf
      serviceAccountName: service-account-sa
      volumes:
      - name: config-volume
        configMap:
          name: config-demo
      - name: dnsmasq-config
        configMap:
          name: dnsmasq-config
```

# StatefulSet

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  annotations:
    meta.helm.sh/release-name: yani
    meta.helm.sh/release-namespace: default
  labels:
    app.kubernetes.io/instance: yani
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: template
    helm.sh/chart: template-1.0.0
    taskId: yani
  name: yani
  namespace: default
spec:
  podManagementPolicy: OrderedReady
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/instance: yani
      app.kubernetes.io/name: template
  serviceName: yani
  template:
    metadata:
      creationTimestamp: null
      labels:
        app.kubernetes.io/instance: yani
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/name: template
        helm.sh/chart: template-1.0.0
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - /script/start-script.sh
        image: docker.io/gradiant/jupyter:6.0.3
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /
            port: http
            scheme: HTTP
          initialDelaySeconds: 600
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        name: yani
        ports:
        - containerPort: 8888
          name: http
          protocol: TCP
        readinessProbe:
          failureThreshold: 5
          httpGet:
            path: /
            port: http
            scheme: HTTP
          initialDelaySeconds: 30
          periodSeconds: 5
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
          requests:
            cpu: 100m
            memory: 128Mi
        securityContext:
          runAsUser: 1001
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /script
          name: start-script
        - mountPath: /home/jovyan
          name: jupyter-pvc
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 1001
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:
          defaultMode: 0777
          name: yani-start-script
        name: start-script
      - emptyDir: {}
        name: jupyter-pvc

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

```
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hpa-cronjob
  namespace: hpa
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hpa-cronjob
            image: image.ac.com:5000/k8s/kubectl:v1.26.8
            command:
            - "/bin/sh"
            - "-c"
            - "kubectl get no"
            volumeMounts:
            - name: kubeconfig
              mountPath: /root/.kube/config
              subPath: config
          volumes:
          - name: kubeconfig
            secret:
              secretName: hpa-kubeconfig
          restartPolicy: OnFailure
EOF
```

> schedule: "0 8-9 * * *"  # 每天的 8:00 AM 和 9:00 AM 执行
> schedule: "0,59 8-10 * * *"  # 每小时的第 0 分钟和第 59 分钟，执行一次

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

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "common.names.fullname" . }}-start-script
  namespace: {{ .Release.Namespace | quote }}
  labels: {{- include "common.labels.standard" . | nindent 4 }}
    {{- if .Values.commonLabels }}
    {{- include "common.tplvalues.render" ( dict "value" .Values.commonLabels "context" $ ) | nindent 4 }}
    {{- end }}
  {{- if .Values.commonAnnotations }}
  annotations: {{- include "common.tplvalues.render" ( dict "value" .Values.commonAnnotations "context" $ ) | nindent 4 }}
  {{- end }}
data:
  start-script.sh: |
    {{- .Values.script | nindent 4 }}
```

```
script: |
  #!/bin/sh
  echo "hl" > /tmp/yani
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
        - name: resolv-host
          mountPath: /etc/resolv.conf
        - name: resolv-config
          mountPath: /tmp/resolv.conf
          subPath: resolv.conf
      volumes:
      - name: resolv-host
        hostPath:
          path: /etc/resolv.conf
          type: FileOrCreate
      - name: resolv-config
        configMap:
          name: resolv-config
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

```
{{- $serviceType := .Values.service.type -}}
apiVersion: v1
kind: Service
metadata:
  name: {{ include "common.names.fullname" . }}
  namespace: {{ .Release.Namespace | quote }}
  labels: {{- include "common.labels.standard" . | nindent 4 }}
    {{- if .Values.commonLabels }}
    {{- include "common.tplvalues.render" ( dict "value" .Values.commonLabels "context" $ ) | nindent 4 }}
    {{- end }}
  {{- if .Values.commonAnnotations }}
  annotations: {{- include "common.tplvalues.render" ( dict "value" .Values.commonAnnotations "context" $ ) | nindent 4 }}
  {{- end }}
spec:
  type: {{ .Values.service.type }}
  sessionAffinity: {{ default "None" .Values.service.sessionAffinity }}
  {{- if and .Values.service.clusterIP (eq .Values.service.type "ClusterIP") }}
  clusterIP: {{ .Values.service.clusterIP }}
  {{- end }}
  {{- if and .Values.service.loadBalancerIP (eq .Values.service.type "LoadBalancer") }}
  loadBalancerIP: {{ .Values.service.loadBalancerIP }}
  {{- end }}
  {{- if and (eq .Values.service.type "LoadBalancer") .Values.service.loadBalancerSourceRanges }}
  loadBalancerSourceRanges: {{- toYaml .Values.service.loadBalancerSourceRanges | nindent 4 }}
  {{- end }}
  {{- if or (eq .Values.service.type "LoadBalancer") (eq .Values.service.type "NodePort") }}
  externalTrafficPolicy: {{ .Values.service.externalTrafficPolicy | quote }}
  {{- end }}
  ports:
    {{- range .Values.service.ports }}
    - name: {{ .name }}
      port: {{ .port }}
      targetPort: {{ .targetPort }}
      {{- if and (or (eq $serviceType "NodePort") (eq $serviceType "LoadBalancer")) (not (empty .nodePort)) }}
      nodePort: {{ .nodePort }}
      {{- end }}
    {{- end }}
  selector: {{- include "common.labels.matchLabels" . | nindent 4 }}
```

```
service:
  type: ClusterIP # clusterIP LoadBalancer NodePort
  ports:
    - name: http
      port: 80
      targetPort: http
```

# Ingress

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    meta.helm.sh/release-name: yani
  generation: 1
  labels:
    app.kubernetes.io/instance: yani
  name: yani
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - host: hl.test.com
    http:
      paths:
      - backend:
          service:
            name: yani
            port:
              number: 80
        path: /
        pathType: Prefix
```

```
{{- if .Values.ingress.enabled -}}
{{- $fullName := include "common.names.fullname" . -}}
{{- $httpPort := .Values.service.port -}}
{{- $pathType := .Values.ingress.pathType -}}
apiVersion: {{ include "common.capabilities.ingress.apiVersion" . }}
kind: Ingress
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Release.Namespace | quote }}
  labels: {{- include "common.labels.standard" . | nindent 4 }}
    {{- if .Values.commonLabels }}
    {{- include "common.tplvalues.render" ( dict "value" .Values.commonLabels "context" $ ) | nindent 4 }}
    {{- end }}
spec:
  {{- if .Values.ingress.ingressClassName }}
  ingressClassName: {{ .Values.ingress.ingressClassName | quote }}
  {{- end }}
  rules:
  {{- range .Values.ingress.hosts }}
  - host: {{ .host }}
    http:
      paths:
      {{- range .paths }}
      - path: {{ default "/" .path }}
        pathType: {{ default "Prefix" $pathType }}
        backend:
          service:
            name: {{ $fullName }}
            port:
              number: {{ .port | default $httpPort }}
      {{- end }}
  {{- end }}
{{- end }}
```

```
ingress:
  enabled: true
  ingressClassName: nginx
  pathType: Prefix
  apiVersion: "networking.k8s.io/v1"
  hosts:
  - host: hl.test.com
    paths:
    - path: /
      port: 80
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

# Secret

```
kubectl config view --raw > kubeconfig.yaml
cat kubeconfig.yaml | base64

kubectl config view --raw|base64

kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: hpa-kubeconfig
  namespace: hpa
type: Opaque
data:
  config: $(kubectl config view --raw|base64|tr -d '\n')
EOF
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

# Namespace

```
apiVersion: v1
kind: Namespace
metadata:
  name: aaa
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

# hostAliases

```
cat <<EOF| kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  hostAliases:
  - hostnames:
    - test.aa.com
    ip: 10.0.0.1
  - hostnames:
    - test1.aa.com
    - test2.aa.com
    ip: 10.0.0.2
  containers:
  - name: test
    image: busybox:1.28
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "sleep 3000"]
EOF
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

# j2

> indent=0 指定转换时不增加额外的缩进。
> indent(2)：在生成的 YAML 内容前面添加 2 个空格的缩进，以便在嵌套上下文中正确对齐。

```
# vars/main.yaml
dnsmasq_replica_count: "{{ groups['dnsmasq'] | length }}"
resources:
  requests:
    cpu: 200m
    memory: 128Mi
  limits:
    cpu: 4
    memory: 4Gi
dnsmasq_global_conf: |
  no-hosts
  no-resolv
  no-poll
  neg-ttl=300
  min-cache-ttl=300
  dns-forward-max=10000
  cache-size=100000
  edns-packet-max=1232
  log-facility=/var/log/dnsmasq.log
  all-servers
  server=114.114.114.114
  server=8.8.8.8
dnsmasq_address_conf: |
  address=/image.ac.com/127.0.0.1
tolerations:
- key: "CriticalAddonsOnly"
  operator: "Exists"
- operator: "Exists"
  effect: "NoSchedule"
- operator: "Exists"
  effect: "NoExecute"
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 1
      podAffinityTerm:
        labelSelector:
          matchExpressions:
            - key: "{{ ingress_label }}"
              operator: In
              values:
                - "true"
        topologyKey: "kubernetes.io/hostname"
config:
  worker-processes: "auto"
  worker-cpu-affinity: "auto"
  max-worker-connections: "16384"

# value.yaml.j2
image:
  repository: {{ dnsmasq_image }}
  pullPolicy: IfNotPresent
name: {{ dnsmasq_name }}
namespace: {{ dnsmasq_namespace }}
replicaCount: {{ dnsmasq_replica_count}}
podLabel: {{ dnsmasq_label }}
resources:
{{ resources | to_nice_yaml(indent=2) }}
dnsmasqConfig: | # 字符串
  {{ dnsmasq_global_conf | to_nice_yaml | from_yaml | indent(2) }}
  {{ dnsmasq_address_conf | to_nice_yaml | from_yaml | indent(2) }}
config:
  {{ config | to_nice_yaml(indent=0) | indent(2) }}
affinity:
  {{ affinity | to_nice_yaml(indent=2) }}
tolerations:
{{ tolerations | to_nice_yaml(indent=0) }}
  
# resolv.conf.j2
{% for item in groups['dnsmasq'] %}
nameserver {{ item }}
{% endfor %}
options timeout:1 attempts:3 rotate

```

```
nodeSelector:
  ingress: "true"
  
  nodeSelector:
    {{ ingress_label }}: "true"
```

```
  tolerations:
  - key: CriticalAddonsOnly
    operator: Exists
  - effect: NoSchedule
    operator: Exists
  - effect: NoExecute
    operator: Exists

# indent=0 指定转换时不增加额外的缩进。
# indent(2)：在生成的 YAML 内容前面添加 2 个空格的缩进，以便在嵌套上下文中正确对齐。
  tolerations:
  {{ tolerations | to_nice_yaml(indent=0) | indent(2) }}
```

# ansible

```
# file/chart/templates/test.yaml
...
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
...

# file/chart/templates/cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: dnsmasq-config
  namespace: {{ .Values.namespace }}
data:
  dnsmasq.conf: |
    {{- .Values.dnsmasqConfig | nindent 4 }}

# task/main.yaml
- name: install
  import_tasks: helm-install.yml
  vars:
    version: "{{ version }}"
    name: "{{ iname }}"

- name: 轮询等待 svc 运行
  wait_for:
    host: "{{ groups['ingress'][0] }}"
    port: "{{ ingress_nodePort }}"
    delay: 10
    timeout: 180

- name: get svc ip
  shell: "{{ kubectl }} get svc test -o jsonpath='{.spec.clusterIP}'"
  register: test_ip

- name: echo stdout
  shell:
    cmd: |
      echo {{ test_ip.stdout }}
  delegate_to: "{{ item }}"
  loop: "{{ groups['ingress'] }}"
  when: etcd.external | default(false) != true

- name: clean
  import_tasks: clean.yml
  when: "'clean' in ansible_run_tags"
  tags:
    - clean
```

