---
title: Euclidean Projection onto the Simplex
date: 2016-10-15 20:30:47
categories: [Optimization]
tag: [Optimization, Machine Learning]
description: 在欧式空间将向量投影到Simplex
---

在ML中，通常通过L1范数约束来达到稀疏性，而对于L1范数约束的求解常用的解法是内点法，这种算法虽然通用，但是针对这一问题有其效率的问题，最近在阅读到一篇文章专门研究在欧式空间下投影到Simplex上的做法，这里总结下来以备忘。

# model 1： Euclidean Projection onto the Simplex
首先确定一下要求解的模型：
$$\min\_w \frac{1}{2}\|w-v\|^2\_2\quad s.t. \quad \sum\_{i=1}^nw\_i = z, w\_i \geq 0$$

> **注**：这里啰嗦一句，上式中的$v$便是我们在欧式空间中待投影的向量，而$w$便是我们要求解的在simplex上与$v$对应的那个向量

下面需要对这个模型进行求解，常用套路便是对上式求Lagrange问题：
$$L(w, \zeta) = \frac{1}{2}\|w-v\|^2 + \theta(\sum\_{i=1}^n w\_i - z) - \zeta \cdot w, \theta\in\mathbb R, \zeta\in \mathbb R\_+^n$$

上式对$w$求偏导得到：
$$\frac{dL}{dw\_i}=w\_i-v\_i+\theta-\zeta\_i=0$$

这里运用到Lagrange变量的性质：当$w_i\gt 0$的时候，我们一定有$\zeta\_i = 0$。因此，当$w\_i\gt 0$时，我们有
$$w\_i = v\_i - \theta + \zeta\_i = v\_i - \theta$$

接着，在一篇文章中(Shalev-Shwartz & Singer, 2006)给出了一个Lemma：对于原始模型，如果有$v\_s\gt v\_j$,那么当$w\_s=0$时，就一定有$w\_j=0$。

> **注：**这里其实揭示了$w$与$v$之间的关系，而下面正是巧妙的运用了这个关系解析的求得了$w$.

根据上面的这个Lemma，我们首先定义一个$\rho$,这里$\rho$表示最终的最优解$w$中大于0的个数，因此有

$$\sum\_{i=1}^n w\_i = \sum\_{i=1}^{\rho} w\_i = \sum\_{i=1}^{\rho}(v\_i - \theta) = z$$

因此，$\theta = \frac{1}{\rho} (\sum\_{i=1}^{\rho} v\_i - z)$.那么，最终的$w$可以得到

$$w\_i = \max\\{v\_i - \theta, 0\\}$$

接下来，我们需要确定的就是上式中的$\rho$了，同样文章(Shalev-Shwartz & Singer, 2006)中给出了求$\rho$的方法：
对于给定的$v$,按照降序来重新排列得到新的序列$\mu$,那么与之对应的$\rho$有以下公式：
$$\rho(z,\mu)=\max\\{j\in[n]:\mu\_j - \frac{1}{j}(\sum\_{r=1}^j \mu\_r - z) > 0\\}$$

那么到这里，这个Model的解析解就得到了。

# model 2: Euclidean Projection onto the ℓ1-Ball

首先确定一下要求解的模型：
$$\min\_w \frac{1}{2}\|w-v\|^2\_2\quad s.t. \quad \|w\|\_1 \leq z$$

在这里，我们可以明确两点：

1. 对于给定的$v$，其对应的1范数应该满足$\|v\|\_1\gt z$,要不然最优的$w$便是$v$本身.

2. 基于1，最优的$w$应该在约束集的边界上取得，因此，可以将原Model中的约束换成$\|w\|\_1 = z$

同时， 这里模型与上面的一个的差异就在于对于$w$的约束不再是大于0了，而是最终的1范数在一个区间内。但是，对于非零的$w$与其对应的$v$之间存在一个巧妙的关系就是：$w\_iv\_i\geq 0$.（这里就不放证明了，可以自行参考论文）

根据上面的这个关系，我们可以先认为设定$u\_i = |v\_i|$, 可以将原模型变换成：
$$\min\_{\beta\in\mathbb R^n}\|\beta-u\|\_2^2 \quad s.t. \quad \|\beta\|\_1 = z \quad \text{and}\quad \beta \geq 0$$
这里的模型就与第一个一样了，然后再求得$\beta$之后，我们就可以得到与之对应的$w\_i = sign(v\_i)\beta\_i$.

# references
1. Efficient Projections onto the ℓ1-Ball for Learning in High Dimensions(ICML'08)
2. Efficient learning of label ranking by soft projections onto polyhedra(JMLR'06)
