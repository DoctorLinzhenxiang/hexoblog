---
title: A Review of Multiple Kernel Learning
date: 2016-09-29 13:31:56
categories: [Machine Learning]
tag: Machine Learning
description: 
---

今天花了一整天的时间看了一篇关于Multiple kernel Learning的综述文章，他是以Visual object recognition作为应用背景来分析的。文章：Multiple Kernel Learning for Visual Object Recognition：A Review，发在TPAMI'14.

文章的出发点是发现在多篇文章中关于MKL的阐述是有冲突的（文章中表1&2）：
1. 在有些文章中说MKL方法优于平均核权重的MKL方法[8,39,48]，而另一些文章中却给出了相反的结论[13,24,30,31]。
2. MKL与single kernel SVM之间的比较
3. L1-MKL与Lp-MKL之间的比较
4. 不同MKL方法在的computational efficiency的比较，由于不同文章中的实验结果评价不一致，比如有些是以迭代步数作为收敛的衡量，而有些是以消耗的时间作为衡量[11,30],有些基于支持向量的个数[8,47]
其实，这主要跟相关文章中的实验设置、核函数的数量及数据集有关，在这篇文章的实验部分给出了详细的分析。

MKL方法围绕两类问题展开：
1. 利用不同的MKL形式来提高分类精度
2. 利用不同的优化方法来提升MKL的学习效率

# Overview of MKL

- 另一篇关于MKL的综述：[32]M. Gönen and E. Alpaydin, “Multiple kernel learning algorithms,” J. Mach. Learn. Res., vol. 12, pp. 2211–2268, Jul. 2011.

- MKL的首次提出：[33]G. Lanckriet, N. Cristianini, P. Bartlett, and L. E. Ghaoui, “Learning the kernel matrix with semi-definite programming,” J. Mach. Learn. Res., vol. 5, pp. 27–72, Jan. 2004.

- SMO for MKL: 
1. [9]S. Vishwanathan, Z. Sun, N. Ampornpunt, and M. Varma, “Multiple kernel learning and the SMO algorithm,” in Proc. NIPS, Nottingham, U.K., 2010, pp. 2361–2369.
2. [16]F. R. Bach, G. R. G. Lanckriet, and M. I. Jordan, “Multiple kernel learning, conic duality, and the SMO algorithm,” in Proc. 21st ICML, New York, NY, USA, 2004.

- L1-norm MKL: [34]S. Sonnenburg, G. Rätsch, and C. Schäfer, “A general and efficient multiple kernel learning algorithm,” in Proc. NIPS, 2006, pp. 1273–1280.

- Lp-norm MKL: 
1. [10]M. Kloft, U. Brefeld, S. Sonnenburg, and A. Zien, “lp-Norm multiple kernel learning,” J. Mach. Learn. Res., vol. 12, pp. 953–997, Mar. 2011. 
2. [31]R. Tomioka and T. Suzuki, “Sparsity-accuracy trade-off in MKL,” in Proc. NIPS Workshop Understanding Multiple Kernel Learning Methods, 2009.
3. [40]F. Yan, K. Mikolajczyk, J. Kittler, and A. Tahir, “A comparison of L1 norm and L2 norm multiple kernel SVMs in image and video classification,” in Proc. Int. Workshop CBMI, 2009.

- negative entropy based MKL: [30]Z. Xu, R. Jin, S. Zhu, M. Lyu, and I. King, “Smooth optimization for effective multiple kernel learning,” in Proc. AAAI Artif. Intell., 2010.

- mixed norms MKL: 
1. [35]M. Kowalski, M. Szafranski, and L. Ralaivola, “Multiple indefinite kernel learning with mixed norm regularization,” in Proc. 26th ICML, Montreal, QC, Canada, 2009, pp. 545–552.
2. [15]C. Longworth and M. J. Gales, “Multiple kernel learning for speaker verification,” in Proc. ICASSP, Las Vegas, NV, USA, 2008, pp. 1581–1584.
3. [72]V. Sindhwani and A. C. Lozano, “Non-parametric group orthogonal matching pursuit for sparse learning with multiple kernels,” in Proc. NIPS, 2011, pp. 414–431.

- Bregman divergence: [9]S. Vishwanathan, Z. Sun, N. Ampornpunt, and M. Varma, “Multiple kernel learning and the SMO algorithm,” in Proc. NIPS, Nottingham, U.K., 2010, pp. 2361–2369.

- theoretical studies show that L1 norm will result in a small generalization error: 
1. [36]C. Cortes, M. Mohri, and A. Rostamizadeh, “Generalization bounds for learning kernels,” in Proc. 27th ICML, Haifa, Israel, 2010.
2. [37]Z. Hussain and J. Shawe-Taylor, “A note on improved loss bounds for multiple kernel learning,” CoRR abs/1106.6258, 2011.

- a nonlinear combination of base kernels: **[non-convex optimization problems]**
1. [12]M. Varma and B. R. Babu, “More generality in efficient multiple kernel learning,” in Proc. 26th ICML, New York, NY, USA, Jun. 2009, pp. 1065–1072. **[polynomial combination]**
2. [23]J. Saketha Nath et al., “On the algorithmics and applications of a mixed-norm based kernel learning formulation,” in Proc. NIPS, 2009.
3. [28]C. Cortes, M. Mohri, and A. Rostamizadeh, “Learning non-linear combinations of kernels,” in Proc. NIPS, 2009, pp. 396–404.**[polynomial combination]**
4. [44]J. Aflalo, A. Ben-Tal, C. Bhattacharyya, J. S. Nath, and S. Raman, “Variable sparsity kernel learning,” J. Mach. Learn. Res., vol. 12, pp. 565–592, Feb. 2011.
5. [45]F. Bach, “Exploring large feature spaces with hierarchical multiple kernel learning,” in Proc. NIPS, 2009.
6. [17]D. P. Lewis, T. Jebara, and W. S. Noble, “Nonstationary kernel combination,” in Proc. 23rd ICML, New York, NY, USA, 2006, pp. 553–560.

- an instance-dependent linear combination: **[non-convex optimization problems]**
1. [6]J. Yang, Y. Li, Y. Tian, L. Duan, and W. Gao, “Group-sensitive multiple kernel learning for object categorization,” in Proc. 12th ICCV, Kyoto, Japan, 2009.
2. [46]J. Yang, Y. Li, Y. Tian, L.-Y. Duan, and W. Gao, “Per-Sample multiple kernel approach for visual concept learning,” EURASIP J. Image Video Process., vol. 2010, pp. 1–13, Jan. 2010.
3. [47]M. Gönen and E. Alpaydin, “Localized multiple kernel learning,” in Proc. 25th ICML, New York, NY, USA, 2008.

- subgradient descent (SD) algorithms: [43]A. Rakotomamonjy, F. Bach, Y. Grandvalet, and S. Canu, “SimpleMKL,” J. Mach. Learn. Res., 9(11), pp. 2491–2521, 2008.

- semi-infinite linear programming (SILP) based algorithm: [50]S. Sonnenburg, G. Rätsch, C. Schäfer, and B. Schölkopf, “Large scale multiple kernel learning,” J. Mach. Learn. Res., vol. 7, pp. 1531–1565, Jul. 2006. 

- SD优于SILP：
1. [8]Z. Xu, R. Jin, H. Yang, I. King, and M. R. Lyu, “Simple and efficient multiple kernel learning by group lasso,” in Proc. 27th ICML, Haifa, Israel, 2010
2. [41]A. Rakotomamonjy, F. Bach, S. Canu, and Y. Grandvalet, “More efficiency in multiple kernel learning,” in Proc. 24th ICML, Corvallis, OR, USA, 2007.
3. [42]Z. Xu, R. Jin, I. King, and M. R. Lyu, “An extended level method for efficient multiple kernel learning,” in Proc. NIPS, 2009, pp. 1825–1832.

- SILP优于SD：[11]M. Kloft et al., “Efficient and accurate lp-norm multiple kernel learning,” in Proc. NIPS, 2009, pp. 997–1005.

# relationship to other approaches

- feature selection: 
1. [51]H. Liu and L. Yu, “Toward integrating feature selection algorithms for classification and clustering,” IEEE Trans. Knowl. Data Eng., vol. 17, no. 4, pp. 491–502, Apr. 2005.
2. [52]F. R. Bach, “Consistency of the group Lasso and multiple kernel learning,” J. Mach. Learn. Res., vol. 9, pp. 1179–1225, Jun. 2008.**[group lasso]**

- metric learning: [53]T. Hertz, “Learning distance functions: Algorithms and applications,” Ph.D. dissertation, Hebrew Univ. Jerusalem, Jerusalem, Israel, 2006.

- kernel alignment: [54]N. Cristianini, J. Kandola, A. Elisseeff, and J. Shawe-Taylor, “On kernel-target alignment,” in Proc. NIPS, 2002, pp. 367–373.

- parametric kernel learning: 
1. [55]O. Chapelle, J. Weston, and B. Schölkopf, “Cluster kernels for semi-supervised learning,” in Proc. NIPS, 2003, pp. 585–592.
2. [56]R. I. Kondor and J. Lafferty, “Diffusion kernels on graphs and other discrete structures,” in Proc. ICML, 2002, pp. 315–322.

- non-parametric kernel learning:**[high computational cost]**
1. [54]
2. [57]J. Zhuang, I. W. Tsang, and S. C. H. Hoi, “SimpleNPKL: Simple non-parametric kernel learning,” in Proc. 26th ICML, Montreal, QC, Canada, 2009.
3. [58]B. Kulis, M. Sustik, and I. Dhillon, “Learning low-rank kernel matrices,” in Proc. 23rd ICML, Pittsburgh, PA, USA, 2006, pp. 505–512.

- semi-supervised and unsupervised kernel learning:
1. [54]
2. [56]
3. [59]S. C. H. Hoi and R. Jin, “Active kernel learning,” in Proc. 25th ICML, Helsinki, Finland, 2008, pp. 400–407.

- multi-label MKL based on OvA:
1. [74]L. Tang, J. Chen, and J. Ye, “On multiple kernel learning with multiple labels,” in Proc. IJCAI, 2009.
2. [22]S. Bucak, R. Jin, and A. Jain, “Multi-label multiple kernel learning by stochastic approximation: Application to visual object recognition,” in Proc. NIPS, 2010, pp. 325–333.
3. [75]A. Zien and S. Cheng, “Multiclass multiple kernel learning,” in Proc. 24th ICML, Corvallis, OR, USA, 2007.
4. [48]S. Ji, L. Sun, R. Jin, and J. Ye, “Multi-label multiple kernel learning,” in Proc. NIPS, 2009.

# MKL optimization techniques: two groups 
- group 1: directly solve the primal problem or the dual problem
1. [33]
2. [16]
3. [30]
4. [9]

- group 2: solve the convex-concave optimization problem by alternating between two steps: the step for updating the kernel combination weights and the step for solving the SVM classifier for the given combination weights.
1. [50] **SILP** problem
2. [43] simpleMKL: **Subgradient Descent** approaches for MKL; **MKL-GL**
3. [12] **MKL-SD**
4. [39] K. Gai, G. Chen, and C. Zhang, “Learning kernels with radiuses of minimum enclosing balls,” in Proc. NIPS, 2010, pp. 649–657. **[MKL-SD]**
5. [23] **MKL-MD** (a mirror descent method)
6. [42] **MKL-level** (an extended level method)
7. [10] **MKL-GL** (group lasso)

- online learning algorithms for MKL: process one training example at each iteration
1. [78] R. Jin, S. C. H. Hoi, and T. Yang, “Online multiple kernel learning: Algorithms and mistake bounds,” in Proc. Int. Conf. ALT, Canberra, ACT, Australia, 2010, pp. 390–404.
2. [21] L. Jie, F. Orabona, M. Fornoni, B. Caputo, and N. Cesa-Bianchi, “OM-2: An online multi-class multi-kernel learning algorithm,” in Proc. IEEE CVPRW, San Francisco, CA, USA, 2010, pp. 43–50.

# experiments: 
three datasets: Caltech 101; Pascal VOC 2007; a subset of ImageNet

AVG： average kernel approach

## classification performance
1. MKL和AVG都优于单核 SVM
2. 当训练集样本很小时（每类有10或20个样本），与AVG相比，L1-MKL性能最差，Lp-MKL与AVG相当；当训练集样本量增大到一定程度时（每类有30个样本），L1-MKL的性能明显优于其他方法
3. 不同的分类算法在迭代过程中的精度变化不一样（文章中图3）
	- simpleMKL迭代很小的步数就收敛了，但是在线性搜索步需要很多迭代，因此总的耗时却很大
	- MKL-SILP震荡比较明显，这由于其贪婪的本质，每次总是选择the most violated constraints 来优化
	![mean average precision for MKLs](/img/blog/mkl/mkl_map_scores.png)

## Number of kernels vs classification accuracy
文章中实验的做法：先用L1-MKL来得到所有kernel的权重并排序，然后通过逐步加入权重有序的核来验证MKL方法的性能。（文章中图4）
![Change in MAP score with respect to the number of base kernels](/img/blog/mkl/map_score_wrt_kernels.png)
上图表明随着核的数量的增加，L2-MKL和AVG方法的精度反而下降了，这表明后面加入的一些week kernels起到的是坏作用，而L1-MKL方法到一定程度之后，就饱和了，精度变化不大了，从另一方面来说，L1-MKL更加的鲁棒。

## computational efficiency
实验结果：
![Total Training Time (Secs), Number of Iterations, and Total Time Spent on Combining the Base Kernels (Secs) for Different MKL Algorithms vs Number of Training Examples](/img/blog/mkl/training_time.png)
上表表明
1. 与L1-MKL方法相比，Lp-MKL需要很小的步数，主要原因在于Lp-MKL拥有光滑的目标函数，更方便优化。
2. 大量的时间主要花费在kernel matrix的组合上，而对于不同的L1-MKL方法而言，差异主要体现在他们在迭代过程中的稀疏性的体现。比如MKL-SILP在优化过程中就产生了稀疏解，因此其实最有效的wrapper algorithm，而SimpleMKL就不用有这样的性质了，所以需要更多的时间。

## evaluation of sparseness
明显L1-MKL是具有很好的稀疏性的，而AVG和Lp-MKL就不具有稀疏性了。

# reference
1. Multiple Kernel Learning for Visual Object Recognition：A Review
2. 作者的[个人主页](http://www.cse.msu.edu/~bucakser/), 里面提供了与本文相关的代码及一些作者的个人信息。