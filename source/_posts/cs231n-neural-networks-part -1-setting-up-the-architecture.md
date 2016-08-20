---
title: "CS231n:Neural Networks Part 1: Setting up the Architecture"
date: 2016-08-17 21:22:45
categories: [Machine Learning]
tag: [Machine Learning, CS231n笔记]
description: 
---


从这一节开始，课程重点：神经网络来了，本节主要讲一些神经网络中常用的损失函数，激活函数以及层次结构等。

#  Modeling one neuron
很多人都知道神经网络中基本单元是受人脑中的神经元的启发而设计的，所以，本节一开始通过人脑中的基本神经元-突触来揭示了其工作原理，以及到神经网络中基本单元的类比。

然后，介绍了几个基本的线性分类器 **Binary Softmax classifier** 和 **Binary SVM classifier**

常用的激活函数：

**sigmoid**:
函数形式： $$\sigma(x)=\frac{1}{1+e^{-x}}$$
函数图像可见[sigmoid function 和 sigmoid kernel
](http://heimingx.cn/2016/05/03/sigmoid-function-and-sigmoid-kernel/)
*优点*：
对于神经元的激活频率有着良好的解释：从完全不激活（0）到在求和后的最大频率处的完全饱和的激活（1）
*缺点*：
1. 饱和会使得梯度消失 
2. 输出不是零中心的

**tanh**
函数形式： $$\tanh(x)=\tanh(\alpha x^T y+c) = 2\sigma(2x)-1$$
具有与sigmoid一样的性能，不过也有梯度会消失的缺点

**ReLU (Rectified Linear Unit)**
函数形式:
$$f(x) = \max(0, x)$$
函数图像：
![ReLU](http://cs231n.github.io/assets/nn1/relu.jpeg)
根据函数图像易知，当 $x <= 0$ 时，函数值为0；当 $x > 0$时，函数斜率为1.

*优点*：
1. 对随机梯度下降的收敛有巨大的作用
2. 计算资源耗费少
*缺点*：
在训练时，比较脆弱，并且可能“死掉”（**这里有疑问，为什么会死掉？**）

**Leaky ReLU**
函数形式：

$$f(x)=\mathbb{I}(x<0) (\alpha x) + \mathbb{I}(x\geq 0) (x)$$

函数图像：
![Leaky ReLU](http://upload-images.jianshu.io/upload_images/2301760-576db63c7825ffcf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
根据函数图像，可以看出Leaky ReLU是对ReLU的一个改进，当 $x<=0$ 时，函数给出一个很小的负数梯度量

**PReLU** (Delving Deep into Rectifiers, Kaiming He, 2015)
把负区间上的斜率当作每个神经元中的一个参数来学习

**Maxout**
函数形式：
$$f(x) = \max(W_1^T x + b_1, W^T_2 x + b_2)$$

该方法是对ReLU和Leaky ReLU的一般化归纳
当 $W_1 = 0, b_1 = 0$ 时，就变成了ReLU
*优点*：
拥有ReLU的优点（线性操作&不饱和），而没有他的缺点（死亡的ReLU单元）
*缺点*：
每个神经元的参数数量增加了一倍，导致整体参数的数量激增。

# Neural Network architectures
1. 层的结构
神经网络模型是通过层状无环结构组成，比如全连接层结构。
其次，通常所说的几层神经网络是不连输入层的，比如下图为3层神经网络。
![3-layer NN](http://cs231n.github.io/assets/nn1/neural_net2.jpeg)

2. 前向传播
一般是先进行一个矩阵乘法，然后加上偏置并运用激活函数来实现。

3. 表达能力
给出任意连续函数 $f(x)$ 和任意 $\epsilon > 0$,均存在一个至少含1个隐层的神经网络 $g(x)$ (并且网络中有合理选择的非线性激活函数，比如Sigmoid)，对于 $\forall x$，使得 $|f(x) - g(x)| < \epsilon$.

4. 设置层的数量和尺寸
隐层单元越多，网络的表达能力越强
当出现Overfit时，应选择规则化而不是减少层数和单元数

# references
1. [CS231n:Neural Networks Part 1: Setting up the Architecture](http://cs231n.github.io/neural-networks-1/)
2. [CS231n (winter 2016) : Assignment1](http://www.jianshu.com/p/004c99623104) 这个作者自己总结的还是挺好的，值得参考，感谢博主！
3. [知乎：CS231n课程笔记翻译：神经网络笔记1上](https://zhuanlan.zhihu.com/p/21462488?refer=intelligentunit)
[知乎：CS231n课程笔记翻译：神经网络笔记1下](https://zhuanlan.zhihu.com/p/21513367?refer=intelligentunit)