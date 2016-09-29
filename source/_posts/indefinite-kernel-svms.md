---
title: Indefinite Kernel SVMs
date: 2016-09-29 19:37:24
categories: [Machine Learning]
tag: Machine Learning
description: 
---

研究生一年级主要都在做不定核SVM的优化，在这一领域大概从2003年开始就有学者不断的提出各种idea来做这个问题，这里做个简要的总结，方便后续的研究。

首先，对于不定核的研究主要可以分为两大类：
1. spectrum approximation
2. directly focus on the indefinite kernel

# Spectrum Approximation
这里主要有两种思想，一种是直接改变手动改变谱，另一种是学习出一个正定的核矩阵

## Spectrum Transformation
谱变换，直接去手动改变不定核矩阵，转换成正定核矩阵之后再用经典的SVM来解决相应的问题。

1. Denoise：$f(\lambda)=\max(0,\lambda)$
2. Flip: $f(\lambda)=|\lambda|$
3. Diffusion: $f(\lambda)=e^{\beta \lambda}$
4. shift: $f(\lambda)=\lambda + \eta$

这些方法都是认为负特征值为noise，显然，这一方法太粗暴了，而且现实中很多应用产生的就是一个不定的相似矩阵，此时就不能简单的认为复特征值都是noise了。

## Kernel Approximation
这个方法的主旨思想是：将对偶形式SVM的求解与正定核的近似放到同一个式子里面去优化。

\begin{align}
\max\_{\alpha\in \mathbb R^d} \min\_{K\in \mathbb R^{d \cdot d}} & \quad \alpha^T e - \frac{1}{2}\alpha^T YKY\alpha + \rho ||K-K\_0||^2\_F \\\\
s.t. & \quad \alpha^T y = 0 \\\\
& \quad 0\leq \alpha \leq C \\\\
& \quad K \geq 0
\end{align}

其中，$K_0$是已知的不定核矩阵，$K$是要求解的近似矩阵（半正定）。

这一模型是由Luss在Support vector machine classification with indefinite kernels_nips'07中提出，后续有很多学者基于此进行了优化方面的改进。比如Ye老师的Training SVM with indefintie kernels_ICML'08,以及Yihua Chen et al. 的Learning Kernels from Indefinite Similarities.其中，后者是从原问题出发（运用表示定理引入核函数）来求解的，文章中也说明了原问题与对偶问题求解的等价性。

然后基于这一思想，Gu and Guo又提出了一个很新颖的方法，将kernel PCA成功的引入到了kernel approximation里面，将不定核降维到一个低维空间中，使得此时对应矩阵变为正定的，然后在此空间中进行训练分类器。（这个方法的巧妙之处在于，用核函数就是一个升维的过程，企图在一个高维空间中能够找到一个线性可分的分类器，但是不巧的是升维之后，我们面临的核矩阵是不定的了，因此常用的凸优化方法就失效了，然后，这个方法就运用kernel PCA方法来对这个高维空间进行降维处理，尝试在降维后的空间中找到一个低维正定的表示）
具体的模型：
\begin{align} \\\\
\min\_{W,w,b,\xi} & \quad \frac{1}{2}w^Tw+C\sum\_i \xi\_i+\rho \|\phi - WW^T\phi\|\_F^2 \\\\
s.t. & \quad y\_i(w^TW^T\phi(:,i)+b)\geq1-\xi\_i, \xi\_i\geq 0 \\\\
& \quad W^TW = I\_d
\end{align}

其中，$||\phi - WW^T\phi||\_F^2$表示KernelPCA,结合SVM的Lagrange对偶+kernelPCA的变换得到
\begin{align} \\\\
\max\_\alpha \min\_V & \quad \alpha^T e - \frac{1}{2}\alpha^T Y K\_0 V V^T K\_0 Y \alpha -\rho \cdot tr(V^TK\_0K\_0V) \\\\
s.t. & \quad \alpha \cdot diag(Y)=0; \\\\
& \quad 0\leq \alpha \leq C \\\\
& \quad V^TK\_0 V = I\_d \\\\
& \quad K\_0 V V^T K\_0 \geq 0
\end{align}

其中，最后一条约束表示在低维空间中的核矩阵，为正定的，并且**只要$V$是实值向量**，就能够保证最后一个约束满足。
这篇文章：Learning svm classifiers with indefinite kernels发在AAAI'12上，并获得了当年的best paper,赞！但是，在我自己代码实现的过程中也发现了一些弊端：
1. 要求矩阵K和M的逆，所以他们的特征值不能有0.
2. 要预先判断K的正特征值个数，所以当K的正特征值很少的时候（接近负定），PCA的降到的维度就会受到相应的影响，那么最终的分类精度也就会受到影响了。

# Directly Focus on the Indefinite Kernel

这一大类里面方法还是挺迥异的，主要有SMO-type SVM，1-norm SVM， SVM in krein等一系列方法。

# SMO-type SVM
这一方法提出的比较早，在03年由林轩田和林智仁共同完成。文章主要基于sigmoid核的不同核参数的组合来讨论的，然后对SMO算法进行了稍微的改动以适用于不定核的求解，这一算法已集成到他们开源的软件LibSVM中，只要用'-t 4'的参数设置他就能自动调用相关算法来求解该问题。
另外，这一算法主要是基于对偶形式的不定核SVM问题来求解的，因此，由于核矩阵的不定性，在应用Lagrange对偶化时就不能忽略对偶间隔的问题。同时，我们的实验结果也验证了这一想法，SMO-type SVM方法的效果并不理想，与谱分解的结果相近。

# 1-norm SVM
这一方法还是挺巧妙的，他避开了在目标函数中产生不定核矩阵的做法，而是直接利用1-norm的形式，并通过强制要求核前面的组合系数大于0,将原来的1-norm svm转换成了一个Linear Programming。

\begin{align} \\\\
\min\_{\lambda, \xi, b} & \quad \sum\_j |\lambda\_j| + C \sum\_{i=1}^m \xi\_i \\\\
s.t. & \quad y\_i \cdot (b + \sum\_j \lambda\_j \cdot h\_j(x\_i)) \geq 1 - \xi\_i \\\\
& \quad \xi\_i \geq 0
\end{align}

强制规定$\lambda\geq 0$后，得到

\begin{align} \\\\
\min\_{\lambda, \xi, b} & \quad \sum\_{i=1}^m ||\lambda\_i||\_1 + C \sum\_{i=1}^m \xi\_i \\\\
s.t. & \quad Q \lambda + b y \geq 1 - \xi \\\\
& \quad \lambda, \xi \geq 0
\end{align}

同时，文章中对于这一形式也从相似函数的角度给出了解释，不过并没有很清楚的给出具体的推导化简的公式，是如何从相似函数间隔最大的公式推导到最后的LP问题的。另外，由于他上面对$\lambda$的强制要求，有点与对偶svm中的Lagrange乘子$\alpha$相似，因此，可能存在对偶间隔的问题。在实验过程中，1-norm的效果有点不太稳定，时好时坏，可能跟数据集有关吧。

# SVM in Krein
这个方法是从Reproducing Kernel Krein Space(RKKS)出发，不再是求解一个minimization problem,而是求解一个stabilization problem。
具体模型：
\begin{align}\\\\
stab\_{f\in\mathcal K, b\in \mathbb R} & \quad \frac{1}{2}\langle f, f\rangle\_{\mathcal K}\\\\
s.t. & \quad \sum\_{i=1}^l \max(0, 1-y\_i(f(x\_i)+b))\leq \tau
\end{align}

根据不定核在krein空间中的性质：$f\_{\mathcal K}=f\_+ + f\_-$, $\langle f, f\rangle\_{\mathcal K}= \langle f\_+, f\_+\rangle\_{\mathcal H^+} - \langle f\_-, f\_-\rangle\_{\mathcal H^-}$. 得到

\begin{align}\\\\
\min\_{f\_+\in\mathcal H^+, b\in \mathbb R} \max\_{f\_-\in \mathcal H^-} & \quad \frac{1}{2}\langle f\_+, f\_+\rangle\_{\mathcal H^+} - \frac{1}{2}\langle f\_-, f\_-\rangle\_{\mathcal H^-}\\\\
s.t. & \quad \sum\_{i=1}^l \max(0, 1-y\_i(f(x\_i)+b))\leq \tau
\end{align}

将上面的与max相关的部分转换成min的形式：

\begin{align}\\\\
\min\_{f\_+\in\mathcal H^+, f\_-\in \mathcal H^-, b\in \mathbb R} & \quad \frac{1}{2}\langle f\_+, f\_+\rangle\_{\mathcal H^+} + \frac{1}{2}\langle f\_-, f\_-\rangle\_{\mathcal H^-}\\\\
s.t. & \quad \sum\_{i=1}^l \max(0, 1-y\_i(f(x\_i)+b))\leq \tau
\end{align}

从$f\_+$和$f\_-$，可以构造一个正的Hilbert space（$\tilde{\mathcal K}$），有$\tilde f = f\_+ + f\_-$和$\langle \tilde{f}, \tilde{f} \rangle\_{\tilde{\mathcal K}} = \langle f\_+, f\_+\rangle\_{\mathcal H^+} + \langle f\_-, f\_-\rangle\_{\mathcal H^-}$,带到上面的公式有

\begin{align}\\\\
\min\_{\tilde{f} \in \tilde{\mathcal K}, b\in \mathbb R} & \quad \frac{1}{2}\langle \tilde{f}, \tilde{f}\rangle\_{\tilde{\mathcal K}}\\\\
s.t. & \quad \sum\_{i=1}^l \max(0, 1-y\_i(\tilde{f}(x\_i)+b))\leq \tau
\end{align}

此时，就将原来的不定核问题转变成了一个凸问题，然后运用Lagrange对偶化得到：
\begin{align}
\max\_{\tilde{\alpha}} & \quad -\frac{1}{2}\tilde{\alpha}^T\tilde{G}\tilde{\alpha} + \tilde{\alpha}^T\textbf{1}\\\\
s.t. & \quad \tilde{\alpha}^T \textbf{y} = 0\\\\
& \quad 0\leq \tilde{\alpha}\_i\leq C
\end{align}

到这里，求解上面的这个凸对偶问题就可以得到$\tilde{\alpha}$，然后再设法找到对应的RKKS中的解就可以了。因此，文章中给出了如何将等价的RKHS中的点映射到RKKS中去（第3.3.1节）。具体做法是$\tilde{\alpha} = P \alpha$.

svm in krein的确是很不错的，从上面的理论推导来看还是很完善的，在我们的实验上来看也是很不错的，在大多数数据集上都能有很好的效果。

# References
1. Chen, J., and Ye, J. 2008. Training svm with indefinite kernels.In Proceedings of the Twenty-fifth International Conference on Machine Learning, 136–143. Helsinki, Finland: ACM.
2. Alabdulmohsin, I. M.; Gao, X.; and Zhang, X. 2014. Support vector machines with indefinite kernels. In Proceedings of the Sixth Asian Conference on Machine Learning, volume 39, 32–47. Nha Trang, Vietnam: JMLR.org.
3. Chen, Y.; Gupta, M. R.; and Recht, B. 2009. Learning kernels from indefinite similarities. In Proceedings of the Twenty-sixth International Conference on Machine Learning, 145–152. Montreal, Quebec, Canada: ACM.
4. Gu, S., and Guo, Y. 2012. Learning svm classifiers with indefinite kernels. In Proceedings of the Twenty-Sixth AAAI Conference on Artificial Intelligence, 942–948. Toronto, Ontario, Canada.: AAAI Press.
5. Lin, H.-T., and Lin, C.-J. 2003. A study on sigmoid kernels for svm and the training of non-psd kernels by smo-type methods. Neural Computation 1–32.
6. Loosli, G.; Canu, S.; and Ong, C. S. 2016. Learning svm in kreˇın spaces. IEEE Transactions on Pattern Analysis and Machine Intelligence 38(6):1204–1216.
7. Luss, R., and d’Aspremont, A. 2008. Support vector machine classification with indefinite kernels. In The Twenty-First Conference on Neural Information Processing Systems,953–960. Vancouver, British Columbia, Canada.: Curran Associates, Inc.
8. Wu, G.; Chang, E. Y.; and Zhang, Z. 2005. An analysis of transformation on non-positive semidefinite similarity matrix for kernel machines. In Proceedings of the Twentysecond International Conference on Machine Learning, volume 8. Citeseer.