---
layout: post
title: Modules Required to Build an AI Model with PyTorch
subtitle: 让我们快速了解PyTorch建立一个AI模型需要那些Module
date: 2024-12-14T00:00:00.000Z
author: Byolio
header-img: img/Post-PyTorchModel.jpg
catalog: true
tags:
  - AI
  - PyTorch
lang: en
---

> 🌐 [中文](/2024-12-14-PyTorchModel/) | [English](/en/en/_posts/2024-12-14-PyTorchModel/)

> This article aims to introduce the Modules required to efficiently build an AI model with PyTorch. It does not cover data preprocessing, model training, etc., only the construction of the model, serving as a quick guide to the Modules needed to build an AI model with PyTorch.

## Environment Required to Build an AI Model with PyTorch
You can refer to my previous [article](https://byolio.top/2024/11/30/torch/), I won't spend time repeating it here.

## Libraries Required to Build an AI Model with PyTorch (only introduces some commonly used libraries)
### 1. torch
torch is the core library of PyTorch, providing various functionalities for building and training neural networks. The torch library contains many classes and functions for building neural networks, including tensors, automatic differentiation, neural network modules, optimizers, etc.
```python
import torch
```
### 2. torch.nn
torch.nn is the model skeleton for building neural networks in PyTorch. It provides many classes and functions for building neural networks, including linear layers, convolutional layers, activation functions, loss functions, etc.
```python
import torch.nn as nn
```
### 3. torch.optim
torch.optim is the optimizer module for neural networks in PyTorch. It provides many classes and functions for optimizing neural networks during backpropagation, including Adam optimizer, SGD optimizer, etc.
```python
import torch.optim as optim
```

## Modules Required to Build an AI Model with PyTorch (only introduces some commonly used Modules and their corresponding parameters)
After understanding these basic libraries, we can start learning about the Modules needed to build a simple model.
### nn.Module
First, our most important model skeleton: nn.Module. \
nn.Module is the base class and skeleton for building neural networks in PyTorch. It is used to define the structure and parameters of a neural network. In subclasses of nn.Module, we can define the layers, parameters, and forward propagation process of the neural network.
```python
import torch.nn as nn
class ByolioModel(nn.Module):
    def __init__(self):
        super(MyModel, self).__init__()
        # Define the layers of the model, in the example below, a linear layer is defined
        self.fc = nn.Linear(10, 1)
    def forward(self, x):
        # Define the forward propagation process
        x = self.fc(x)
        return x
```
How to call:
```python
model = ByolioModel()
outputs = model(inputs)         # Call the model via the __call__ method
```
### Convolutional Layer:
#### nn.Conv2d
nn.Conv2d is a class in PyTorch used to define a 2D convolutional layer. It accepts parameters such as input channels, output channels, kernel size, stride, and padding, and internally creates a convolution kernel and a bias vector. During forward propagation, it performs convolution between the input and the kernel, adds the bias vector, and then applies a nonlinear transformation through an activation function.
```python
conv2d = nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True)
```
Parameters:
* in_channels: Number of input channels
* out_channels: Number of output channels
* kernel_size: Size of the convolution kernel
* stride: Stride of the convolution
* padding: Size of padding
* dilation: Dilation rate of the convolution kernel
* groups: Number of groups for grouped convolution
* bias: Whether to use a bias vector (default is True)
### Fully Connected Layer:
#### nn.Linear
nn.Linear is a class in PyTorch used to define a fully connected layer. It accepts the dimensions of input and output as parameters, and internally creates a weight matrix and a bias vector. During forward propagation, it multiplies the input by the weight matrix, adds the bias vector, and then applies a nonlinear transformation through an activation function.
```python
linear = nn.Linear(in_features, out_features, bias=True)
```
Parameters:
* in_features: Dimension of input features
* out_features: Dimension of output features
* bias: Whether to use a bias vector (default is True)
### Activation Layer:
#### nn.ReLU
nn.ReLU is a class in PyTorch used to define the ReLU activation function. It accepts the input as a parameter and internally computes the value of the ReLU function. During forward propagation, it applies the ReLU function to the input, performing a nonlinear transformation.
```python
relu = nn.ReLU(inplace=False)
```
Parameters:
* inplace: Whether to directly replace the input tensor's values (default is False)
#### nn.Softmax
nn.Softmax is a class in PyTorch used to define the Softmax activation function. It accepts the input as a parameter and internally computes the value of the Softmax function. During forward propagation, it applies the Softmax function to the input, performing a nonlinear transformation.
```python
softmax = nn.Softmax(dim=None)
```
Parameters:
* dim: The dimension along which Softmax is computed (default is None)
#### nn.Sigmoid
nn.Sigmoid is a class in PyTorch used to define the Sigmoid activation function. It accepts the input as a parameter and internally computes the value of the Sigmoid function. During forward propagation, it applies the Sigmoid function to the input, performing a nonlinear transformation.
```python
sigmoid = nn.Sigmoid()
```
### Pooling Layer:
#### nn.MaxPool2d
nn.MaxPool2d is a class in PyTorch used to define a max pooling layer. It accepts parameters such as pooling window size, stride, and padding, and internally computes the max pooling operation. During forward propagation, it applies the pooling operation to the input, performing downsampling.
```python
maxpool2d = nn.MaxPool2d(kernel_size, stride=None, padding=0, dilation=1, return_indices=False, ceil_mode=False)
```
Parameters:
* kernel_size: Size of the pooling window
* stride: Stride of the pooling operation (default is kernel_size)
* padding: Size of padding
* dilation: Dilation rate of the pooling operation
* return_indices: Whether to return the indices of the pooling operation (default is False)
* ceil_mode: Whether to use ceil mode for the pooling operation (default is False)
### Batch Normalization Layer:
#### nn.BatchNorm2d (set the model to training mode during training, and evaluation mode during testing)
nn.BatchNorm2d is a class in PyTorch used to define a batch normalization layer. It accepts the number of input channels as a parameter and internally computes the batch normalization operation. During forward propagation, it applies batch normalization to the input, performing normalization.
```python
batchnorm2d = nn.BatchNorm2d(num_features, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
```
Parameters:
* num_features: Number of input channels
* eps: A small value for numerical stability
* momentum: Momentum used for computing running mean and variance
* affine: Whether to use learnable affine transformations (default is True)
* track_running_stats: Whether to track running mean and variance (default is True)
### Flatten Layer:
#### nn.Flatten
nn.Flatten is a class in PyTorch used to define a flatten layer. It accepts the dimensions of the input as parameters and internally flattens the input into a one-dimensional vector. During forward propagation, it flattens the input into a one-dimensional vector, performing downsampling.
```python
flatten = nn.Flatten(start_dim=1, end_dim=-1)
```
Parameters:
* start_dim: The starting dimension for flattening
* end_dim: The ending dimension for flattening (default is -1), flattens to the last dimension
### Loss Function:
#### nn.CrossEntropyLoss
nn.CrossEntropyLoss is a class in PyTorch used to define the cross-entropy loss function. It accepts the input and target labels as parameters and internally computes the cross-entropy loss. During forward propagation, it computes the loss between the input and target labels.
```python
crossentropyloss = nn.CrossEntropyLoss(weight=None, size_average=None, ignore_index=-100, reduce=None, reduction='mean')
```
Parameters:
* weight: Weight for each class (default is None)
* size_average: Whether to compute the average loss (default is None)
* ignore_index: Index of the target label to ignore (default is -100)
* reduce: Whether to reduce the loss (default is None)
* reduction: The reduction method for the loss (default is 'mean')
#### nn.MSELoss
nn.MSELoss is a class in PyTorch used to define the mean squared error loss function. It accepts the input and target labels as parameters and internally computes the mean squared error loss. During forward propagation, it computes the loss between the input and target labels.
```python
mse = nn.MSELoss(size_average=None, reduce=None, reduction='mean')
```
Parameters:
* size_average: Whether to compute the average loss (default is None)
* reduce: Whether to reduce the loss (default is None)
* reduction: The reduction method for the loss (default is 'mean')
### Optimizer:
#### torch.optim.SGD
torch.optim.SGD is a class in PyTorch used to define the stochastic gradient descent optimizer. It accepts model parameters and hyperparameters such as learning rate, and updates model parameters according to the rules of stochastic gradient descent. After backpropagation, the optimizer uses the gradient information of the model parameters to perform parameter updates, thereby minimizing the loss function.
```python
optimizer = torch.optim.SGD(model.parameters(), lr=0.01, momentum=0.9)
```
Parameters:
* model.parameters(): Parameters of the model
* lr: Learning rate
* momentum: Momentum (default is 0)

## How to Use These Modules (does not include data preprocessing, model training, etc., only involves model construction)
1. Parts defined in the subclass of nn.Module and their order:
    1. Repeatable part (you can remove unnecessary layers, adjust parameters accordingly): \
        Convolutional Layer + Activation Layer + Pooling Layer + Batch Normalization Layer
    2. Non-repeatable part:
        Fully Connected Layer + Activation Layer
2. Parts not defined in the subclass of nn.Module: \
    Loss Function + Optimizer \
    Their usage steps are (can be optimized, the following is the unoptimized version):
    1. Definition:
    ```python
    # Loss function (using cross-entropy as an example)
    loss_fn = nn.CrossEntropyLoss()
    # Optimizer (using SGD as an example)
    optimizer = torch.optim.SGD(model.parameters(), lr=0.01, momentum=0.9)
    ```
    2. Usage (after calling the forward propagation function):
    ```python
    # Loss function
    loss = loss_fn(outputs, labels)
    # Optimizer
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    ```
## Example (this example indicates the order of model construction; the Modules may differ when training different models):

```python

class ByolioModel(nn.Module):
    def __init__(self):
        super(MyModel, self).__init__()
        # Use sequential model definition
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
        # Define the forward propagation process
        x = self.model(x)
        return x

if __name__ == '__main__':  
    # Define the model
    model = ByolioModel()
    loss_fn = nn.CrossEntropyLoss()
    optimizer = torch.optim.SGD(model.parameters(), lr=0.01, momentum=0.9)

```

## Summary
The above are the Modules required to build an AI model with PyTorch. I hope this article can help you.
