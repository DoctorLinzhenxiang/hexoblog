---
title: Goldstein-Armijo Rule and backtracking line search
date: 2016-06-28 23:21:47
categories: Optimization
tag: Optimization
description: Goldstein-Armijo规则以及回溯线性搜索
---

在迭代型算法的优化问题中，我们经常会看到用Armijo Rule来更新步长，找到一个可行点进入下一次迭代。下面将我对Armijo的理解整理下来。

# Goldstein-Armijo Rule
我们这里以梯度方法为例：
> 初始点 $x\_0 \in R^n$
迭代 ： $ x\_{k+1} = x\_k - h\_k f'(x\_k), k=0,1,... $

这里$h_k$表示步长（大于0），$f'(x_k)$表示函数$f(x)$在$x_k$这一点的梯度方向。前面为负表示负梯度方向为函数下降最快的方向，至于为什么在这一点会下降最快，其实是有很好的理论证明的，后面有时间整理出来。
那么，对于步长的确定可以有很多方法，最简单的一种便是提前预定义好：
$$h\_k = h > 0$$
$$h\_k = \frac{h}{\sqrt{k+1}}$$
这种方法在凸优化的框架下使用较多，主要由于凸优化函数的变化趋势是可预测的。
另外还有一种非常理想化的方法：
$$h\_k = arg \min\_{h\geq0}f(x\_k - h f'(x\_k))$$
显然，这种方法在实际的运用中属于空中楼阁，并没有可操作性。
下面最最常用的便是Armijo Rule了：
> 
Find $x_{k+1} = x_k - h f'(x_k)$ such that
> 
$$\alpha <f'(x_k), x\_k - x\_{k+1}> \quad \leq \quad f(x\_k) - f(x\_{k+1}) \tag 1$$
> 
$$\beta <f'(x\_k), x\_k - x\_{k+1}> \quad \geq \quad f(x\_k) - f(x\_{k+1}) \tag 2$$
where $0 < \alpha < \beta < 1$ are some fixed parameters

对于这个规则，可能咋看并不能理解他的意图，下面通过一段证明来看一下这个规则的作用。

# 证明
假设有如下问题
$$\min_{x\in R^n} f(x)$$
其中$f\in C^{1,1}_L(R^n)$。并且，我们这里假设函数有下界。
> 这里$f\in C^{1,1}_L(R^n)$表示函数$f(x)$在$R^n$上是一阶连续可微的，并且其一阶导满足Lipschitz 连续性，参数为L,具体的公式如下
$$|f(y)-f(x)-<f'(x), y-x>| \quad \leq \quad \frac{L}{2} ||y-x||^2$$

对于一个gradient step，我们有

\begin{align}
f(y) & \leq  f(x) + <f'(x), y-x> + \frac{L}{2}\|y-x\|^2 \\\
& = f(x) - h\|f'(x)\|^2 + \frac{h^2}{2}L\|f'(x)\|^2 \\\
& = f(x) - h(1-\frac{h}{2}L)\|f'(x)\|^2 \tag 3
\end{align}

所以，为了使得我们的目标函数值有尽大程度的减小，我们需要求解下面的一维问题：
$$\Delta(h) = -h(1-\frac{h}{2}L) \to \min_h$$
对上式求导，得到$\Delta'(h)=hL-1=0$,所以$h^* = \frac{1}{L}$,由于$\Delta''(h)=L \gt 0$,所以此时取得最小值。这样，将其代入上式，我们就能够得到单步梯度下降，目标函数值至少下降多少的形式：
$$f(y)\leq f(x)-\frac{1}{2L}\|f'(x)\|^2 \tag 4$$

下面，我们来看一下Armijo Rule规定的两个式子与这目标函数值最小下降之间的关系。
首先，令$x_{k+1} = x_k - h_k f'(x_k)$,对于固定步长策略$h_k = h$,我们有
$$f(x\_k) - f(x\_{k+1}) \geq h(1-\frac{h}{2}L)\|f'(x)\|^2$$
所以当我们选择$h_k = \frac{2\alpha}{L}, \alpha \in (0, 1)$时，有

$$f(x\_k) - f(x\_{k+1}) \geq \frac{2}{L}\alpha (1-\alpha)\|f'(x)\|^2 \tag 5$$

显然，当$\alpha = \frac{1}{2}$时，取得最大值，即为前面(4)式等价。
接着，我们考虑Armijo Rule提到的两个式子：
首先，我们考虑公式（2），有
$$f(x\_k)-f(x\_{k+1}) \quad \leq \quad \beta <f'(x\_k), x\_k-x\_{k+1}>\quad = \quad\beta h\_k \|f'(x)\|^2$$
结合公式（3），我们有
$$h_k \geq \frac{2}{L}(1-\beta) \tag 6$$
接着，我们考虑公式（1），有

$$f(x\_k)-f(x\_{k+1}) \quad \geq \quad \alpha <f'(x\_k), x\_k-x\_{k+1}>\quad = \quad\alpha h\_k \|f'(x)\|^2$$

结合不等式（6），我们有

$$f(x\_k)-f(x\_{k+1}) \quad \geq \quad \frac{2}{L}\alpha(1-\beta)\|f'(x\_k)\|^2 \tag 7$$

结合公式（4）（5）（7），容易得到
$$\frac{2}{L}\alpha(1-\beta) \leq \frac{2}{L}\alpha(1-\alpha) \leq \frac{1}{2L}, 0\leq \alpha \leq \beta \leq 1$$
所以，这表明对于给定的$\alpha,\beta$，只要满足公式（1）（2），就至少有公式（7）的下降程度，也就更趋近于最大下降（相当于满足了我们期望的下降）。

# reference
[1] : introductory lectures on convex optimization: a basic course(26页), Yurii Nesterov








