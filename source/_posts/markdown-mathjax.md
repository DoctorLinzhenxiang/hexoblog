---
title: Hexo中一些蛋疼的数学公式编辑
date: 2016-06-09 11:24:40
categories: [Hexo]
tag: [Hexo]
description: 如题~
---

由于写的博客中有很多公式，在线下编辑的时候都能正常显示，可是发布了之后出现了各种蛋疼的效果，折腾了好久，总结下来以备忘。

# 多行公式显示

\begin{align} \\\\
\max_{w,b} \quad & \gamma \\\\
s.t. \quad &y_i(\frac{w}{||w||}x_i+\frac{b}{||w||})\geq \gamma, i=1,2,...,N
\end{align}

```
code:
\begin{align} \\\\
\max_{w,b} \quad & \gamma \\\\
s.t. \quad &y_i(\frac{w}{||w||}x_i+\frac{b}{||w||})\geq \gamma, i=1,2,...,N
\end{align}
```
注意，这里在最后不能加 `$` ，加上之后会显示不正常，然后每一行公式的最后要加四个 `\` ,我想原因是hexo中对于 `\` 是需要通过 `\` 转义的,所以两个 `\\`经过转义后等于一个 `\`
另外，对于绝对值的写法在这里也失效了，只能简单粗暴的用四个 `|` 来解决了

# 对于下标的转义

$$L(w,b,\alpha) = \frac{1}{2}||w||^2 - \sum\_{i=1}^N\alpha\_i y\_i (w \cdot x\_i + b) + \sum\_{i=1}^N\alpha\_i$$

```
code:
$$L(w,b,\alpha) = \frac{1}{2}||w||^2 - \sum\_{i=1}^N\alpha\_i y\_i (w \cdot x\_i + b) + \sum\_{i=1}^N\alpha\_i$$
```
这里出现了两个`\sum\_{i=1}^N`如果不对下标进行转义，写成`\sum_{i=1}^N`，就不能正常显示，不知道原因。只能先加上解决问题了

# 对于$*$上标的转义 
$$\lambda^\*$$

```
code:
$$\lambda^\*$$
```