---
title: methods on optimizing large-scale SVM problems
date: 2016-11-03 11:14:47
categories: [Machine Learning]
tag: [Optimization, Machine Learning]
description: 大规模SVM问题的求解
---

# Backgrounds
首先，直入主题，SVM原问题模型：

\begin{align}
\min\_{w, b} \quad & \frac{1}{2} \|w\|^2 + C\sum\_{i=1}^n\eta\_i\\\\
s.t. \quad & y\_i (w^T \phi(x\_i)+b) \geq 1-\eta\_i,\\\\
& \eta\_i\geq 0, \quad i=1,2,...,n
\end{align}

接着，我们对原模型求Lagrange对偶问题，并对$w,b$分别求偏导，置为0，得到结果带到对偶问题中得到对应的对偶问题：

\begin{align}
\min\_{\alpha} \quad & \frac{1}{2} \alpha^T Q \alpha - e^T \alpha \\\\
s.t. \quad & y\alpha = 0,\\\\
& 0 \leq \alpha\_i\leq C, \quad i=1,2,...,n
\end{align}
其中$Q\_{i,j}=y\_iy\_jK(x\_i, x\_j)$.

到这里，我们得到的问题就是一个凸二次规划问题。通常我们可以用通用的优化工具包来求解，但是当样本规模很大时，一方面核矩阵会变得非常的大（核的维度与样本个数相关），其实对于工业界而言百万级的数据量是正常的，而这对于svm来说是很可怕的，所以这也是svm在工业界用的很少的主要原因；另一方面需要求解的变量也会随之增加，求解的时效性也是一个问题。所以，这个时候就产生了很多针对SVM问题的优化方法，比如著名的**SMO算法**！其实，关于SMO算法，是一类方法的一种极限情况，这一类方法的主要思想是：首先，我们从问题的本身出发，对于SVM原问题与其对偶问题，通常情况下，两者的解之间是存在一个对偶间隔的，只有当两者的对偶间隔消失时，对偶问题的解才与原问题的解是等价的，而对偶间隔的消失的充要条件是对偶问题的解都满足KKT条件，所以，大家就想，我们是不是可以判断对偶问题的解是否都满足了KKT条件来间接判断我们的目标是否已最小化。于是，这一类方法就是通过不断更新部分的$\alpha$来使他们都满足KKT条件。在每一次的更新中，都是选择一部分的$\alpha$来更新，这样就能很有效的减少了核矩阵的规模（只需要选择与这些$\alpha$相关的核矩阵值就可以）。这一类方法通常被称为**Decomposition Methods**.

# Decomposition Methods

Decompositon methods通常将需要求解的$\alpha$分解成两个部分：一个inactive部分，一个active部分（这一部分称为**working set**）。具体的算法流程为：

> 
while the optimality conditions are violated
1. select q variables for the working set B. The remaining (l-q) variables are fixed at their current value.
2. decompose problem and solve QP-subproblem: optimize $W(\alpha)$ on B.
terminate and return $\alpha$

根绝上面的算法流程，我们主要需要完成两个任务：
1. 找到合理的working set
2. 求解分解后的子问题
下面逐个进行介绍。

## 分解后的子问题

对于这个问题，我们可以通过对偶问题的KKT条件来推导得到。

对偶问题的KKT条件：
\begin{align}
& g(\alpha) + (\lambda^{eq} \textbf{y} - \lambda^{lo} + \lambda^{up}) = 0 \\\\
\forall i \in [1,...,n]: \quad & \lambda\_i^{lo}(-\alpha\_i) = 0\\\\
\forall i \in [1,...,n]: \quad & \lambda\_i^{up}(\alpha\_i - C) = 0\\\\
& \lambda^{lo}  \geq \textbf{0} \\\\
& \lambda^{up}  \geq  \textbf{0} \\\\
& \alpha^T\textbf{y} = 0 \\\\
& 0 \leq\alpha \leq C\textbf{1}
\end{align}
其中$g(\alpha)$是对偶问题目标函数关于$\alpha$的偏导数。$g(\alpha) = -\textbf{1} + Q \alpha$。

根据decomposition methods的算法，我们将$\alpha$分解为两个set:
$$\alpha = 
\begin{vmatrix}
\alpha_B \\\\
\alpha_N
\end{vmatrix}
$$
其中，$\alpha_B$是active set，即为需要更新的set；$\alpha_N$是inactive set，即为暂时固定的set。那么对应的$\textbf{y}, Q$分解为：
$$\textbf{y}=
\begin{vmatrix}
\textbf{y}\_B\\\\
\textbf{y}\_N
\end{vmatrix}
$$
$$Q = 
\begin{vmatrix}
Q\_{BB} & Q\_{BN} \\\\
Q\_{NB} & Q\_{NN}
\end{vmatrix}
$$
由于$Q$ 是对称的，所以这里有$Q\_{BN} = Q\_{NB}^T$.将上面分解的形式代入到对偶问题中得到：
\begin{align}
\min\_{\alpha} \quad & - \alpha\_B^T(\textbf{1} - Q\_{BN} \alpha\_N) + \frac{1}{2}\alpha^T\_B Q\_{BB} \alpha\_B + \frac{1}{2}\alpha^T\_N Q\_{NN} \alpha\_N - \alpha\_N^T \textbf{1} \\\\
s.t. \quad & \alpha_B^Ty\_B + \alpha\_N^T y\_N = 0 \\\\
& 0\leq \alpha \leq C \textbf{1}
\end{align}
这里由于$\alpha_N$是固定的，所以$\frac{1}{2}\alpha^T\_N Q\_{NN} \alpha\_N$ 和 $\alpha\_N^T \textbf{1}$都是固定的值，可以省去。所以上式可以化简为：
\begin{align}
\min\_{\alpha} \quad & \frac{1}{2}\alpha^T\_B Q\_{BB} \alpha\_B - \alpha\_B^T(\textbf{e}\_B - Q\_{BN} \alpha\_N) \\\\
s.t. \quad & \alpha\_B^Ty\_B + \alpha_N^T y\_N = 0 \\\\
& 0 \leq (\alpha\_B)\_i \leq C, \quad i = 1,\cdots, q
\end{align}
此时，我们需要求解上面一个缩小版的对偶问题，来更新$\alpha_B$。这样就解决了数据量很大时不能求解的问题。

## 合理的working set的选择
对于working set的选择，我们的目标是希望选择一个working set使得当前的迭代能够让目标函数有尽量大的下降。[Zoutendijk (1970)](https://www.amazon.com/Methods-feasible-directions-nonlinear-programming/dp/B0006AWLPG)提出了一个一阶近似的方法。他的思想主要是找到一个下降最陡峭的可行方向集$d$，其中仅有$q$个非零元素。那个对这些非零$d$对应的点即表示该轮被选中的working set中的元素。
优化目标：
\begin{align}
\min \quad & V(d) =  g(\alpha^{(t)})^T\textbf{d} \\\\
s.t. \quad & \textbf{y}^T\textbf{d} = 0 \\\\
& d\_i  \geq  0 \quad \forall i:\alpha_i = 0\\\\
& d\_i\leq 0 \quad \forall i: \alpha\_i = C \\\\
& -\textbf{1} \leq \textbf{d} \leq \textbf{1} \\\\
& |\{d\_i : d\_i\neq 0\}| = q
\end{align}
其中，$\alpha^{(t)}$表示第t轮的$\alpha$值。下面的前三个约束确保了下降的方向是沿着对偶问题中的等式约束进行投影的，并且遵循了active bound constraints.倒数第二个约束对d进行了规范化，确保优化问题well-posed.最后一个约束确保了这个求解的方向只能包含q个激活变量。这里需要约束$q$是一个偶数。

这一选择working set的方法最早是由[Joachims（1998）](https://www.cs.cornell.edu/people/tj/publications/joachims_99a.pdf)提出，后来[Change等人(2000)](http://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=857780)指出上面最后一个约束并不是总是满足，因此将其改成了：
$$|\{d_i : d_i\neq 0\}| \leq q$$
这也是SVM开源工具包 [$\text{SVM}^{light}$](http://svmlight.joachims.org/) 的求解方案。

至此， 这里decomposition methods的两个任务就都介绍完了。当然，或许对于working set的选择，还是会感到很迷惑，不知道为什么要这么选择，下面我就依照[PALAGI et al.](http://china.tandfonline.com/doi/pdf/10.1080/10556780512331318209?needAccess=true&)里面的讲解思路来复述一下。

首先，对于一个满足条件的$\alpha$，我们可以表示出其对应的上下界的集合：

$$L(\alpha) = \\{i:\alpha_i=0\\},\quad U(\alpha) = \\{i:\alpha_i=C\\}$$

对于一个可行解 $\alpha^\*$ ,必存在一个标量 $\lambda^\*$ ,结合上面对偶问题的KKT条件，我们有：

$$
(\nabla f(\alpha^\*))\_i + \lambda^\* y\_i =
\begin{cases}
\geq 0 & \text{if} \quad i \in L(\alpha^\*) \\\\
\leq 0 & \text{if} \quad i \in U(\alpha^\*) \\\\
= 0 & \text{if} \quad i \notin L(\alpha^\*)\cup U(\alpha^\*) 
\end{cases}
$$

> **注：**
这里根绝上面对偶问题的KKT条件：
\begin{align}
& g(\alpha) + (\lambda^{eq} \textbf{y} - \lambda^{lo} + \lambda^{up}) = 0 \\\\
\forall i \in [1,...,n]: \quad & \lambda\_i^{lo}(-\alpha\_i) = 0\\\\
\forall i \in [1,...,n]: \quad & \lambda\_i^{up}(\alpha\_i - C) = 0\\\\
& \lambda^{lo}  \geq \textbf{0} \\\\
& \lambda^{up}  \geq  \textbf{0} \\\\
\end{align}
所以，当$i \in L(\alpha^*)$时，有$\alpha_i=0$.所以，$\lambda^{lo}$可以是任意正值，且$\lambda_i^{up}=0$,那么我们有$g(\alpha) + \lambda^{eq} \textbf{y} = \lambda^{lo} \geq 0$，这就是上式的第一条结论。后面两个可以类似的推导得到。

根据上面结论，我们可以变换一种写法：
$$L^-(\alpha) = \{i\in L(\alpha): y_i\lt 0\}, \quad L^+(\alpha) = \{i\in L(\alpha): y_i \gt 0\}$$
$$U^-(\alpha) = \{i\in U(\alpha): y_i\lt 0\}, \quad U^+(\alpha) = \{i\in U(\alpha): y_i \gt 0\}$$

随之，我们就可以得到一个关于Optimality Conditions的定理：

$$\lambda^\*\geq -\frac{(\nabla f(\alpha^\*))\_i}{y\_i} \quad \forall i \in L^+(\alpha^\*)\cup U^-(\alpha^\*)$$

$$\lambda^\*\leq -\frac{(\nabla f(\alpha^\*))\_i}{y\_i} \quad \forall i \in L^-(\alpha^\*)\cup U^+(\alpha^\*)$$

$$\lambda^\* =  -\frac{(\nabla f(\alpha^\*))\_i}{y\_i} \quad \forall i \notin L(\alpha^\*)\cup U(\alpha^\*)$$

对应的，我们可以将上面定理中条件定义成：
$$R(\alpha) = L^+(\alpha^\*)\cup U^-(\alpha^\*) \cup \{i: 0 \lt \alpha\_i \lt C\}$$
$$S(\alpha) = L^-(\alpha^\*)\cup U^+(\alpha^\*) \cup \{i: 0 \lt \alpha\_i \lt C\}$$
其中，$R(\alpha)$被称为”bottom“ 候选集，$S(\alpha)$被称为”Top“候选集。这其实也就是选择working set元素的一个指导。

根据上面的一个定理，我们可以得到另一个定理：
对于可行解$\alpha^\*$,其中，$i\in R(\alpha^\*), j\in S(\alpha^\*)$,必然不会存在如下的形式
$$-\frac{(\nabla f(\alpha^\*))\_i}{y\_i} \gt -\frac{(\nabla f(\alpha^\*))\_j}{y\_j}$$

接着，对于每一对$i \in R(\alpha),j \in S(\alpha)$,方向$d$有：
$$d\_i = \frac{1}{y\_i}, \quad d\_j = -\frac{1}{y\_j}, \text{and} \quad d\_l \leq 0 \quad \text{for} \quad l\neq i,j$$

另外，我们可以定义：
$$\mathcal I(\alpha) = \left \\{ i: i=\arg \max\_{t\in R(\alpha)} - \frac{\nabla f(\alpha)\_t}{y\_t} \right \\}$$
$$\mathcal J(\alpha) = \left \\{ j: j=\arg \max\_{t\in S(\alpha)} - \frac{\nabla f(\alpha)\_t}{y\_t} \right \\}$$

最终，我们可以得到选择working set的算法：
![working set selection algorithm](/img/blog/large_scale_svm/working_set_selection.png)

# 后序改进的工作
对于SVM优化问题的探讨，在2000年左右是一个高峰，有点类似现在deeplearning，基于这一个想法的相关改进工作也产生了很多的论文，其中，主要基于两个方向，一个是对分解子问题的改进，另一个就是对working set选择的改进。

对于分解子问题的改进：
1. shrinking：由于通常Support Vector是少量的，其中由于noise的因素，某些$\alpha_i$会到约束的上界$C$（这些点被称为"bounded support vectors"），假设先验的知道哪些点是BSV，那么我们就可以将这些点固定为$C$,这样，子问题就会进一步的缩小规模。[Joachims 的工作](https://www.cs.cornell.edu/people/tj/publications/joachims_99a.pdf)
2. 关于 $\text{SVM}^{light}$ 的工作的渐进收敛性的证明，需要基于如下的假设：
$$\min\_{I:|I|\leq q}\quad (eig\_{\min}(Q\_{II})) \gt 0$$
其中，$I$是一个样本子集index，$Q_{II}$为分界后的子矩阵。这一要求能够使得目标函数是强凸的，以满足定理证明的需要。后续的researcher将其进行了改进，应用了一个proximal point的技术，在目标函数后面添加了一项：$\tau\|\alpha_B - \alpha_B^k\|^2$.具体的证明可以参考[Palagi et al.](http://china.tandfonline.com/doi/pdf/10.1080/10556780512331318209?needAccess=true&)

对于working set选择的改进
1. 前面的方法运用的是一阶信息，[Fan et al.](http://www.jmlr.org/papers/volume6/fan05a/fan05a.pdf)运用了二阶的信息

# References
1. Working set selection using second order information for training support vector machines
2. On the convergence of a modified version of SVM light algorithm
3. Making Large-Scale SVM Learning Practical
4. On the Convergence of the Decomposition Method for Support Vector Machines
5. Combining DC Algorithms (DCAs) and Decomposition Techniques for the Training of Nonpositive–Semidefinite Kernels
6. The Analysis of Decomposition Methods for Support Vector Machines
