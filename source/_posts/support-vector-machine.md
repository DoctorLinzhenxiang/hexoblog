---
title: 支持向量机 梳理
date: 2016-06-08 22:24:45
categories: [Machine Learning]
tag: [Machine Learning]
description:
---

SVM是非常经典的分类方法，在李航的《统计学习方法》中给出基本模型是定义在特征空间上间隔最大的线性分类器，学习策略是间隔最大化，学习算法是求解凸二次规划的最优化算法。详细的推导过程书上写的很明确了，现在我自己再梳理一下，并穿插就一些疑问尝试给出自己的答案。

<!--more-->

首先，SVM的一种思路就是间隔最大化，对于间隔有函数间隔与几何间隔，由于函数间隔并不唯一确定，所以选择几何间隔，得到原始优化目标函数

\begin{align} \\\\
\max_{w,b} \quad & \gamma \\\\
s.t. \quad &y_i(\frac{w}{||w||}x_i+\frac{b}{||w||})\geq \gamma, i=1,2,...,N
\end{align}

根据函数间隔与几何间隔之间的关系，可以将约束中的$||w||$去掉

\begin{align} \\\\
\max_{w,b} \quad &\frac{\hat \gamma}{\|w\|} \\\\
s.t. \quad &y_i(w\cdot x_i+b)\geq \hat \gamma, i=1,2,...,N
\end{align}

取$\hat \gamma = 1$并作简单变换得到

\begin{align} \\\\
\min_{w,b} \quad &\frac{1}{2}\|w\|^2 \\\\
s.t. \quad &y_i(w\cdot x_i + b) - 1 \geq 0,i=1,2,...,N
\end{align}

这样，我们就得到了线性可分SVM的基本优化目标函数。接下来，是运用Lagrange对偶化来求解对偶问题。

> 这里Lagrange对偶化有一个对偶间隔的问题，当目标函数和约束函数均是凸的情况下，当对偶问题的解满足KKT条件时，对偶间隔为0，即对偶问题与原问题等价。所以我们可以放心大胆的去求对偶问题了。注意，这里的KKT条件并不是SVM中独有的，应该是针对Lagrange对偶化的一个条件。但是，由于Lagrange对偶化是针对凸函数而言的，当核函数不定时，目标函数不再是凸函数，所以也就不能运用Lagrange对偶化来求解了，因为存在对偶间隔。

Lagrange对偶化后得到Lagrange函数

$$L(w,b,\alpha) = \frac{1}{2}||w||^2 - \sum\_{i=1}^N\alpha\_i y\_i (w \cdot x\_i + b) + \sum\_{i=1}^N\alpha\_i$$

对$w,b$求偏导，并代入得到

\begin{align} \\\\
\min\_\alpha \quad &\frac{1}{2}\sum\_{i=1}^N\sum\_{i=1}^N \alpha\_i \alpha\_j y\_i y\_j (x\_i\cdot x\_j) - \sum\_{i=1}^N \alpha\_i \\\\
s.t. \quad &\sum\_{i=1}^N \alpha\_i y\_i = 0 \\\\
&\alpha\_i \geq 0, i= 1,2,...,N
\end{align}

到了这里，SVM的对偶优化目标就得到了。但是目前的模型容易过拟合，并且当样本数据不能够线性划分时，需要引入软间隔，此时原问题的优化目标为

\begin{align}
\min\_{w,b} \quad &\frac{1}{2}||w||^2 + C\sum\_{i=1}^N \xi\_i \\\\
s.t. \quad &y\_i(w\cdot x\_i + b) \geq 1 - \xi\_i ,i=1,2,...,N \\\\
&\xi\_i \geq 0, i=1,2,...,N
\end{align}

同样运用Lagrange对偶化后得到对应的对偶优化目标

\begin{align}
\min\_\alpha \quad &\frac{1}{2}\sum\_{i=1}^N\sum\_{i=1}^N \alpha\_i \alpha\_j y\_i y\_j (x\_i\cdot x\_j) - \sum\_{i=1}^N \alpha\_i \\\\
s.t. \quad &\sum\_{i=1}^N \alpha\_i y\_i = 0 \\\\
&0 \leq \alpha\_i \leq C, i= 1,2,...,N
\end{align}

然后根据求解得到最优的$\alpha^\*$,根据公式就能够求得$w^\*$,$b^\*$了。这里$w^\*$是唯一的，而$b^\*$不是。
到这里，SVM的通过间隔最大化的大体推导思路就结束了。接下来需要关注的就是对于支持向量的讨论了，在《统计学习方法》的P113有较为详细的分析，讲的非常好，这里作简要分析。
![软间隔支持向量](/img/blog/svm/svm_sv.png)

根据软间隔支持向量的图，可以看出主要有四种情况：

- $\alpha_i^* \lt C$,则 $\xi_i = 0$,且支持向量$x_i$恰好落在间隔边界上（即图中的虚线上）
    >在Lagrange对偶化的过程中，有$C-\alpha_i-\mu_i = 0$这样一个约束，由于$\alpha_i^\* \lt C$,那么有$\mu_i^* > 0$,那么表明对应的$\xi_i = 0$(KKT条件),即表明对于这一样本点，对于间隔超平面偏差为0.

- $\alpha_i^* = C， 0 <\xi_i< 1$,则该样本分类正确，且落在间隔边界与分离超平面之间
- $\alpha_i^* = C, \xi_i = 1$, 则该样本落在分离超平面上
- $\alpha_i^* = C, \xi_i > 1$, 则该样本位于分离超平面误分的一侧。

到这里线性SVM的基本知识点就差不多了。然后，在《统计学习方法》和西瓜书《机器学习》中都有降到对于SVM的另外一种解释。就是损失函数+正则化项的目标函数。

$$\min\_{w,b} [1-y\_i(w\cdot x\_i + b)]\_+ + \lambda ||w||^2$$

在西瓜书中明确说了这一模型与前面说的硬间隔最大化的原问题模型是等价的。其中，损失函数是一个非凸和非连续的函数，所以通常就选择一种替代损失函数，有hinge loss，指数损失， 对率损失。后一项正则化项可以根据需求来选择不同的范数，在间隔最大化的思路中，正则化项就选择的是$L_2$范数，这倾向于给出更多的支持向量点，而需要稀疏的支持向量时，可以选择$L_0, L_1$范数，其中，$L_0$范数会导致目标函数非凸，这样就又变成一个新的SVM问题---非凸SVM的求解。

> 对于损失函数，通常选择的是hinge loss，一个很重要的点是它也能够带来稀疏的支持向量点。

接下来，就是核函数的引入了，其实做法很简单，就是将样本内积换成核函数的形式，其他的都没有改动。

\begin{align}
\min\_\alpha &\frac{1}{2}\sum\_{i=1}^N\sum\_{i=1}^N \alpha\_i \alpha\_j y\_i y\_j K(x\_i\cdot x\_j) - \sum\_{i=1}^N \alpha\_i \\\\
s.t. &\sum\_{i=1}^N \alpha\_i y\_i = 0 \\\\
&0 \leq \alpha\_i \leq C, i= 1,2,...,N
\end{align}

其实对于核函数的理解需要了解再生核希尔伯特空间的一些具体性质（比如为什么$K(x_i, x_j)$就能够表示样本映射到特征空间（Hilbert空间）后的内积，这其实就是再生核的性质，具体的可以参考[再生核希尔伯特空间](http://heimingx.cn/2016/05/24/reproducing-kernel-hilbert-spaces/)的介绍）
最后，就是对这个优化问题的求解了，经典的快速算法就是SMO算法，可以参考[SMO算法实现](http://heimingx.cn/2016/05/01/smo-algorithm/)

参考
1. 李航老师的统计学习方法
2. 周志华老师的机器学习