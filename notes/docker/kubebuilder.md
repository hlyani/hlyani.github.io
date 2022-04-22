# kubebuilder

# 一、环境准备

## 1、容器环境

```
docker run -itd --name go_dev -w /go/src -v /go_src:/go/src/ golang:1.17.6 bash
docker exec -it go_dev bash
```

## 2、安装kubebuilder

```
curl -L -o kubebuilder https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)
chmod +x kubebuilder && mv kubebuilder /usr/local/bin/
```

## 3、设置 goproxy

```
go env -w GO111MODULE=on && go env -w GOPROXY=https://goproxy.cn,direct
```

## 4、编译环境（可选）

```
FROM golang:1.17.6

ARG http_proxy=http://192.168.0.1:1080
ENV http_proxy=$http_proxy
ENV https_proxy=$http_proxy

WORKDIR /go/src

RUN curl -L -o kubebuilder https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH) && \
    chmod +x kubebuilder && mv kubebuilder /usr/local/bin/ && \
    go env -w GO111MODULE=on && go env -w GOPROXY=https://goproxy.cn,direct
```

```
docker build -t crd_dev_golang1.17.6:1.0.0 .
```

```
## 开发环境
docker run -itd --name crd_dev -w /go/src -v /crd_rtc:/go/src/ 192.168.0.90:3000/amd64/crd_dev_golang1.17.6:1.0.0
docker exec -it crd_dev bash

## 运行
make manifests
make install
make run

## 创建实例
kubectl apply -f rt.yaml
```

# 二、开发

## 1、初始化项目

```
go mod init rtcontainer
kubebuilder init --domain rtcontainer.io
kubebuilder create api --group rt --version v1 --kind RTContainer
```

## 2、添加类型

> api/v1/rtcontainer_types.go

```
type RootAgent struct {
	IP   string `json:"ip,omitempty"`
	Port string `json:"port,omitempty"`
	NFS  string `json:"nfs,omitempty"`
}

type RTContainerSpec struct {
	RootAgent []RootAgent `json:"rootagent,omitempty"`
	Action    string      `json:"action,omitempty"`
	Image     string      `json:"image,omitempty"`
	Name      string      `json:"name,omitempty"`
	Replicas  *int32      `json:"replicas,omitempty"`
}
```

## 3、编写controller

> delete ？

```
// +kubebuilder:rbac:groups=crd.crd.RTContainer.io,resources=RTContainers,verbs=get;list;watch;create;update;patch;delete
// +kubebuilder:rbac:groups=crd.crd.RTContainer.io,resources=RTContainers/status,verbs=get;update;patch
// +kubebuilder:rbac:groups=crd.crd.RTContainer.io,resources=RTContainers/finalizers,verbs=update
```

> add

```
	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	rtv1 "rtcontainer/api/v1"

func (r *RTContainerReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	_ = log.FromContext(ctx)

	// TODO(user): your logic here

	RTContainer := &rtv1.RTContainer{}
	if err := r.Client.Get(ctx, req.NamespacedName, RTContainer); err != nil {
		if errors.IsNotFound(err) {
			log.Log.Info("RTContainer resource not found, skipping reconcile")
			return ctrl.Result{}, nil
		}
		return ctrl.Result{}, nil
	}

	//资源标记为删除
	if RTContainer.DeletionTimestamp != nil {
		log.Log.Info("RTContainer in deleting", "name", req.String())
		return ctrl.Result{}, nil
	}

	if err := r.CreateDeployRTContainer(ctx, RTContainer); err != nil {
		log.Log.Error(err, "failed to Create rtcontainer", "name", req.String())
		return ctrl.Result{}, nil
	}

	return ctrl.Result{}, nil
}

func (r *RTContainerReconciler) CreateDeployRTContainer(ctx context.Context, obj *rtv1.RTContainer) error {
	logger := log.FromContext(ctx)

	RTContainer := obj.DeepCopy()
	name := types.NamespacedName{
		Namespace: RTContainer.Namespace,
		Name:      RTContainer.Name,
	}
	// 构造owner
	owner := []metav1.OwnerReference{
		{
			APIVersion:         RTContainer.APIVersion,
			Kind:               RTContainer.Kind,
			Name:               RTContainer.Name,
			Controller:         pointer.BoolPtr(true),
			BlockOwnerDeletion: pointer.BoolPtr(true),
			UID:                RTContainer.UID,
		},
	}
	labels := map[string]string{"app": obj.Name}
	selector := &metav1.LabelSelector{MatchLabels: labels}

	meta := metav1.ObjectMeta{
		Name:            RTContainer.Name,
		Namespace:       RTContainer.Namespace,
		Labels:          labels,
		OwnerReferences: owner,
	}
	// 获取对应deployment, 如不存在则创建
	deploy := &appsv1.Deployment{}
	if err := r.Get(ctx, name, deploy); err != nil {
		if !errors.IsNotFound(err) {
			return err
		}

		deploy = &appsv1.Deployment{
			ObjectMeta: meta,
			Spec: appsv1.DeploymentSpec{
				Replicas: RTContainer.Spec.Replicas,
				Selector: selector,
				Template: corev1.PodTemplateSpec{
					ObjectMeta: metav1.ObjectMeta{
						Labels: labels,
					},
					Spec: corev1.PodSpec{
						Containers: newContainers(RTContainer),
					},
				},
			},
		}

		if err := r.Create(ctx, deploy); err != nil {
			return err
		}
		logger.Info("Create deployment successfully.", "name", name.String())
	}
	return nil
}

func newContainers(app *rtv1.RTContainer) []corev1.Container {
	return []corev1.Container{
		{
			Name:            app.Name,
			Image:           app.Spec.Image,
			ImagePullPolicy: corev1.PullIfNotPresent,
		},
	}
}
```

## 4、编译CRD，更新字段

```
make manifests
```

> cat config/crd/bases/rt.rtcontainer.io_rtcontainers.yaml 

## 5、安装kustomize

```
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
```

```
mv kustomize bin/
```

## 6、安装、运行

```
make install
make run
```

## 7、运行实例

```
apiVersion: rt.rtcontainer.io/v1
kind: RTContainer
metadata:
  name: rt1
spec:
  rootagent:
  -  ip: "192.168.2.2"
     port: "8888"
     nfs: "/mount/fatfs_C/nfs_root"
  action: "Create"
  image: vm1:1.0
  replicas: 1
```

