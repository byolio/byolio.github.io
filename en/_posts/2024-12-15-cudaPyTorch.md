---
layout: post
title: Accelerating PyTorch Model Training with GPU
subtitle: 通过CUDA来加速PyTorch深度学习模型的训练
date: 2024-12-15T00:00:00.000Z
author: Byolio
header-img: img/post-cudaPyTorch.jpg
catalog: true
tags:
  - python
  - PyTorch
  - cuda
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2024-12-15-cudaPyTorch/) | [English](https://www.byolio.blog/en/en/_posts/2024-12-15-cudaPyTorch/)

> This article introduces how to accelerate PyTorch deep learning model training using CUDA (requires an NVIDIA GPU with CUDA support, and the CUDA driver version must be compatible with the PyTorch version)

## What is CUDA?
CUDA (Compute Unified Device Architecture) is a parallel computing platform and programming model developed by NVIDIA, allowing developers to leverage the powerful computing capabilities of NVIDIA GPUs to accelerate various high-performance computing tasks. Its core idea is to distribute compute-intensive tasks across multiple GPU cores for parallel processing, thereby significantly improving computational efficiency. Key features of CUDA include:
1. High parallelism: GPUs have thousands of computing cores that can process large amounts of data simultaneously, whereas traditional CPUs have fewer cores.
2. Flexible programming: CUDA provides a programming interface similar to C/C++, allowing developers to directly control GPU parallel computing.
3. Broad support: CUDA supports deep learning frameworks (such as PyTorch, TensorFlow), scientific computing libraries (such as cuBLAS, cuFFT), and image processing libraries (such as OpenCV).

Therefore, by using CUDA, developers can leverage the parallel computing capabilities of GPU TensorCores to accelerate the training and inference processes of deep learning models, thereby improving computational efficiency and model training speed.

## Configuring the Required Environment
I have previously written a corresponding [article](https://byolio.top/2024/11/30/torch/), so I won't go into detail here.

## Libraries Required for CUDA Acceleration
When using CUDA to accelerate PyTorch model training, the following libraries are needed:
1. torch: The core library of PyTorch, used for building and training deep learning models.
```python
import torch
```
2. torch.cuda: Provides CUDA-related functions and operations.
```python
import torch.cuda
```
## CUDA Related Information and Detection Functions (only commonly used functions are listed)
These functions are mainly in torch.cuda and can be called as follows:
1. Check if CUDA is available:
```python
print(torch.cuda.is_available())
```
2. Get the number of available GPUs:
```python
print(torch.cuda.device_count())
```
3. Get the device index of the current default GPU:
```python
print(torch.cuda.current_device())
```
4. Get the name of the corresponding GPU device index:
```python
print(torch.cuda.get_device_name(0))  # 0 is the GPU device index
```
5. Check device memory allocation (requires tensors allocated to the GPU for accurate viewing):
```python
print(torch.cuda.memory_allocated(0))  # Allocated memory on device 0 (bytes)
print(torch.cuda.memory_reserved(0))   # Reserved memory on device 0 (bytes)
```
6. Check the maximum memory of the device:
```python
print(torch.cuda.max_memory_allocated(0))  # Maximum allocated memory on device 0 (bytes)
print(torch.cuda.max_memory_reserved(0))   # Maximum reserved memory on device 0 (bytes)
```
7. Device management:
```python
torch.cuda.set_device(0)                  # Set the default device to device 0
torch.cuda.empty_cache()                  # Clear unused cache
torch.cuda.reset_max_memory_allocated(0)  # Reset maximum allocated memory statistics for device 0
torch.cuda.reset_max_memory_reserved(0)   # Reset maximum reserved memory statistics for device 0
```

## Methods and Approaches for CUDA-Accelerated Model Training
Before acceleration, ensure that you have built the corresponding model. For details, refer to the previous [article](https://byolio.top/2024/12/14/PyTorchModel/), so I won't elaborate here. \
Using CUDA to accelerate PyTorch model training mainly involves moving the following items to the GPU:
1. Model
2. Data and labels
3. Loss function

The specific usage is as follows (using the to() function method, which offers higher flexibility):
```python
# Define the model
model = ByolioModel()
# Move the model to the GPU
model = model.to('cuda')
# Define the loss function
loss_fn = nn.CrossEntropyLoss()
# Move the loss function to the GPU
loss_fn = loss_fn.to('cuda')
# Move data and labels to the GPU (after loading data with Dataloader)
imgs, targets = imgs.to('cuda'), targets.to('cuda')
```
## Advanced Usage of CUDA
1. Automatic device selection (enhances code portability and flexibility):
```python
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = model.to(device)
```
2. Use pin_memory=True in PyTorch's DataLoader to load data into pinned memory:
Pinned/Locked Memory: A special type of memory that is locked in physical memory, preventing the operating system from swapping it to disk, thus allowing faster data transfer to the GPU (direct transfer via DMA channel).
3. Use Automatic Mixed Precision (AMP) training in PyTorch, with gradient scaling, optimizer parameter updates, and scaling factor updates.
4. Optionally perform memory warm-up.

## How to Perform Multi-GPU Training
There are two methods:
1. DataParallel (DP)
```python
import torch.nn as nn
model = nn.DataParallel(model).cuda()   # Automatically distribute to multiple GPUs
```
This method is simple and suitable for quick validation and small tasks, but may encounter memory issues in large tasks.
2. DistributedDataParallel (DDP)
This method requires multi-process training and is more complex, but because there is no main GPU load bottleneck, performance is better, so it is often used in large tasks. \
Since it involves initializing process groups, configuring GPU settings, cleaning up the distributed environment, etc., I will not discuss it here.

## Summary
The above are the methods for accelerating PyTorch model training using CUDA. I hope this article helps you.
