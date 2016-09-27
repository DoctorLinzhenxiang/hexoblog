---
title: "CS231n:Linear classification: Support Vector Machine, Softmax"
date: 2016-08-15 20:20:45
categories: [Machine Learning]
tag: [Machine Learning, CS231n笔记]
description: 
---

在第一节的课程中主要介绍了KNN方法，由于knn是lazy Learning，主要有两个缺点：要记住所有的训练点；将测试点与所有训练点进行比较，为了克服这些缺点，通常会选择其他的分类方法，本节介绍了SVM和softmax classifier.

# 对线性分类器的解释
线性分类器的形式：
$$f(x_i;W,b)=Wx_i + b$$

## 结果向量 $f(x_i;W,b)$ 中值的大小

![](/img/blog/deeplearning/imagemap.jpg)

上图表示判定样例$x_i$是属于各种类别的得分情况，向量$f(x_i;W,b)$的值来看此样例小猫被判定到了狗狗的类别中，所以分类错误，需要进一步的学习。
## 权重矩阵$W$的每一行都是一个template

![](/img/blog/deeplearning/templates.jpg)

上图是通过将权重矩阵$W$中每一行reshape后画出来的图像，我们其实可以粗略的判断出一些类别，比如第二幅图是一辆偏粉红色的小汽车，表明，我们数据集中的小汽车颜色可能都偏红色，倒数第三章是有左右两个头的马儿，其实这跟训练数据集有关，可能数据集中包含了两个方向的马儿图片。
在分类时，其实是将样例与template进行内积运算以判断与template的距离，与相近则值越大。

# 损失函数
## 多类分类SVM
损失函数：
$$L\_i = \sum\_{j\neq y\_i} \max(0, S\_j - S\_{y\_i} + \Delta); \quad S\_j = f(x\_i, W)\_j \tag 1$$

这里损失函数想要表达的是：每一个错误类别的得分至少要比正确类别的得分小$\Delta$, 另外，这里的$\Delta$其实有点像经典SVM推导中的函数间隔，由于可以通过$W$的大小来调节最终值得大小，所以这里可以将$\Delta$设为1。
> 这里的损失函数是hinge loss，不过与我们常见的SVM中的形式不太一样，
$$L_i = \max(0, 1- y_i f(x_i))\tag 2$$
一方面，公式（1）可以用于多类分类，而公式（2）用于多类分类需要使用其他策略，比如one-VS-all或者all-VS-all。

规则化：
$$R(W)=\sum\_k\sum\_l W\_{k,l}^2$$

所以，多类分类最终的目标函数为：
$$\min L = \frac{1}{N}\sum_i L_i + \lambda R(W)\tag 3$$


## Softmax 分类器
主要将公式（3）中的hinge loss换成cross-entropy 损失（交叉熵损失）：
$$L\_i = - \log(\frac{e^{f\_{y\_i}}}{\sum\_j e^{f\_j}}) = -f\_{y\_i} + \log\sum\_j e^{f\_j} \tag 4$$

其中$f_j(z) = \frac{e^{z_j}}{\sum_k e^{z_k}}$ 称为Softmax function，所以将这种损失函数命名为Softmax classifier。

Softmax classifier其实是binary Logistic  regression  classifier 在Multiple class上的一个泛化。

Softmax分类器的优点主要是能够给出分类类别的概率。

对于Softmax 分类器的解释主要有两种：

1. information theory view
    在“真实”分布p和估计分布q之间的交叉熵定义如下：
    $$H(p, q)=-\sum_x p(x) \log q(x)$$
    
    因此，Softmax分类器所做的就是最小化在估计分类概率（就是上面的$e^{f_{y_i}}/\sum_j e^{f_j}$）和“真实”分布之间的交叉熵，在这个解释中，“真实”分布就是所有概率密度都分布在正确的类别上（比如： $p=[0,\cdots, 1,\cdots, 0]$中的$y_i$的位置就有一个单独的1）。
    交叉熵损失函数“想要”预测分布的所有概率密度都在正确分类类别上。

2. probabilistic interation
 Softmax classifier将输出向量$f$中的评分值解释为没有归一化的对数概率，所以，先对$f$进行指数操作得到没有归一化的概率，然后，在进行归一化得到相应的概率值。

> 从数值上进行分析： 首先 $e^{f\_{y\_i}}/\sum\_j e^{f\_j}$ 必定属于 $(0, 1]$，要想目标函数尽量的小，那么 $\log()$ 就要尽量的大，由于 $\log$ 函数是单调递增的，所以即要求 $e^{f\_{y\_i}}/\sum\_j e^{f\_j}$ 尽量的大，而这正符合正确分类的要求。

# Logistic Regression
对于Logistic Regression的讲解，西瓜书上讲的挺清楚的，这里将一个简单的思路记录写下来以促进理解。
首先，对于一般的二类分类问题，输出标记为 $\{0, 1\}$, 而通常的线性回归模型产生的预测值为实值，我们需要将这些实值转成 $0/1$ 值，当然，最理想的便是“单位阶跃函数”

$$y=
\begin{cases}
0, z < 0; \\
0.5, z = 0; \\
1, z > 0,
\end{cases}$$

但是，单位阶跃函数不连续的性质，在求解的时候会很不方便，所以就需要寻找一个替代函数，并希望其能够单调可微。这时便需要用到对数几率函数。
对数几率函数的形式为：
$$y=\frac{1}{1+ e^{-(w^Tx + b)}}\tag 5$$
这其实是[Sigmoid函数](http://heimingx.cn/2016/05/03/sigmoid-function-and-sigmoid-kernel/)的形式。该函数对应的模型称为Logistic Regression。

将公式（5）进行简单的变化得到：
$$\ln\frac{y}{1-y}=w^Tx + b$$
这里将 $y$ 视为样本 $x$ 作为正例的可能性，则 $1-y$ 是其反例的可能性，两者的比值为 $y/(1-y)$ 称为“几率”（odds），反映了 $x$ 作为正例的相对可能性。对几率取对数则得到"对数几率"（log odds），所以，Logistic regression 又被称为“对数几率回归”。

# references
1. [cs231n: Linear classification: Support Vector Machine, Softmax](http://cs231n.github.io/linear-classify/#interpret)
2. [CS231n课程笔记翻译：线性分类笔记（下）](https://zhuanlan.zhihu.com/p/21102293?refer=intelligentunit)
3. 机器学习. 周志华 对数几率回归章节