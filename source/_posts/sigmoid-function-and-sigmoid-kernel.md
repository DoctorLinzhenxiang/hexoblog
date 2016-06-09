---
title: sigmoid function 和 sigmoid kernel
date: 2016-05-03 22:31:56
categories: [Mathematics]
tag: [machine learning, mathematics]
description: 两者之间的联系与区别
---

今天在跟老师讨论多核学习问题时，遇到sigmoid kernel，发现自己记忆的与老师说的好像不一样，然后查了一下，发现原来在形式上是有两种表示形式的，但是本质是相同的。

# sigmoid function
sigmoid function : $$sigmoid(x) = \frac{1}{1+e^{-x}}$$
sigmoid function在神经网络中是一种典型的神经元激活函数,图形为：
![sigmoid function](/img/blog/sigmoid/sigmoidfunction.jpg)

# sigmoid kernel
sigmoid kernel : $$k(x, y) = \tanh(\alpha x^Ty + c)$$
其中$\tanh(x)$是双曲正切函数
$$\tanh(x) = \frac{e^x - e^{-x}}{e^x + e^{-x}} = \frac{1}{1+e^{-2x}} - \frac{1}{e^{2x} + 1}$$
图形为：
![tanh function](/img/blog/sigmoid/tanhfunction.jpg)

可以看到两者的曲线趋势都是差不多的，s型函数，只是值域不同。那么我们再来将tanh拆开画图看看效果：
$$\frac{1}{1+e^{-2x}}$$
![tanh function 1](/img/blog/sigmoid/tanhfunction1.jpg)
$$- \frac{1}{e^{2x} + 1}$$
![tanh function 2](/img/blog/sigmoid/tanhfunction2.jpg)

我们可以看到这两个的图形趋势也是相同的，同样值域不一样，而且前者值域是$[0, 1]$,后者的值域是$[-1, 0]$，那么两者相加正好实现了原双曲正切的值域范围。
那么，我们平时所用的sigmoid函数只是取了前半部分。


