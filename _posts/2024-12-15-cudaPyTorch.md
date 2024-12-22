---
layout:     post
title:      "用GPU加速PyTorch模型训练"
subtitle:   "通过CUDA来加速PyTorch深度学习模型的训练"
date:       2024-12-15
author:     "Byolio"
header-img: "img/post-cudaPyTorch.jpg"
catalog: true
tags:
    - python
    - PyTorch
    - cuda
---
> 本文介绍如何通过CUDA来加速PyTorch深度学习模型的训练(本文需要有CUDA的NVIDIA GPU显卡, 且CUDA驱动版本要与PyTorch版本相兼容)

## 什么是CUDA？
CUDA(Compute Unified Device Architecture)是由NVIDIA公司开发的一种并行计算平台和编程模型，允许开发者利用NVIDIA GPU的强大计算能力来加速各种高性能计算任务。它的核心思想是将计算密集型任务分散到GPU的多个核心上并行处理，从而显著提升计算效率。CUDA的关键特性包括：
1. 高并行性：GPU拥有成千上万个计算核心，可以同时处理大量数据，而传统的CPU核心数较少。
2. 灵活编程：CUDA 提供了一个与 C/C++ 类似的编程接口，允许开发者直接控制GPU的并行计算。
3. 广泛支持：CUDA 支持深度学习框架（如 PyTorch、TensorFlow）、科学计算库(如 cuBLAS、cuFFT)和图像处理库（如OpenCV）。

因此，通过使用CUDA，开发者可以利用GPU的TensorCore来并行计算能力加速深度学习模型的训练和推理过程，从而提高计算效率和模型训练速度。

## 配置所需的环境
我之前写过对应的[文章](https://byolio.top/2024/11/30/torch/), 这里不过多赘述。

## 使用CUDA加速所需的库
在使用CUDA加速PyTorch模型训练时，需要使用以下库：
1. torch: PyTorch的核心库，用于构建和训练深度学习模型。
```python
import torch
```
2. torch.cuda: 提供了与CUDA相关的功能和操作。
```python
import torch.cuda
```
## CUDA的相关信息与检测函数(只列出比较常用的函数)
这些函数主要在torch.cuda中, 可以通过以下方式调用:
1. 查看CUDA是否可用:
```python
print(torch.cuda.is_available())
```
2. 获取当前可用的GPU数量:
```python
print(torch.cuda.device_count())
```
3. 获取当前默认GPU的设备编号:
```python
print(torch.cuda.current_device())
```
4. 获取对应GPU设备编号的名称:
```python
print(torch.cuda.get_device_name(0))  # 0为GPU设备编号
```
5. 查看设备显存分配情况(需要有张量被分配到GPU时才能准确查看):
```python
print(torch.cuda.memory_allocated(0))  # 0号设备已分配的显存 (bytes)
print(torch.cuda.memory_reserved(0))   # 0号设备保留的显存 (bytes)
```
6. 查看设备的最大显存:
```python
print(torch.cuda.max_memory_allocated(0))  # 0号设备已分配的最大显存 (bytes)
print(torch.cuda.max_memory_reserved(0))   # 0号设备保留的最大显存 (bytes)
```
7. 对设备进行管理:
```python
torch.cuda.set_device(0)                  # 设置默认设备为0号设备
torch.cuda.empty_cache()                  # 清空未使用的缓存
torch.cuda.reset_max_memory_allocated(0)  # 重置0号设备最大分配内存统计
torch.cuda.reset_max_memory_reserved(0)   # 重置0号设备最大保留内存统计
```

## CUDA加速模型训练方式和方法
加速前需要确保你已经建立好对应的模型,  具体可以参考之前的[文章](https://byolio.top/2024/12/14/PyTorchModel/), 这里不过多赘述。 \
PyTorch中使用CUDA加速模型训练主要是将以下几个东西移动到GPU上:
1. 模型
2. 数据, 标签
3. 损失函数

具体的使用如下(这边使用的是to()函数的方法, 灵活性更高):
```python
# 定义模型
model = ByolioModel()
# 将模型移动到GPU上
model = model.to('cuda')
# 定义损失函数
loss_fn = nn.CrossEntropyLoss()
# 将损失函数移动到GPU上
loss_fn = loss_fn.to('cuda')
# 将数据和标签移动到GPU上(使用Dataloader进行加载data后)
imgs, targets = imgs.to('cuda'), targets.to('cuda')
```
## CUDA的进阶使用
1. 自动选择设备(增强代码的可移植性和灵活性):
```python
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = model.to(device)
```
2. 在PyTorch的DataLoader调用pin_memory=True来将数据加载到固定内存上:
固定内存（Pinned/Locked Memory）：一种特殊类型的内存，被锁定在物理内存中，操作系统不会将其交换到磁盘，因此允许更快地将数据传输到 GPU（通过 DMA 通道直接传输）。
3. PyTorch中使用自动混合精度训练（AMP, Automatic Mixed Precision）, 并进行梯度缩放, 更新优化器参数, 更新缩放因子
4. 可以适当的进行显存预热

## 如何进行多卡训练
这有两种方法:
1. DataParallel(DP)
```python
import torch.nn as nn
model = nn.DataParallel(model).cuda()   # 自动分发到多个 GPU
```
这种方法比较简单, 适合快速验证和小型任务, 但是在大型任务中可能会遇到内存不足的问题。
2. DistributedDataParallel(DDP)
这个方法需要使用多进程来进行训练, 比较复杂, 因为没有主 GPU 负载瓶颈，性能更优, 因此常被用于在大型任务中。 \
因为其涉及到初始化进程组, 设置配置GPU, 清理分布式环境等操作, 因此我就不在此讲述了。

## 总结
以上就是使用CUDA加速PyTorch模型训练的方法, 希望这篇文章能够帮助到你。
