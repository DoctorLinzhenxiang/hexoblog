---
title: 在CVX中使用Mosek
date: 2016-03-11 23:21:47
categories: MATLAB
tag: matlab
description: 如题~
---

在cvx中默认的[solver][1]是SDT3，解决一般的问题没有问题，但是发现涉及到高维向量（e.p. 求解1000+维向量）的求解时，会很慢，所以尝试换一个solver，以改善性能。目前尝试了Mosek，感觉是比SDT3快点的。搞了一个晚上，总结一下。

# 申请academic license
因为非professional版本的cvx不包含Mosek，所以需要购买，但是对于科研用途，cvx会提供一个license，期限好像是一年。需要填个[表格][2]就ok了。

# Mosek安装
在收到cvx发的邮件后，按照Mosek安装教程[MOSEK and CVX][3]上的步骤执行就好了。
在安装好后，可以通过mosekopt语句来验证是否已经安装成功

# Mosek使用
在确定安装成功后，可以按照[Using MOSEK with CVX¶][4]来设置使用Mosek就可以了。
> ***note:***
不过这里需要注意的是，我们在使用Mosek的时候，并不需要按照Mosek官网提供的user's guide来写优化代码，还是按照cvx的guide来写就可以了;另外，我是将Mosek设置成default的，不知道什么原因，由于之前是使用的SDT3，即使选择了Mosek作为当前的solver，都没用，所以所幸将Mosek设置成了default。


对了，这里还有个资源[How to use MOSEK in CVX?][5]


[1]: http://cvxr.com/cvx/doc/solver.html
[2]: http://cvxr.com/cvx/academic/
[3]: http://cvxr.com/cvx/mosek/
[4]: http://cvxr.com/cvx/doc/mosek.html#mosek
[5]: http://ask.cvxr.com/t/how-to-use-mosek-in-cvx/28
