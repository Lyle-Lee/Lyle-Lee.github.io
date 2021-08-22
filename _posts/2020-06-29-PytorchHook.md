---
layout:     post
title:      Understand Pytorch Hook
subtitle:   About Pytorch Hook
date:       2020-6-29
author:     Lyle
header-img: img/post-bg-futaba.jpeg
catalog: true
tags:
    - Python
    - Pytorch
    - Deep Learning
---

>Pytorch中的Hook有何作用？

## Hook的类型

首先来看一下三种不同的Hook：

- `register_hook`，针对`Variable`对象

```python
# Python method, in Automatic Differentiation package

torch.autograd.Variable.register_hook
```


- `register_forward_hook`，针对`nn.Module`对象

```python
# Python method, in torch.nn

torch.nn.Module.register_forward_hook
```


- `register_backward_hook`，也是针对`nn.Module`对象

```python
# Python method, in torch.nn

torch.nn.Module.register_backward_hook
```

## Hook的作用

那么为什么要设计Hook呢，或者说在什么场景下我们需要使用Hook呢？

<p>举个简单的例子，<a href="https://www.codecogs.com/eqnedit.php?latex=\inline&space;{\color{Black}&space;\boldsymbol{x}&space;\in&space;\mathbb{R}^2}" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\inline&space;{\color{Black}&space;\boldsymbol{x}&space;\in&space;\mathbb{R}^2}" title="{\color{Black} \boldsymbol{x} \in \mathbb{R}^2}" /></a><a href="https://www.codecogs.com/eqnedit.php?latex=\inline&space;{\color{Black}&space;\boldsymbol{y}&space;=&space;\boldsymbol{x}&space;&plus;&space;2}" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\inline&space;{\color{Black}&space;\boldsymbol{y}&space;=&space;\boldsymbol{x}&space;&plus;&space;2}" title="{\color{Black} \boldsymbol{y} = \boldsymbol{x} + 2}" /></a><a href="https://www.codecogs.com/eqnedit.php?latex=\inline&space;{\color{Black}&space;z&space;=&space;\frac{1}{2}\boldsymbol{y}^\top\boldsymbol{y}}" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\inline&space;{\color{Black}&space;z&space;=&space;\frac{1}{2}\boldsymbol{y}^\top\boldsymbol{y}}" title="{\color{Black} z = \frac{1}{2}\boldsymbol{y}^\top\boldsymbol{y}}" /></a></p>

你想通过梯度下降法求最小值，在Pytorch里面很容易实现，你只需要：

```python
import torch
from torch.autograd import Variable

x = Variable(torch.randn(2, 1), requires_grad=True)
y = x + 2
z = torch.mean(torch.pow(y, 2))
lr = 1e-3
z.backward()
x.data -= lr * x.grad.data
```

<p>但问题是，如果想求中间变量<a href="https://www.codecogs.com/eqnedit.php?latex=\inline&space;{\color{Black}&space;\boldsymbol{y}}" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\inline&space;{\color{Black}&space;\boldsymbol{y}}" title="{\color{Black} \boldsymbol{y}}" /></a>的梯度，系统会返回错误。</p>

可以看一下原因：

```python
print(type(y.grad))
```

此时的输出为`NoneType`

这是由于Pytorch的设计初衷，为了优化模型训练时的内存开销，对于中间变量，也即是各种深度学习网络模型的中间层计算结果，一旦它们完成了自身反向传播的使命就会被释放掉。

因此，Hook就派上用场了。简而言之，`register_hook`的作用是，当反向传播时，除了完成原有的反向传播，额外多完成一些任务。你可以定义一个中间变量的Hook，将它的`grad`值打印出来，当然你也可以定义一个全局列表，将每次的`grad`值添加到里面去。

```python
import torch
from torch.autograd import Variable

grad_list = []

def print_grad(grad):
    grad_list.append(grad)

x = Variable(torch.randn(2, 1), requires_grad=True)
y = x + 2
z = torch.mean(torch.pow(y, 2))
lr = 1e-3
y.register_hook(print_grad)
z.backward()
x.data -= lr * x.grad.data
```

需要注意的是，`register_hook`作为函数接收的是一个函数，这个函数有如下的形式：

```python
hook(grad) -> Variable or None
```

也就是说，这个函数是拥有改变梯度值的威力的！

实际应用中，`register_forward_hook`和`register_backward_hook`由于对象是`nn.Module`，在深度学习中被使用的频率会更高一些，使用方法则大同小异，以`backward_hook`为例：

```python
import torch

def backward_hook(self, in_grad, out_grad):
    self.in_grad = in_grad
    self.out_grad = out_grad

x = torch.tensor([[1.], [2.]],requires_grad=True)
y = torch.tensor([[1.], [1.]])
model = torch.nn.Linear(1, 1, bias=False)
for param in model.parameters():
    torch.nn.init.constant_(param, 2)
model.register_full_backward_hook(backward_hook)
criterion = torch.nn.MSELoss(reduction='mean')
y_p = model(x)
loss = criterion(y_p, y)

print('loss   :', loss.data)
loss.backward()

print('dl/dy_p:', model.out_grad[0].data)
print('dl/dx  :', model.in_grad[0].data)
for param in model.parameters():
    print('grad   :', param.grad.data)
```

上面的代码可以输出一个单层网络模型的反向传播过程中，通过计算图生成的每一个中间变量的梯度。

如果想要观察整个模型训练的过程，可以通过一个巧妙的办法将`forward_hook`与`backward_hook`结合起来：

```python
def forward_hook(self, in_data, out_data):
    in_data = in_data[0].clone().detach()
    def backward_hook(out_grads):
        self.in_data = in_data
        self.out_grads = out_grads
    out_data.register_hook(backward_hook)
```

## 总结

当你训练一个网络，想要提取中间层的参数、或者特征图的时候，Hook就发挥它的作用了。

## 参考资料

- [Why cant I see .grad of an intermediate variable?](https://discuss.pytorch.org/t/why-cant-i-see-grad-of-an-intermediate-variable/94)
- [Extract feature maps from intermediate layers without modifying forward()](https://discuss.pytorch.org/t/extract-feature-maps-from-intermediate-layers-without-modifying-forward/1390)
