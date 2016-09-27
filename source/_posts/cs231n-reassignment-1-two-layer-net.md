---
title: "CS231n:任务1中two-layer net的梯度推导过程"
date: 2016-08-20 21:22:45
categories: [Machine Learning]
tag: [Machine Learning, CS231n笔记]
description: 
---

在任务1中，要实现的是一个2层网络的训练及预测，两层网络的结构如下：

Inputs $\to$ fully connected layer $\to$ ReLU $\to$ fully connected layer $\to$ Softmax loss + $L_2$ regularization

在[博客](http://www.jianshu.com/p/004c99623104)中给出了一个类似结构的三层网络的推导过程：

![compute the gradient](/img/blog/deeplearning/compute_gradient.jpg)


其中，对Softmax loss的求导公式如下：

![Derivative of softmax loss function](/img/blog/deeplearning/derivative_of_softmax_loss_function.jpg)