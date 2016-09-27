---
title: "CS231n:Neural Networks Part 3: Learning and Evaluation"
date: 2016-08-19 21:22:45
categories: [Machine Learning]
tag: [Machine Learning, CS231n笔记]
description: 
---


本节主要讲了一些关于神经网络的学习与评估方面的知识。

# Gradient checks
1. 使用中心化公式：
$$\frac{df(x)}{dx} = \frac{f(x+h)-f(x-h)}{2h}$$

2. 使用相对误差来比较两者的差异：
$$relative \ error = \frac{|f'_a-f'_n|}{\max(|f'_a|,|f'_n|)}$$

在实际的计算中，对于relative error（re）的评价，主要分一下几种情况：
1） $re > 1e-2$: 出错
2） $1e-2 > re > 1e-4$ : 不舒服
3） $1e-4 > re$: 当函数有不可导点存在时，这样的精度时OK的；但如果没有不可导点时，这个精度还是太高了
4） $1e-7 > re$: OK

课程中还讲解了很多细节的地方：
1. 编程时，要使用双精度来作为变量的声明与定义
2. 目标函数中不可导导致的梯度检查出现问题，需要额外关注，使用少量数据点来解决这一问题
3. 谨慎设置步长 h
4. 网络“预热”后再检查
5. 不让正则化吞没了数据的损失
6. 在做梯度检查时，记得关闭Dropout和数据扩张(augmentations)
7. 检查少量的维度

# Before learning: sanity checks Tips/Tricks

1. 寻找特定状况下的正确损失值
比如CIFAR-10数据集，对于Softmax分类器，初始的期望损失值为2.302，这是因为一个样本被判定为某个个类别（共10个类别）的概率是0.1，所以 $-\ln(0.1) = 2.302$
对于Weston Watkins SVM，假设所有边界都被越多，则损失值为9.

2. 提高正则化强度时导致损失值变大
3. 对于小数据子集过拟合

# Babysitting the learning process
检查整个的学习过程，与**周期**相关的性能指标
这里的**周期**衡量了在训练中每个样本数据都被观察过次数的期望，周期与数据的批尺寸有关

指标有如下一些：
1. 损失函数： 损失值
2. training set和validation set的准确率，可以防止过拟合
3. 权重更新比例： 大概 $1e-3$ 左右
code：
```python
    #assume parameter vector W and its gradient vector dW
    param_scale = np.linalg.norm(W.ravel())
    update = -learning_rate*dW # simple SGD update
    update_scale = np.linalg.norm(update.ravel())
    W += update # the actual update
    print update_scale / param_scale # want ~1e-3
```

4.  检查每层的激活数据及梯度分布    
5. 如果是图像数据，可以将第1层可视化来判断网络的学习情况

# Parameter updates
1. 普通的更新方法(**一阶方法**)： $x += - learning\\_rate * dx$
2. **动量更新**(momentum)：
解释：损失值可以理解为是山的高度（高度的势能：$U = mgh$）
用随机数字初始化参数等同于在某个位置给质点设定初始速度为0，质点的力与梯度的潜在能量有关（$F=-\nabla U$）,质点所受的力就是损失函数的（负）梯度，又$F=ma$，所以（负）梯度与质点加速度成比例
更新公式：
```python
# Momentum update
v = mu * v - learning_rate * dx # integrate velocity
x += v # integrate position
```
	v 初始化为0，超参mu（一般设为0.9）可以通过交叉验证来确定[0.5, 0.9, 0.95, 0.99]

3. **Nesterov 动量**
	Nesterov动量与普通动量有些许不同，最近变得比较流行。在理论上对于凸函数它能得到更好的收敛，在实践中也确实比标准动量表现更好一些。

	Nesterov动量的核心思路是,当参数向量位于某个位置 $x$ 时,观察上面的动量更新公式可以发现,动量部分(忽视带梯度的第二个部分)会通过 mu \* v 稍微改变参数向量。因此,如果要计算梯度，那么可以将未来的近似位置 x + mu \* v 看做是“向前看”,这个点在我们一会儿要停止的位置附近.因此,计算 x + mu \* v 的梯度而不是“旧”位置x的梯度就有意义了.

	可以通过下图来感受一下：
![momentum and Nesterov momentum update](/img/blog/deeplearning/nesterov.jpeg)
	对应的更新公式为：
	```python
	x_ahead = x + mu * v
	# evaluate dx_ahead (the gradient at x_ahead instead of at x)
	v = mu * v - learning_rate * dx_ahead	
	x += v
	```
	然而在实践中，人们更喜欢和普通SGD或上面的动量方法一样简单的表达式。通过对x_ahead = x + mu * v使用变量变换进行改写是可以做到的，然后用x_ahead而不是x来表示上面的更新。也就是说，实际存储的参数向量总是向前一步的那个版本。x_ahead的公式（将其重新命名为x）就变成了：
	```python
	v_prev = v # back this up
	v = mu * v - learning_rate * dx # velocity update stays the same
	x += -mu * v_prev + (1 + mu) * v # position update changes form
	```

4. 学习率退火
	当一个模型的学习率很高时，系统功能过大，参数向量就会无规律地跳动，不能够稳定到损失函数更深更窄的部分去。

	对于学习率的变化，主要有三种方式：
	1）随步数衰变
	2）指数衰减  $\alpha = \alpha_0 e^{-k t}$
	3) $1/t$衰减：$\alpha = \alpha_0 / (1 + kt)$

	在实际运用中，通常使用随步数衰减的Dropout

5. 二阶方法
	牛顿法： $x \leftarrow x - [H f(x)]^{-1} \nabla f(x)$
其中 $H f(x)$ 表示损失函数的局部曲率，它能够让最优化过程在曲率小的时候大步前进，曲率小的时候，小步前进

6. 逐参数适应学习率方法
前面讨论的所有方法都是对学习率进行全局地操作，并且对所有的参数都是一样的。学习率调参是很耗费计算资源的过程，所以很多工作投入到发明能够适应性地对学习率调参的方法，甚至是逐个参数适应学习率调参。很多这些方法依然需要其他的超参数设置，但是其观点是这些方法对于更广范围的超参数比原始的学习率方法有更良好的表现。在本小节我们会介绍一些在实践中可能会遇到的常用适应算法：

	**[Adagrad](http://www.jmlr.org/papers/volume12/duchi11a/duchi11a.pdf)**
	```python
	# Assume the gradient dx and parameter vector x
	cache += dx**2
	x += - learning_rate * dx / (np.sqrt(cache) + eps)
	```
	注意，变量cache的尺寸和梯度矩阵的尺寸是一样的，还跟踪了每个参数的梯度的平方和。这个一会儿将用来归一化参数更新步长，归一化是逐元素进行的。注意，接收到高梯度值的权重更新的效果被减弱，而接收到低梯度值的权重的更新效果将会增强。有趣的是平方根的操作非常重要，如果去掉，算法的表现将会糟糕很多。用于平滑的式子eps（一般设为1e-4到1e-8之间）是防止出现除以0的情况。Adagrad的一个缺点是，在深度学习中单调的学习率被证明通常过于激进且过早停止学习。

	**RMSprop** 
	是一个非常高效，但没有公开发表的适应性学习率方法。有趣的是，每个使用这个方法的人在他们的论文中都引用自Geoff Hinton的Coursera课程的第六课的[第29页PPT](http://www.cs.toronto.edu/~tijmen/csc321/slides/lecture_slides_lec6.pdf)。这个方法用一种很简单的方式修改了Adagrad方法，让它不那么激进，单调地降低了学习率。具体说来，就是它使用了一个梯度平方的滑动平均：
	```python
	cache =  decay_rate * cache + (1 - decay_rate) * dx**2
	x += - learning_rate * dx / (np.sqrt(cache) + eps)
	```

	在上面的代码中，decay_rate是一个超参数，常用的值是[0.9,0.99,0.999]。其中x+=和Adagrad中是一样的，但是cache变量是不同的。因此，RMSProp仍然是基于梯度的大小来对每个权重的学习率进行修改，这同样效果不错。但是和Adagrad不同，其更新不会让学习率单调变小。

	**[Adam](http://arxiv.org/pdf/1412.6980v8.pdf)** 
	Adam是最近才提出的一种更新方法，它看起来像是RMSProp的动量版。简化的代码是下面这样：
	```python
	m = beta1*m + (1-beta1)*dx
	v = beta2*v + (1-beta2)*(dx**2)
	x += - learning_rate * m / (np.sqrt(v) + eps)
	```

	注意这个更新方法看起来真的和RMSProp很像，除了使用的是平滑版的梯度m，而不是用的原始梯度向量dx。论文中推荐的参数值eps=1e-8, beta1=0.9, beta2=0.999。在实际操作中，我们推荐Adam作为默认的算法，一般而言跑起来比RMSProp要好一点。但是也可以试试SGD+Nesterov动量。完整的Adam更新算法也包含了一个偏置（bias）矫正机制，因为m,v两个矩阵初始为0，在没有完全热身之前存在偏差，需要采取一些补偿措施。建议读者可以阅读论文查看细节，或者课程的PPT。

	课程中给出两个动图来显示不同的学习率下优化性能的比较：
![Contours of a loss surface and time evolution of different optimization algorithms](/img/blog/deeplearning/opt2.gif)
![A visualization of a saddle point in the optimization landscape, where the curvature along different dimension has different signs](/img/blog/deeplearning/opt1.gif)

# Hyperparameter optimization
1. 实现
一个具体的设计是用仆程序持续地随机设置参数然后进行最优化。在训练过程中，仆程序会对每个周期后验证集的准确率进行监控，然后向文件系统写下一个模型的记录点（记录点中有各种各样的训练统计数据，比如随着时间的损失值变化等），这个文件系统最好是可共享的。在文件名中最好包含验证集的算法表现，这样就能方便地查找和排序了。然后还有一个主程序，它可以启动或者结束计算集群中的仆程序，有时候也可能根据条件查看仆程序写下的记录点，输出它们的训练统计数据等。

2. 比起交叉验证最好使用一个验证集

3. 超参数范围：
学习率 ： learning_rate = 10 ** uniform(-6, 1)

4. 随机搜索优于网格搜索
[Random Search for Hyper-Parameter Optimization](http://www.jmlr.org/papers/volume13/bergstra12a/bergstra12a.pdf)中说“随机选择比网格化的选择更加有效”，而且在实践中也更容易实现

5. 对于边界上的最优值要小心
6. 从粗到细地分阶段搜索
7. 贝叶斯超参数最优化

# Evaluation
模型的集成
1. 同一个模型，不同的初始化
2. 在交叉验证中发现最好的模型
3. 一个模型设置多个记录点
4. 在训练的时候跑参数的平均值
模型集成的一个劣势就是在测试数据的时候会花费更多时间。最近Geoff Hinton在[“Dark Knowledge”](https://www.youtube.com/watch?v=EK61htlw8hY)上的工作很有启发：其思路是通过将集成似然估计纳入到修改的目标函数中，从一个好的集成中抽出一个单独模型。

# references
1. [CS231n:Neural Networks Part 3: Learning and Evaluation](http://cs231n.github.io/neural-networks-3/)
2. [知乎：CS231n课程笔记翻译：神经网络笔记3（下）](https://zhuanlan.zhihu.com/p/21798784?refer=intelligentunit) 翻的很棒！