---
title: "CS231n:Optimization: Stochastic Gradient Descent"
date: 2016-08-15 20:20:45
categories: [Machine Learning]
tag: [Machine Learning, CS231n笔记]
description: 
---

这一节主要介绍一种优化方法，随机梯度下降，用于求解前面章节提到的目标函数中的权重矩阵$W$.

# Visualizing the loss function
这里主要通过一些简单的例子来表明，我们所要求解的目标函数是一个凸问题，通过形象化的图形来体会优化的过程。同时，课程中也提到如果在神经网络中，目标函数就不再是凸的了，所以需要新的求解方法。

# Optimization
## Strategy 1: Random Search
首先限定一个最大迭代步数，然后在每一次迭代中都将随机的$W$值代入求得目标函数值，并记录下使得目标函数值取得当前最小的$W$作为最终的输出

code: 
```python
for num in xrange(1000):
    W = np.random.randn(10, 3073) * 0.0001 # generate random parameters
    loss = L(X_train, Y_train, W) # get the loss over the entire training set
    if loss < bestloss: # keep track of the best solution
        bestloss = loss
        bestW = W
    print 'in attempt %d the loss was %f, best %f' % (num, loss, bestloss)
```
## Strategy 2: Random Local Search
对上面方法的一个改进，就是不是在每次的迭代中都给$W$赋随机值，而是根据初始赋随机值进行微调，如果能够使得目标函数值下降，则记录下该微调之后的$W$，直到迭代步数达到设定的值。

code:
```python
W = np.random.randn(10, 3073) * 0.001 # generate random starting W
bestloss = float("inf")
for i in xrange(1000):
    step_size = 0.0001
    Wtry = W + np.random.randn(10, 3073) * step_size
    loss = L(Xtr_cols, Ytr, Wtry)
    if loss < bestloss:
        W = Wtry
        bestloss = loss
    print 'iter %d loss is %f' % (i, bestloss)
```
## Strategy 3: Following the gradient
显然，策略2已经很接近通用的梯度下降方法了，唯一的区别是，我们在每一步迭代中都是取使目标函数下降最大的方向进行优化的，而要想能够找到使得目标函数下降最大的方向，就需要用到梯度了。

# Computing the gradient
对于梯度的计算，课程中提供了两种方法
## Numerically with finite differences
标签：a slow, approximate，but easy way

公式：
$$\frac{d f(x)}{d x}=\lim_{x\to 0} \frac{f(x+h)-f(x)}{h}$$

code: 
```python
fx = f(x) # evaluate function value at original point
grad = np.zeros(x.shape)
h = 0.00001

# iterate over all indexes in x
it = np.nditer(x, flags=['multi_index'], op_flags=['readwrite'])
while not it.finished:

    # evaluate function at x+h
    ix = it.multi_index
    old_value = x[ix]
    x[ix] = old_value + h # increment by h
    fxh = f(x) # evalute f(x + h)
    x[ix] = old_value # restore to previous value (very important!)

    # compute the partial derivative
    grad[ix] = (fxh - fx) / h # the slope
    it.iternext() # step to next dimension
```

## Analytically with calculus
标签： a fast, exact but more error-prone way

以SVM损失函数在某个样本下的形式为例：
$$L\_i = \sum\_{j\neq y\_i} \left [\max(0, w\_j^T x\_i - w\_{y\_i}^T x\_i + \Delta) \right ]$$
计算上式对 $w\_{y\_i}$ 的梯度：
$$\nabla\_{w\_{y\_i}}L\_i = -\left (\sum\_{j\neq y\_i} \mathbb{I} (w\_j^T x\_i - w\_{y\_i}^T x\_i + \Delta > 0)\right )x\_i$$
计算上式对 $j\neq y\_i$ 的 $W\_j$ 的梯度：
$$\nabla\_{w\_j}L\_i = \mathbb{I} (w\_j^T x\_i - w\_{y\_i}^T x\_i + \Delta > 0)x\_i$$

# Gradient Descent
梯度下降方法的主要思路是，在梯度方向上，进行某一步长的移动以使得目标函数呈不增的趋势。

code:
```python
while True:
  weights_grad = evaluate_gradient(loss_fun, data, weights)
  weights += - step_size * weights_grad # perform parameter updat
```

**Mini-batch gradient descent**
当数据集非常的大时，机器资源可能不能一次性的加载进所有的数据，所以可以每次只选择其中的一小部分进行处理。

code:
```python
while True:
  data_batch = sample_training_data(data, 256) # sample 256 examples
  weights_grad = evaluate_gradient(loss_fun, data_batch, weights)
  weights += - step_size * weights_grad # perform parameter update
```

对于Mini-batch的大小，通常不用交叉验证的方式来选择，通常是根据经验提前设定的（根据计算资源而定）：$[32, 64, 128,...]$,通常选择的是2的倍数，因为这符合计算机的二进制计数原理。

**Stochastic Gradient Descent (SGD)**
当Mini-batch gradient descent取极限时，即每次只取所有样本中的一个样例用于训练。但是由于在实践中发现，计算100个样本的梯度往往要优于计算某一个样本100次所得到的权重矩阵，所以Mini-batch gradient descent较为常用，而SGD有时也指Mini-batch gradient descent。

# reference
1. [CS231n: Optimization: Stochastic Gradient Descent](http://cs231n.github.io/optimization-1)
