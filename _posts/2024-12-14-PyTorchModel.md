---
layout:     post
title:      "PyTorch建立AI模型需要那些Module"
subtitle:   "让我们快速了解PyTorch建立一个AI模型需要那些Module"
date:       2024-12-14
author:     "Byolio"
header-img: "img/Post-PyTorchModel.jpg"
catalog: true
tags:
    - AI
    - PyTorch
---
> 本文旨在介绍PyTorch高效建立一个AI模型所需的Module, 不涉及数据的预处理, 模型训练等, 只涉及到模型的建立, 作为一个快速了解PyTorch建立一个AI模型所需的Module的文章。

## 用PyTorch建立一个AI模型所需的环境
可以查看我之前写的[文章](https://byolio.top/2024/11/30/torch/), 这边就不在花时间去赘述了。

## 用PyTorch建立一个AI模型所需的库(仅介绍了一些常用的库)
### 1. torch
torch是PyTorch的核心库，它提供了用于构建和训练神经网络的各种功能。torch库包含了许多用于构建神经网络的类和函数，包括张量、自动求导、神经网络模块、优化器等。
```python
import torch
```
### 2. torch.nn
torch.nn是PyTorch中用于构建神经网络的模型骨架。它提供了许多用于构建神经网络的类和函数，包括线性层、卷积层、激活函数、损失函数等。
```python
import torch.nn as nn
```
### 3. torch.optim
torch.optim是PyTorch中用于神经网络的优化器模块。它提供了许多用于优化神经网络用于反向创波的类和函数，包括Adam优化器、SGD优化器等。
```python
import torch.optim as optim
```

## 用PyTorch建立一个AI模型所需的Module(仅介绍了一些常用的Module和其对应参数)
了解完这些基础要用的库以后, 我们就可以开始了解建立一个简单所需的Module了。
### nn.Module
首先是我们最重要模型骨架: nn.Module。 \
nn.Module是PyTorch中用于构建神经网络的基类和骨架。它被用于定义神经网络的结构和参数。在nn.Module的子类中，我们可以定义神经网络的层、参数和前向传播的过程。
```python
import torch.nn as nn
class ByolioModel(nn.Module):
    def __init__(self):
        super(MyModel, self).__init__()
        # 定义模型的层, 在下面的例子中, 定义了一个线性层
        self.fc = nn.Linear(10, 1)
    def forward(self, x):
        # 定义前向传播的过程
        x = self.fc(x)
        return x
```
如何调用:
```python
model = ByolioModel()
outputs = model(inputs)         # 通过__call__方法调用模型
```
### 卷积层（Convolutional Layer）:
#### nn.Conv2d
nn.Conv2d是PyTorch中用于定义二维卷积层的类。它接受输入通道数、输出通道数、卷积核大小、步长和填充等参数，并在内部创建一个卷积核和一个偏置向量。在前向传播的过程中，它将输入与卷积核进行卷积运算，并加上偏置向量，然后通过激活函数进行非线性变换。
```python
conv2d = nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True)
```
参数:
* in_channels: 输入通道的数量
* out_channels: 输出通道的数量
* kernel_size: 卷积核的大小
* stride: 卷积的步长
* padding: 填充的大小
* dilation: 卷积核的膨胀率
* groups: 分组卷积的组数
* bias: 是否使用偏置向量 (默认为True)
### 全连接层（Fully Connected Layer）:
#### nn.Linear
nn.Linear是PyTorch中用于定义全连接层的类。它接受输入和输出的维度作为参数，并在内部创建一个权重矩阵和一个偏置向量。在前向传播的过程中，它将输入与权重矩阵相乘并加上偏置向量，然后通过激活函数进行非线性变换。
```python
linear = nn.Linear(in_features, out_features, bias=True)
```
参数:
* in_features: 输入特征的维度
* out_features: 输出特征的维度
* bias: 是否使用偏置向量 (默认为True)
### 激活层（Activation Layer）:
#### nn.ReLU
nn.ReLU是PyTorch中用于定义ReLU激活函数的类。它接受输入作为参数，并在内部计算ReLU函数的值。在前向传播的过程中，它将输入与ReLU函数进行运算，然后通过激活函数进行非线性变换。
```python
relu = nn.ReLU(inplace=False)
```
参数:
* inplace: 直接替换输入张量的值 (默认为False)
#### nn.Softmax
nn.Softmax是PyTorch中用于定义Softmax激活函数的类。它接受输入作为参数，并在内部计算Softmax函数的值。在前向传播的过程中，它将输入与Softmax函数进行运算，然后通过激活函数进行非线性变换。
```python
softmax = nn.Softmax(dim=None)
```
参数:
* dim: 计算Softmax的处理维度 (默认为None)
#### nn.Sigmoid
nn.Sigmoid是PyTorch中用于定义Sigmoid激活函数的类。它接受输入作为参数，并在内部计算Sigmoid函数的值。在前向传播的过程中，它将输入与Sigmoid函数进行运算，然后通过激活函数进行非线性变换。
```python
sigmoid = nn.Sigmoid()
```
### 池化层（Pooling Layer）:
#### nn.MaxPool2d
nn.MaxPool2d是PyTorch中用于定义最大池化层的类。它接受池化窗口的大小、步长和填充等参数，并在内部计算最大池化操作。在前向传播的过程中，它将输入与池化窗口进行运算，然后通过池化操作进行下采样。
```python
maxpool2d = nn.MaxPool2d(kernel_size, stride=None, padding=0, dilation=1, return_indices=False, ceil_mode=False)
```
参数:
* kernel_size: 池化窗口的大小
* stride: 池化操作的步长 (默认为kernel_size)
* padding: 填充的大小
* dilation: 池化操作的膨胀率
* return_indices: 是否返回池化操作的索引 (默认为False)
* ceil_mode: 是否使用向上取整的方式进行池化操作 (默认为False)
### 批归一化层（Batch Normalization Layer）:
#### nn.BatchNorm2d (训练时请将模型设置为训练模式, 测试时请将模型设置为评估模式)
nn.BatchNorm2d是PyTorch中用于定义批归一化层的类。它接受输入通道数作为参数，并在内部计算批归一化操作。在前向传播的过程中，它将输入与批归一化操作进行运算，然后通过批归一化操作进行归一化处理。
```python
batchnorm2d = nn.BatchNorm2d(num_features, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
```
参数:
* num_features: 输入通道的数量
* eps: 用于数值稳定性的小值
* momentum: 用于计算运行平均值和方差的动量
* affine: 是否使用可学习的仿射变换 (默认为True)
* track_running_stats: 是否跟踪运行平均值和方差 (默认为True)
### 展平层（Flatten Layer）:
#### nn.Flatten
nn.Flatten是PyTorch中用于定义展平层的类。它接受输入的维度作为参数，并在内部将输入展平为一维向量。在前向传播的过程中，它将输入展平为一维向量，然后通过展平操作进行下采样。
```python
flatten = nn.Flatten(start_dim=1, end_dim=-1)
```
参数:
* start_dim: 展平的起始维度
* end_dim: 展平的结束维度 (默认为-1), 展平到最后一个维度
### 损失函数（Loss Function）:
#### nn.CrossEntropyLoss
nn.CrossEntropyLoss是PyTorch中用于定义交叉熵损失函数的类。它接受输入和目标标签作为参数，并在内部计算交叉熵损失。在前向传播的过程中，它将输入与目标标签进行运算，然后通过损失函数进行损失计算。
```python
crossentropyloss = nn.CrossEntropyLoss(weight=None, size_average=None, ignore_index=-100, reduce=None, reduction='mean')
```
参数:
* weight: 每个类别的权重 (默认为None)
* size_average: 是否计算平均损失 (默认为None)
* ignore_index: 忽略的目标标签索引 (默认为-100)
* reduce: 是否对损失进行降维 (默认为None)
* reduction: 损失的降维方式 (默认为'mean')
#### nn.MSELoss
nn.MSELoss是PyTorch中用于定义均方误差损失函数的类。它接受输入和目标标签作为参数，并在内部计算均方误差损失。在前向传播的过程中，它将输入与目标标签进行运算，然后通过损失函数进行损失计算。
```python
mse = nn.MSELoss(size_average=None, reduce=None, reduction='mean')
```
参数:
* size_average: 是否计算平均损失 (默认为None)
* reduce: 是否对损失进行降维 (默认为None)
* reduction: 损失的降维方式 (默认为'mean')
### 优化器（Optimizer）:
#### torch.optim.SGD
torch.optim.SGD是PyTorch中用于定义随机梯度下降优化器的类。它接受模型参数和学习率等超参数，并根据随机梯度下降的规则更新模型参数。在反向传播之后，优化器利用模型参数的梯度信息来执行参数更新，从而最小化损失函数。
```python
optimizer = torch.optim.SGD(model.parameters(), lr=0.01, momentum=0.9)
```
参数:
* model.parameters(): 模型的参数
* lr: 学习率
* momentum: 动量 (默认为0)

## 如何使用这些Module(不包含数据的预处理, 模型训练等, 只涉及到模型的建立)
1. nn.Module的子类中定义的部分及其顺序:
    1. 可循环部分(可以删减其中不需要的层, 注意进行调整参数): \
        卷积层 + 激活层 + 池化层 + 批归一化层
    2. 不可循环部分:
        全连接层 + 激活层
2. 不在nn.Module的子类中定义的部分: \
    损失函数 + 优化器 \
    其使用步骤为 (可以进行优化, 以下为不优化版本):
    1. 定义:
    ```python
    # 损失函数(以交叉熵为例)
    loss_fn = nn.CrossEntropyLoss()
    # 优化器(以SGD为例)
    optimizer = torch.optim.SGD(model.parameters(), lr=0.01, momentum=0.9)
    ```
    2. 使用(在调用前向传播函数后使用):
    ```python
    # 损失函数
    loss = loss_fn(outputs, labels)
    # 优化器
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    ```
## 例子(此例子用于指示模型建立顺序, 训练不同模型时其Module会有所不同):

```python

class ByolioModel(nn.Module):
    def __init__(self):
        super(MyModel, self).__init__()
        # 使用序列化模型定义
        self.model = nn.Sequential(
        nn.Conv2d(1, 10, kernel_size=3, stride=1, padding=1),
        nn.MaxPool2d(2),
        nn.BatchNorm2d(10),
        nn.Flatten(),
        nn.Linear(10, 10),
        nn.Linear(10, 1),
        nn.Sigmoid()
        )
    def forward(self, x):
        # 定义前向传播的过程
        x = self.model(x)
        return x

if __name__ == '__main__':  
    # 定义模型
    model = ByolioModel()
    loss_fn = nn.CrossEntropyLoss()
    optimizer = torch.optim.SGD(model.parameters(), lr=0.01, momentum=0.9)

```

## 总结
以上就是PyTorch建立一个AI模型所需的Module, 希望这篇文章能够帮助到你。
