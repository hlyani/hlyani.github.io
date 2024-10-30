# kubeflow

# 一、安装

```
git clone https://github.com/kubeflow/training-operator.git
cd training-operator
git checkout v1.8.1
kubectl apply -k manifests/overlays/standalone/
kubectl get pods -n kubeflow
kubectl get crd
kubectl edit -n kubeflow deployments training-operator
```

```
kubectl patch -n kubeflow deployments training-operator --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/command/1", "value": "--gang-scheduler-name=volcano"}]'

kubectl patch -n kubeflow deployments training-operator --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/command/2", "value": "--pytorch-init-container-image=easzlab.io.local:5000/alpine:3.10"}]'
```

# 二、使用

## 1、tfjob MNIST 示例

https://www.kubeflow.org/docs/components/training/user-guides/tensorflow/

```
kubectl create -f https://raw.githubusercontent.com/kubeflow/training-operator/master/examples/tensorflow/simple.yaml
```

https://github.com/kubeflow/training-operator/blob/master/examples/tensorflow/mnist_with_summaries/mnist_with_summaries.py

tf-mnist.yaml

```
apiVersion: "kubeflow.org/v1"
kind: TFJob
metadata:
  name: tfjob-simple
  namespace: kubeflow
spec:
  tfReplicaSpecs:
    Worker:
      replicas: 2
      restartPolicy: OnFailure
      template:
        spec:
          containers:
            - name: tensorflow
              image: kubeflow/tf-mnist-with-summaries:latest
              imagePullPolicy: IfNotPresent
              command:
                - "python"
                - "/var/tf_mnist/mnist_with_summaries.py"
```

```
kubectl apply -f tf-mnist.yaml
kubectl -n kubeflow logs tfjob-simple-worker-0
kubectl get tfjob -n kubeflow
kubectl delete -f tf-mnist.yaml
```

使用 gpu

tf-gpu.yaml

```
apiVersion: "kubeflow.org/v1"
kind: "TFJob"
metadata:
  name: "tf-smoke-gpu"
spec:
  tfReplicaSpecs:
    PS:
      replicas: 1
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
            - args:
                - python
                - tf_cnn_benchmarks.py
                - --batch_size=32
                - --model=resnet50
                - --variable_update=parameter_server
                - --flush_stdout=true
                - --num_gpus=1
                - --local_parameter_device=cpu
                - --device=cpu
                - --data_format=NHWC
              image: docker.io/kubeflow/tf-benchmarks-cpu:v20171202-bdab599-dirty-284af3
              name: tensorflow
              ports:
                - containerPort: 2222
                  name: tfjob-port
              resources:
                limits:
                  cpu: "1"
              workingDir: /opt/tf-benchmarks/scripts/tf_cnn_benchmarks
          restartPolicy: OnFailure
    Worker:
      replicas: 1
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
            - args:
                - python
                - tf_cnn_benchmarks.py
                - --batch_size=32
                - --model=resnet50
                - --variable_update=parameter_server
                - --flush_stdout=true
                - --num_gpus=1
                - --local_parameter_device=cpu
                - --device=gpu
                - --data_format=NHWC
              image: docker.io/kubeflow/tf-benchmarks-gpu:v20171202-bdab599-dirty-284af3
              name: tensorflow
              ports:
                - containerPort: 2222
                  name: tfjob-port
              resources:
                limits:
                  nvidia.com/gpu: 1       # GPU数量
              workingDir: /opt/tf-benchmarks/scripts/tf_cnn_benchmarks
          restartPolicy: OnFailure
```

```
kubectl apply -f tf-gpu.yaml
kubectl logs tf-smoke-gpu-worker-0
```

