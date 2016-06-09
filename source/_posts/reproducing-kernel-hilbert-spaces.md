---
title: 再生核希尔伯特空间
date: 2016-05-24 15:24:45
categories: [Mathematics]
tag: [mathematics]
description: 与再生核希尔伯特空间相关的一些定义及自己的理解
---

# 一些定义
1. 向量空间(vector space)： 满足加法和标量乘操作的集合
2. 范数向量空间(normed vector space)： 定义了向量长度的向量空间，这样我们就可以衡量向量间的长度了。
3. 度量空间(metric space)： 定义了两个点的距离的**集合**（这里不必须是向量空间）。范数向量空间是度量空间，但是度量空间不一定是范数向量空间。
4. Banach 空间： 一个完备的范数向量空间。一个**完备**的空间指没有缺失的元素（所有的*柯西序列都是收敛的*）。
5. 内积空间(inner product space)
内积空间$(\mathcal{V}, \langle., .\rangle)$是在域$\mathbb{F}$上可进行$\langle., .\rangle : \mathcal{V}\times \mathcal{V} \to \mathbb{F}$运算法操作的向量空间，其满足三个原则：
1)共轭对称：$\langle \mathbf{x}, \mathbf{y} \rangle = \overline{\langle \mathbf{y}, \mathbf{x} \rangle}$
2)第一个变量满足线性性：$\langle a\mathbf{x}, \mathbf{y} \rangle = a \langle \mathbf{x}, \mathbf{y} \rangle$和$\langle \mathbf{x} + \mathbf{z}, \mathbf{y} \rangle = \langle \mathbf{x}, \mathbf{y} \rangle + \langle \mathbf{z}, \mathbf{y} \rangle$
3)正定性：$\langle \mathbf{x},\mathbf{x} \rangle \geq 0$其中等式只有在$\mathbf{x}=0$时取到。

    > 注：
    1. 对于内积空间，我们可以推导出范数向量空间，即$l^2$范数空间，但是反过来就不一定成立了，因为如$l^1$范数，$||\mathbf{x}||\_{1} = \sum\_{i = 1}^n |x\_i|$就不能满足内积空间
    2. An inner product space is also known as a **pre-Hilbert space**, meaning that it is one step behind a Hilbert space

6. 希尔伯特空间： 当一个内积空间满足通过内积空间可推导出范数空间(赋范空间)，并且是完备的，那么这个内积空间就是希尔伯特空间。

    > 对于常见的$\mathbb R^n$,满足内积运算，能够推导出$l^2$范数，且是完备的，所以是希尔伯特空间。

    另外希尔伯特空间是一个函数空间，即空间中每个元素都是一个函数，所以在希尔伯特空间中的函数可以理解为泛函（functional）

7. 再生核：
$\mathcal{X}$是非空集，$\mathcal{H}$是定义在$\mathcal{X}$上的希尔伯特空间。核$\mathcal{k}$满足下面的两条性质就称为$\mathcal{H}$的再生核：
1) 对于每个$x_0 \in \mathcal{X}$, $k(y, x_0)$作为$y$的函数属于空间$\mathcal{H}$
2) reproducing property: 对于每个$x_0 \in \mathcal{X}$和$f \in \mathcal{H}$,有$f(x\_0) = \langle f(.), k(.,x\_0) \rangle\_\mathcal{H}$

    > 对于再生核的理解，首先性质1表示在$\mathcal{X}$中的每个点都可以通过核函数$k$来映射到希尔伯特空间$\mathcal{H}$中，性质2，表示，对于任意的属于$\mathcal{H}$的核函数都可以在$\mathcal{H}$中找到一个函数$f$(注意希尔伯特空间是一个函数空间)来重构出一个新的函数（这里如果函数$f$是一个核函数的话，会再生出一个核函数）

8. 再生核希尔伯特空间：
A Hilbert space $\mathcal{H}$ of complex-valued functions on a nonempty set $\mathcal{X}$ is called a reproducing kernel Hilbert space if the point evaluation functional $L_x$ is a bounded (equivalently, continuous) linear operator for all $x\in \mathcal{X}$

    补充定义：
    **Point evaluation functional** ：
    The point evaluation functional $L_x:\mathcal{H} → \mathbb{C}$ at a point $x\in \mathcal{X}$ is defined as $L_x(f):=f(x)$.
    
    **Linear operator (Linear map)** :
    Let $f:V→W$ be a function between two vector spaces $V,W$ over the same field $\mathbb F$. $f$ is called a linear operator if the following two conditions are satisfied for all $x,y\in \mathcal{V}$ and $a \in \mathbb{F}$.
    1)$f(u+v)=f(u)+f(v)$
    2)$f(au)=af(u)$
    
    **Bounded linear operator** :
    Let $f:\mathcal{V}→\mathcal{W}$ be a linear operator between two normed vector spaces $(\mathcal{V},||.||\_{\mathcal{V}})$ and $(\mathcal{W},||.||\_{\mathcal{W}})$, $f$ is called a bounded linear operator if there exists some $M>0$ such that for all $\mathbb u\in \mathcal{V}$,$||f(\mathbb u)||\_{\mathcal{W}} \leq M ||\mathbb u||\_{\mathcal{V}}$

9. 定理：
一个希尔伯特空间$\mathcal{H}$是一个再生核希尔伯特空间，当且仅当它有一个再生核（证明要见参考1）

10. 对于给定的希尔伯特空间，再生核是唯一的。


# 参考
1. [a-tutorial-introduction-to-reproducing-Kernel-Hilbert-Spaces](http://sadeepj.blogspot.com/)3篇有关希尔伯特空间的系列文章，很赞！不过需要翻墙~
2. [Free Mind](http://blog.pluskid.org/?p=723)



