---
title: manifold learning
date: 2015-11-25 23:33:43
categories: Machine Learning
tag: machine learning
description: 介绍几种经典的流形学习算法
mathjax: true
---

好久好久木有写了，由于时间比较零散，总是感觉没有学到什么东西，也就没什么可写的，最近模式识别课正好有个作业是写一个综述，我选的流形学习，把自己的所学记录下来。

# 概括
流形学习是一种实现非线性降维的方法，这一方法的发展主要得益于2000年Science上两篇文章的发表---ISOMAP和LLE（Local linear embedding）,第一篇主要是对MDS方法的改进，用测地线距离（geodesic diatance）取代了欧氏距离，从全局的角度对高维数据的实现降维。而第二篇正好相反，它主要从局部角度出发，用每个样本点的k临近点来线性表示这个样本点，然后在低维空间中进行重构。
接着，大体从这两个角度出发，后继研究者又给出了Laplacian Eigenmap（LE）、Hessian Eigenmap（HE）、Local Tangent Space Alignment(LTSA)、Maximum Variance Unfolding（MVU）等等。

# ISOMAP
algorithm 
> step1:建立邻域关系图G(V,D)，其中，V表示vertex，D表示distance
> step2:计算测地线距离矩阵GD，方法是在邻域矩阵G中运用Dijkstra算法或Floyd算法来寻找最短路径。
> step3:在测地线距离矩阵GD上运用古典MDS算法，寻找低维构造点$Y=\\{y_1,y_2,...,y_N\\}$

在step2中，算法假设在流形结构的局部是可以线性化的，即再局部可以用欧式距离来近似代替测地线距离。

# LLE
algorithm
> step1:建立k近邻邻域关系图G(V,D)
> step2:计算重构权值矩阵：对于每个 $x_i\in\mathbb R^m$,计算每个由k近邻 $N\_i=\\{x\_j,j\in G\\}$ 重构 $x\_i$ 的权值系数 $w\_{ij}$
> $$\epsilon\_{LLE}=\sum\_{i=1}^N||x\_i-\sum\_{j=G\_i}w\_{ji}x\_j||^2$$
> step3:寻找在低维空间中的表示：
> $$\min\_Y\Phi(Y)=\sum\_{i=1}^N||y\_i-\sum\_jw\_{ij}y\_j||^2$$

对于step3，写成矩阵的形式：
\begin{align} \\\\
& \min\_Y||Y-WY||^2 \\\\ 
& =\min\_Y trace(Y-WY)^T(Y-WY) \\\\
& =\min\_Y trace(Y^T(I-W)^T(I-W)Y) \\\\
& s.t.Y^TY=I 
\end{align}
求解一个关于$Y$的二次型最小化问题，解得唯一最小解$Y^\*$,满足$(I-W)^T(I-W)Y^\*=\lambda Y^\*$,相当于对$(I-W)^T(I-W)$进行特征值分解。

**LLE VS PCA**
PCA最终实际上是对所有样本点的散度进行特征值分解，而LLE算法，我们可以看到实际上是对由权重矩阵构成的矩阵求特征值分解，这相当于只运用了局部样本点的属性。所以，我们可以从LLE中看到PCA的影子。

# LE
algorithm
> step1:建立k近邻邻域关系图G(V,D)
> step2:**选择**权值，运用热核公式$$W_{ij}=e^{-\frac{||x_i-x_j||^2}{t}}$$,这里如果$x\_i$和$x\_j$有连接，则$||x\_i-x\_j||$表示两点的距离；否则令权重为0
> step3:最小化损失函数：$$\Phi (Y)=\sum\_{ij}(y\_i-y\_j)^2w\_{ij}$$ 
 
**对step3的公式的理解：** $w\_{ij}$权重值越大就表示数据点$x\_i$和$x\_j$靠的越近。所以，在低维表示中,$y\_i$和$y\_j$之间的距离就能够体现在损失函数上，比如，假设$w\_{ij}$权重值很大，即表明在原始空间中，数据点$x\_i$和$x\_j$靠的很近，如果在低维空间中的点$y\_i$和$y\_j$离得很远，就会得到惩罚而不会在此处取得最佳值。所以，这保证了在高维空间中近邻的点，在低维空间中会变得更近邻。

对于step3的求解，算法运用到了图理论中的拉普拉斯矩阵将原问题转化为了一个特征问题：
\begin{align} \\\\
\Phi (Y) & = \sum\_{ij}(y\_i-y\_j)^2w\_{ij} \\\\
& =\sum\_{ij}(y\_i^2+y\_j^2-2y\_iy\_j)w\_{ij} \\\\
& =\sum\_iy\_i^2m\_{ii}+\sum\_jy\_j^2m\_{jj}-2\sum\_{ij}y\_iy\_jw\_{ij} \\\\
& =2Y^TMY-2Y^TWY \\\\
& =2Y^TLY
\end{align}

其中，$m\_{ii}=\sum\_jW\_{ji}$,$M\_{ii}=\sum\_jW\_{ji}$,$L=M-W$是Laplacian矩阵。因此，对于低维表示Y的计算可以通过一个泛化特征方程的求解得到.
$$Lf=\lambda Mf$$

# HE
algorithm
> step1:建立k近邻邻域关系图G(V,D)
> step2:在这k个最近邻距离G上运用PCA来获得对应样本点$x_i$的一组局部切空间基底.即在每个样本点处的局部切空间基底就等价于计算距离的协方差矩阵$cov(x_i.)$的d个主特征向量$M=\\{m_1,m_2,...,m_d\\}$,且$k &geq; d$.
> step3:通过已经得到的局部切空间基底来估计流形的Hessian矩阵.

HE算法理解的还不够透彻，算法的大概思想就是通过保持局部样本点的Hessian矩阵这一属性来实现降维。后面会有一个与LLE算法的比较，能够直观的感受到HE算法的思想。

# LTSA
algorithm
> step1:建立k近邻邻域关系图G(V,D)
> step2:计算每个数据点$x\_i$处局部切空间的基底，方法同样是将PCA算法应用在由数据点$x\_i$ 的k近邻点$x\_{i_j}$得到的局部散布矩阵上，它能够将数据点$x\_i$的近邻点映射到局部切空间$\theta\_i$中.局部切空间$\theta$拥有一个重要的属性──从局部切空间坐标$\theta\_{i\_j}$到低维表示$y\_{i\_j}$存在着一个线性映射$L\_i$。
> step3:最小化损失函数$$\min\_{Y\_i,L\_i} \sum\_i||Y\_iJ\_k-L\_i\theta\_i||^2$$其中，$J\_k$表示k维的中心矩阵。

对于**LLE/LTSA/HE**这三种方法之间的关系，可以从LTSA论文的推导中清晰的看出来,同时也在一篇博士论文内看到这样的阐述：
> 
令$ x\in\mathbb R^m$,把函数f(x)在x=0点进行二阶Taylor展开如下：
$$ f(x)=f(0)+T\_{f(0)}^T\cdot x+x^T\cdot H\_{f(0)}\cdot x+o(f(0))$$
其中，$T\_{f(0)}=grad(f(x))|\_{x=0}$是f(x)在x=0点的梯度，$H\_{f(0)}$是f(x)在x=0点的Hessian矩阵。
那么对于LLE，它通过领域内的数据点本身的线性组合重构样本来获取局部几何，LTSA利用样本在局部切空间上的投影重构样本来获取局部几何，而HE则是利用样本点之间的Hessian矩阵来获取局部几何。

# MVU
algorithm
> step1:建立k近邻邻域关系图G(V,D)
> step2:计算保持局部距离的最大方差展开：
\begin{align} \\\\
& \max \sum\_i||y\_i||^2 \\\\
& s.t.  ||y\_i-y\_j||^2=||x\_i-x\_j||^2,if &amp; (x\_i,x\_j) &amp; is &amp; k-nn \\\\
& \sum\_i y\_i=0
\end{align}
step3:对角化Gram矩阵$K$,令$K\_{ij}=y\_i\cdot y\_j$为Gram矩阵，它决定了输出数据$Y=K^{1/2}$,可以将step2中问题转化成SDP问题
\begin{align} \\\\
& \max \sum\_iK\_{ii} \\\\
& s.t.  K\_{ii}+K\_{jj}-2K\_{ij}=||x\_i-x\_j||^2,if &amp;(x\_i,x\_j) &amp;is &amp; k-nn \\\\
& \sum\_i K\_{ij}=0 \\\\
& K &geq; 0 	(保证半正定)
\end{align}

根据该半定规划求解得到的Gram矩阵 ，可以利用特征值分解来得到数据的低维表示。所以，MVU算法又称为半定嵌入（Semi-definite Embedding, SDE）。

# reference

1.Joshua B. Tenenbaum, Vin de Silva, John C. Langford. A global geometric framework for nonlinear dimensionality reduction. Science, 2000,290,2319-2323.

2.Sam T. Roweis, Lawrence K. Saul. Nonlinear Dimensionality reduction by locally linear embedding. Science, 2000. 290,2323-2326.

3.Zhang Zh., Zha H., Principal Manifolds and Nonlinear Dimension Reduction via Local Tangent Space Alignment, Journal. Sci. Comput. SIAM. vol.26, no.1, Jan. 2004, pp.319-338.

4.Belkin M., Niyogi P., Laplacian Eigenmaps for dimensionality reduction and data representation, Neural Computation, vol.15, no.6, 2003, pp.1373-1396.

5.Donoho D.L., Grimes C.E., Hessian Eigenmaps: locally linear embedding techniques for high dimensional data, Proceedings of the National Academy of Arts and Sciences, vol.100, 2003, pp.5591-5596.

6.Weinberger K.Q., Saul L.K., Unsupervised learning of image manifolds by semidefinite programming, CVPR, II, 2004, pp.988-995.

7.Elnaz Golchin, Keivan Maghooli. Overview of manifold learning and its application in medical data set. International journal of biomedical engineering and science(IJBES),Vol.1, No. 2. July 2014

8.李春光.流形学习及其在模式识别中的应用,2007,博士毕业论文

9.杨剑，流形学习问题，2004.

10.[浅谈流形学习][1]

11.[流形学习综述-张重][2]

12.[scikit-learn:manifold learning][3]



[1]: http://blog.pluskid.org/?p=533
[2]: http://blog.sciencenet.cn/blog-722391-583413.html
[3]: http://scikit-learn.org/stable/modules/manifold.html
