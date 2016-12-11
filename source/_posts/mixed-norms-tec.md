---
title: 混合范数初窥
date: 2016-12-11 20:32:45
categories: [Machine Learning]
tag: [Machine Learning]
description: 
---

在机器学习里面，如何避免过拟合是一个永恒的话题，缓解该问题的一个有效方法就是在损失函数后面加上一个规则化项，而对于规则化项的不同选择便会带来不同的效果，下面就对他们进行简单的对比介绍。

# $L_p$范数

常见$L_p$范数规则化的表达式为：

$$||x||\_p=\left (\sum\_{i=1}^n|x\_i|^p\right )^{\frac{1}{p}}$$

对于这些范数的表达，先给两个比较经典的图一起来感受一下：

![different norms](/img/blog/mixed_norms/norm.png)

![Minkowski3](/img/blog/mixed_norms/Minkowski3.png)

根据上面两个图，我们不难发现，当$p \leq 1$时，图形不是光滑的，在某些点是不可微的，而正是这一属性可以给解带来稀疏性，同时，我们也可以看出此时p值越小，解的稀疏性也就越大，极限情况便是$p=0$时，稀疏性达到最大，便是熟知的$L_0$范数,但是，由于此时，函数是离散的，不太好优化，所以，通常选择与其解近似的$L_1$范数来达到稀疏解的要求。当$p \geq 1$时，从图像中明显可以看出来，他们变得越来越光滑，相应的其约束的解也会变得相对平滑，也就不再具有稀疏性了，不过，通常$L_2$范数的性能会更好一点，因为毕竟解的分布范围比较的广，更加的泛化。

鉴于这些范数有着各自不同的优缺点，有学者便想着将他们组合起来，以达到更加丰富的效果。

# mixed norms

对于混合范数的研究已经有很多成果了，比如比较经典的[Group Lasso](http://pages.stat.wisc.edu/~myuan/papers/glasso.final.pdf) 和 [Elitist Lasso](http://link.springer.com/article/10.1007/s11760-008-0076-1)。这里先给自己挖一个大坑，后面有时间看一看这两篇文章~~~下面，我将从一个关于mixed norm的应用来切入，探究一下其奇特的效果。

## 文章标题 

**Multiple Indefinite Kernel Learning with Mixed Norm Regularization** (ICML‘09)

## 主要内容

这篇文章主要解决的是一个关于多核学习的问题，作者没有按照通用的思路，只在核函数的维度，来考虑不同核函数组合带来的效果，而是另辟蹊径，同时在样本和核函数两个维度去考虑，这也为引入 mixed norm 埋下了伏笔。

> 注：
多核学习问题的通用思路为：多核核函数的线性组合，然后将这组合之后的核代入到原SVM问题
核函数线性组合：$$K(x, x') = \sum\_{m=1}^M d\_m K\_m(x, x')$$
优化目标：
\begin{align}
\min\_{f\_m, b, \xi, d} &\quad \frac{1}{2}\sum\_m \|f\_m\|\_{\mathcal{H}\_m}^2 + C\sum\_i\xi\_i \\\\
s.t. &\quad y\_i \sum_m f\_m(x_i) + y\_i b \geq 1 -\xi\_i \quad \forall i \\\\
&\quad \xi\_i \geq 0 \\\\
&\quad \sum\_m d\_m = 1, d\_m \geq 0
\end{align}
(引用自Simple MKL)

本文作者直接构造的核函数组合的形式为：

$$\mathcal{F}\_S = \left \\{x \to \sum\_{i,t=1}^{n,\tau} \alpha\_{it} k\_t(x\_i, x): \alpha\in\mathbb{R}^{n\tau}, k\_t \in \mathcal{K} \right \\}$$

其中，$\mathcal K = \\{ k1, k2, \cdots , k_\tau\\}$表示候选核函数集合。同时，作者也将该形式作为其最终预测的判别函数，即判别函数为**$sign(f(x))$**, $f\in \mathcal{F}\_S$.

因此，作者得到了其待优化的目标函数：
$$\min\_{\alpha\in\mathbb{R}^{n\tau}}\quad \sum\_{i=1}^n |1-y\_ik\_i^T\alpha|\_+^2 + \frac{\lambda}{q} ||\alpha||\_{pq;r}^q$$
这里，目标函数主要可以分两段来看，前半段为loss function，一个squared hinge loss，后半段为regularization term，即 mixed norm. 其中，mixed norm的具体形式为：
$$||\alpha||\_{pq;1} = \left [\sum\_{i=1}^n \left [ \sum\_{t=1}^\tau |\alpha\_{it}|^p\right ]^{q/p}\right ]^{1/q}$$
$$||\alpha||\_{pq;2} = \left [\sum\_{t=1}^\tau \left [ \sum\_{i=1}^n |\alpha\_{it}|^p\right ]^{q/p}\right ]^{1/q}$$
(**注意，两个式子的求和顺序是不一样的！**)

对于这两个式子，文章中2.3节给出了详细的分析，并给出了四个简图来说明。
![difference of mixed norms](/img/blog/mixed_norms/difference_of_mixed_norms.png)
对这四幅图简单的分析：
- 第一幅图的mixed norm参数为$p=1,q=2$，表示在核函数的维度上具有稀疏性，样本维度上不单独具有，即对于不同的样本，其对应的核函数组合可以体现出不同的稀疏性。
$$||\alpha||\_{12;1} = \left [\sum\_{i=1}^n \left [ \sum\_{t=1}^\tau |\alpha\_{it}| \right ]^{2}\right ]^{1/2}$$

- 第二幅图 $p=2,q=1$，表示在核函数的维度上不具有单独的稀疏性，而是通过样本维度的稀疏性来传递到核函数组合整体上的稀疏，即对于样本维度而言，通过1范数来达到了稀疏性，当某个样本被选择稀疏表达，那么与该样本相关的所有核函数组合便都具有了稀疏性。
$$||\alpha||\_{21;1} = \left [\sum\_{i=1}^n \left [ \sum\_{t=1}^\tau |\alpha\_{it}|^2\right ]^{1/2}\right ]^{1}$$

- 第三幅图 $p=1,q=2$, 与第一幅图类似，表示在样本维度具有稀疏性，核函数维度不具有，即对于不同的核函数，其对应的所有涉及到的样本可以体现出不同的稀疏性（核内稀疏不同）。
$$||\alpha||\_{12;2} = \left [\sum\_{t=1}^\tau \left [ \sum\_{i=1}^n |\alpha\_{it}| \right ]^{2}\right ]^{1/2}$$

- 第四幅图 $p=2,q=1$，与第二幅图类似，表示在样本维度上不具有单独的稀疏性，而是通过核函数维度的稀疏性来传递到样本维度的稀疏，即对于核函数组合的维度而言，通过1范数来达到了稀疏性，当某一核函数被选择稀疏表达，那么与该核函数相关的所有样本的$\alpha$就都变得稀疏了。
$$||\alpha||\_{21;2} = \left [\sum\_{t=1}^\tau \left [ \sum\_{i=1}^n |\alpha\_{it}|^2\right ]^{1/2}\right ]^{1}$$

文章中表明，$||\cdot||\_{12;r}$与Elitist Lasso相关，$||\cdot||\_{21;r}$与Group Lasso相关。

对于这个优化问题的求解，作者运用了proximal algotithm来求解，由于与本主题无关，详细的求解方法可以参考论文。

最后，作者在$Titanic$数据集上进行了实验，并给出了与上面图类似的关于稀疏性的实验结果图：
![experiments on titanic dataset](/img/blog/mixed_norms/experiment_on_mixed_norms.png)
这里对实验结果的分析就不再赘述了，文章中给出了详细的分析。

## 小评

首先，这篇paper的作者在给出了一种比较巧妙的方法来解决多核学习的问题，没有按照常规思路从原始的SVM问题出发，通过对偶化来引入核函数，而是直接以核函数的组合来作为评价函数，同时，从样本和核函数组合两个维度进行了考虑，这也为后面引入混合函数做好了准备。总之，想法还是挺新颖的！不过，也有一些问题吧，其一，这个题目是Multiple **Indefinite** Kernel Learning，但是全篇只有在conclusion部分提到了Indefinite Kernel的问题，所以，给人一种全篇都在跑题的感觉Orz~，虽然，核函数是不定的时候，该模型也能work，但是好歹在实验的时候搞个不定的核吧！（仅个人看法~）其二，对于实验而言，发现Titanic数据集上线性核居然比高斯核效果要好，回头实验一把看看！还有就是多核的意义应该主要体现在了对于核函数的自动选择，而不再需要人为的靠经验选择了，与单核相比，实验精度并不一定会有提升。所以，对于多核学习的研究，应该重点放在对模型的改进，或是新思路的探讨吧，最后只需要与多核方法进行比较就可以了。

# references
1. [Multiple Indefinite Kernel Learning with Mixed Norm Regularization](http://www.machinelearning.org/archive/icml2009/papers/520.pdf)
2. Simple MKL

