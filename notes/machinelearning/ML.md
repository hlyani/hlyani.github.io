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





* torchrun作为PyTorch官方提供的分布式训练工具。torchrun通过简单的命令行接口，使得用户可以轻松地启动多进程训练，极大地降低了入门门槛。

* accelerate则是Hugging Face推出的一款分布式训练工具，特别适合使用Hugging Face生态系统的用户。accelerate支持多种后端，包括PyTorch、TensorFlow等，具有很高的灵活性和可扩展性。
* deepspeed是由微软开发的一款高性能分布式训练库，特别适合对性能和训练规模有更高要求的用户。它提供了丰富的优化功能，如混合精度训练、梯度累积、ZeRO优化等，能够显著提高训练速度和内存利用率。deepspeed在大规模模型训练中表现出色，被广泛应用于诸如BERT、GPT-3等大型语言模型的训练。
* Megatron则是由NVIDIA开发的一款专为大规模模型训练设计的分布式训练框架。它通过模型并行和数据并行相结合的方式，实现了高效的分布式训练。Megatron在处理超大规模模型时表现出色，能够充分利用多GPU和多节点的计算资源，显著缩短训练时间。
  * **ZeRO (Zero Redundancy Optimizer)** 零冗余优化器 (Zero Redundancy Optimizer，简称ZeRO，是微软DeepSpeed库的核心)。是一种用于大规模分布式深度学习模型训练的优化方法，由微软的 DeepSpeed 提出。其主要目标是通过**优化显存利用率**，实现超大规模模型（如数百亿或数万亿参数）的高效训练。ZeRO 通过减少冗余的数据存储和计算需求，使得在多 GPU 系统上高效训练成为可能。



大模型单位：

* billion: 十亿
* million: 百万



“10b”、“13b”、"70b"等术语通常指的是大型神经网络模型的参数数量。其中的 “b” 代表 “billion”，也就是十亿。表示模型中的参数量，每个参数用来存储模型的权重和偏差等信息。例如：

* “10b” 意味着模型有大约 100 亿个参数。
* “13b” 意味着模型有大约 130 亿个参数。
* “70b” 意味着模型有大约 700 亿个参数。



**1.模型权重**

模型权重是模型参数中的一部分，通常是指神经网络中连接权重（weights）。这些权重决定了输入特征与网络层之间的连接强度，以及在前向传播过程中特征的传递方式。所以模型

**2.梯度**

在训练过程中，计算梯度用于更新模型参数。梯度与模型参数的维度相同。

**3.优化器参数**

一些优化算法（如带有动量的优化器）需要保存一些状态信息，以便在每次更新时进行调整。这些状态信息也会占用一定的显存。

**比如：**

- 采用 AdamW 优化器：每个参数占用8个字节，需要维护两个状态。意味着优化器所使用的显存量是模型权重的 2 倍；
- 采用 经过 bitsandbytes 优化的 AdamW 优化器：每个参数占用2个字节，相当于权重的一半；
- 采用 SGD 优化器：占用显存和模型权重一样。

**4.输入数据和标签**

训练模型需要将输入数据和相应的标签加载到显存中。这些数据的大小取决于每个批次的样本数量以及每个样本的维度。

**5.中间计算**

在前向传播和反向传播过程中，可能需要存储一些中间计算结果，例如激活函数的输出、损失值等。

**6.临时缓冲区**

在计算过程中，可能需要一些临时缓冲区来存储临时数据，例如中间梯度计算结果等。减少中间变量也可以节省显存，这就体现出函数式编程语言的优势了。

**7.硬件和依赖库的开销**

显卡或其他硬件设备以及使用的深度学习框架在进行计算时也会占用一些显存。



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

