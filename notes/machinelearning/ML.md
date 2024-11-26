# ML

# 一、ddp、dp、tp

## 1、Data Parallelism (DP)

​	DP 将整个模型复制到多个 GPU 上，并将训练数据分块，分配给每个 GPU。这些 GPU 使用相同的模型副本进行前向和后向计算，最后将梯度聚合并更新主模型参数。



​	数据并行模式会在每个worker之上复制一份模型，这样每个worker都有一个完整模型的副本。输入数据集是分片的，一个训练的小批量数据将在多个worker之间分割；worker定期汇总它们的梯度，以确保所有worker看到一个一致的权重版本。对于无法放进单个worker的大型模型，人们可以在模型之中较小的分片上使用数据并行。

数据并行扩展通常效果很好，但有两个限制：

1. 超过某一个点之后，每个GPU的batch size变得太小，这降低了GPU的利用率，增加了通信成本；
2. 可使用的最大设备数就是batch size，限制了可用于训练的加速器数量。

## 2、Distributed Data Parallel (DDP)

​	DDP 是一种分布式数据并行实现，通常是 PyTorch 中的 `torch.nn.parallel.DistributedDataParallel`。与普通的 DP 类似，DDP 将数据分布到多个 GPU 上，每个 GPU 上都有模型副本并进行训练。

## 3、Tensor Parallelism (TP)

​	TP 将一个模型的权重矩阵（如神经网络层）拆分成多个块，分布到不同的 GPU 上。每个 GPU 只负责处理部分张量的计算，输出的部分结果再合并在一起。

​	使用一些内存管理技术，如激活检查点（activation checkpointing）来克服数据并行的这种限制，也会使用模型并行来对模型进行分区来解决这两个挑战，使得权重及其关联的优化器状态不需要同时驻留在处理器上。

​	模型并行模式会让一个模型的内存和计算分布在多个worker之间，以此来解决一个模型在一张卡上无法容纳的问题，其解决方法是把模型放到多个设备之上。

模型并行分为两种：流水线并行和张量并行，就是把模型切分的方式。

1. 流水线并行（pipeline model parallel）是把模型不同的层放到不同设备之上，比如前面几层放到一个设备之上，中间几层放到另外一个设备上，最后几层放到第三个设备之上。
2. 张量并行则是层内分割，把某一个层做切分，放置到不同设备之上，也可以理解为把矩阵运算分配到不同的设备之上，比如把某个矩阵乘法切分成为多个矩阵乘法放到不同设备之上



# 二、概念

**Epoch**： 所有的样本数据都输入到模型中，称为一个`epoch`
**Iteration**： 一个Batch的样本输入到模型中，称为一个`Iteration`
**Batchsize**： 一个批次的大小，一个`Epoch=Batchsize*Iteration`





# IB

```
ibstat
ibstatus
ibv_devinfo
ibv_devices
rdma link
```

```
mlnx_perf -i ib0 | grep rdma
```

```
  resources:
    requests:
      rdma/rdma_shared_device_a: 1
    limits:
      rdma/rdma_shared_device_a: 1
  securityContext:
    capabilities:
      add:
      - IPC_LOCK
```



# GPU

```
nvidia-smi pmon
```

```
nvidia-smi -r
```





```
docker run -it --rm \
  -v /hl/megatron-deepspeed-llama2/megatron-deepspeed-llama2:/root/megatron-deepspeed-llama2 \
  -w /root/megatron-deepspeed-llama2 \
  -v /opt/hyhal:/opt/hyhal \
  --device=/dev/kfd \
  --device=/dev/dri \
  --security-opt seccomp=unconfined \
  --cap-add=SYS_PTRACE \
  --shm-size=16G \
  --net=host \
  megatron-deepspeed-llama2 \
  bash
```



# pytorch

```
python -c  "import torch; print(torch.cuda.is_available()); print(torch.cuda.device_count())"
```



NCCL

```
python -c "import torch.distributed as dist; print(dist.is_nccl_available())"
```

