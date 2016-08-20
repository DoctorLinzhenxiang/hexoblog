---
title: "CS231n:Backpropagation, Intuitions"
date: 2016-08-16 21:22:45
categories: [Machine Learning]
tag: [Machine Learning, CS231n笔记]
description: 
---

这一节主要介绍了反馈网络中的链式求导法则，非常的基础。

# Simple expressions and interpretation of the gradient
这段通过一个简单的栗子来解释梯度的意义，其实在高数中有学过，物理意义是：一个函数在某一点沿着某个方向的梯度方向便是上升最快的地方，所以，要使得目标函数值能够下降最快，便是沿着负梯度方向就好了。
课程中的解释是通过具体的数字来体现的：
函数：$$f(x, y) = xy$$
所以，该函数在 $x, y$ 方向上的梯度是 $$\frac{\partial f}{\partial x} = y;\quad \frac{\partial f}{\partial y} = x$$
当取$x=4, y=-3$时，函数$f(x, y) = -12$,且在 $x$ 上的导数是$\frac{\partial f}{\partial x} = -3$，这表明当我们给 $x$ 值增加一个很小的值，目标函数值就会下降（导数值是负的），且下降的幅度为 $3$。所以，函数在某个变量上的导数告诉了我们整个函数对于这个变量的敏感程度。
课程中还举了一个加法和max的例子，这里就不赘述了。

# Compound expressions with chain rule
主要介绍链式法则。
函数：
$$f(x, y, z) = (x + y)z$$
将这个函数拆分成两个表达式：$q = x + y;\quad f = qz$，接着，分别对这两个子表达式中变量进行求导：$\frac{\partial f}{\partial q} = z; \quad \frac{\partial f}{\partial z} = q$, $\frac{\partial q}{\partial x} = 1; \quad \frac{\partial q}{\partial y} = 1$.那么要求函数$f$对$x$的导数便易得：$\frac{\partial f}{\partial x} = \frac{\partial f}{\partial q} \frac{\partial q}{\partial x}$。
code:
```python
# set some inputs
x = -2; y = 5; z = -4

# perform the forward pass
q = x + y # q becomes 3
f = q * z # f becomes -12

# perform the backward pass (backpropagation) in reverse order:
# first backprop through f = q * z
dfdz = q # df/dz = q, so gradient on z becomes 3
dfdq = z # df/dq = z, so gradient on q becomes -4
# now backprop through q = x + y
dfdx = 1.0 * dfdq # dq/dx = 1. And the multiplication here is the chain rule!
dfdy = 1.0 * dfdq # dq/dy = 1
```

# Intuitive understanding of backpropagation
对于反馈网络的理解，课程中是通过网络中每个门(gate)来解释的，每个门在接受输入后，完成两件事：1.给出输出；2.给出关于输出的，输入的局部梯度。
所以，整个反馈网络都是通过着一些个门来互相交流的，比如他们是想要更高或是更低的输出，最终对整个函数的结果产生影响。

后面课程中有举了两个复杂一点的函数例子，这里就不搬过来了。

# Patterns in backward flow
课程介绍了三种门的特性：
**add gate**: distribute gradients equally to all its inputs
**max gate**: routes the gradients to the higher input
**multiply gate**: take the input activations, swap them and multiplies by its gradient

对样本数据预处理的理解：
一般函数中都会有 $W^T x_i$ , 由于这样乘法的形式，如果给样本数据 $x_i * 1000$, 那么权重 $W$ 的梯度就会增加1000倍，变化率是很大的了，此时，就需要更低的学习率来中和这样的影响，所以，预处理要将样本数据进行规范化，以便不再出现上面奇怪的问题。

# reference
1. [CS231n: Backpropagation, Intuitions](http://cs231n.github.io/optimization-2/)