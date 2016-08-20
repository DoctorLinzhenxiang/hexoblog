---
title: "CS231n:Neural Networks Part 2: Setting up the Data and the Loss"
date: 2016-08-18 21:22:45
categories: [Machine Learning]
tag: [Machine Learning, CS231n笔记]
description: 
---

本节主要介绍了样本数据预处理，权重矩阵的初始化，规则化技术，以及一些损失函数。

# Setting up the data and the model

## Data Preprocessing
1. 均值减法

2. 归一化： 
零中心化 + 每个维度/std（x, axis = 0）
每个维度做归一化，使得其值最大为1，最小为-1

3. PCA
	首先将数据去中心化，然后求数据的协方差矩阵，并进行奇异值分解，接着可以得到原始样本数据去相关性后的新样本数据，同时，如果我们想要对数据进行降维处理，只需要选择奇异值分解得到的左特征矩阵的前n个特征向量来去相关性计算就可以了。
	code：
    ```python
    # Assume input data matrix X of size [N x D]
    X -= np.mean(X, axis = 0) # zero-center the data (important)
    cov = np.dot(X.T, X) / X.shape[0] # get the data covariance matrix
    U,S,V = np.linalg.svd(cov)
    Xrot = np.dot(X, U) # decorrelate the data
    Xrot_reduced = np.dot(X, U[:,:100]) # Xrot_reduced becomes [N x 100]
    ```

4. 白化
白化操作实际上是在去相关性的数据样本上除以他们对应的特征值矩阵，进行归一化。
几何解释：当输入数据服从一个多元高斯分布，那么白化就是将这些样本数据转变成服从一个零均值，单位协方差矩阵的高斯分布。
code：
    ```python
    # whiten the data:
    # divide by the eigenvalues (which are square roots of the singular values)
    Xwhite = Xrot / np.sqrt(S + 1e-5)
    ```

下面三幅图片分别描述原始数据，去相关性的数据，和白化后的数据。

![](http://cs231n.github.io/assets/nn2/prepro2.jpeg)

在实际的操作中，对于卷积神经网络，PCA/白化技术运用的并不多，通常会将数据进行零中心化处理。

## Weight Initialization
1. 错误：全零初始化，每个单元都一致，造成了对称性。

2. 小随机值初始化： 非常接近0但又不等于0

3. 使用 $1/sqrt(n)$ 来校准方差： 
理论推导：
考虑内积$s=\sum_i^n w_i x_i$, 计算 $s$ 的方差：

	\begin{align}
	Var(s) &= Var(\sum_i^n w_ix_i) \\\
	&= \sum_i^n Var(w_ix_i) \\\
	&= \sum_i^n[E(w_i)]^2 Var(x_i)+E[(x_i)]^2 Var(w_i)+Var(x_i)Var(w_i) \\\
	&= \sum_i^n Var(x_i)Var(w_i) \\\
	&= (n Var(w))Var(x)
	\end{align}
	所以，初始化权重矩阵 `w = np.random.randn(n) / sqrt(n)`	
	对于ReLU，初始化：`w = np.random.randn(n) * sqrt(2.0/n)`
	[Delving Deep into Rectifiers: Surpassing Human-Level Performance on ImageNet Classification](https://arxiv.org/pdf/1502.01852v1.pdf)(He et al.)

4. 稀疏初始化

5. bias初始化：通常为0

6. [Batch Normalization](http://arxiv.org/pdf/1502.03167v3.pdf): 在网络的每一层之前都做预处理，只是这种操作以另一种方式与网络集成在一起。
下图给出了Batch Normalization的具体形式：
![batch normalization](http://upload-images.jianshu.io/upload_images/2301760-ebbfe8659b9e6b27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Regularization
1. $L_2$规则化：   $\frac{1}{2}\lambda w^2$
对于大数值的权重向量进行严厉的惩罚，倾向于更加分散的权重向量。

2. $L_1$规则化：  $\lambda |w|$
具有稀疏性

3. $L_1 \& L_2$混合规则化:   $\lambda_1 |w| + \lambda_2 w^2 $
[Elastic net regularization](http://web.stanford.edu/~hastie/Papers/B67.2%20%282005%29%20301-320%20Zou%20&%20Hastie.pdf)

4. Max norm constraints（最大范式约束）:
给每个神经元中权重向量的数量设定上限，并使用投影梯度下降来确保这一约束

5. [Dropout](http://www.cs.toronto.edu/~rsalakhu/papers/srivastava14a.pdf)(随机失活)：
让神经元以超参数p的概率被激活或被设置为0
这属于网络在前向传播中有随机行为的方法

![Dropout](http://cs231n.github.io/assets/nn2/dropout.jpeg)
	上图表示Dropout方法的具体形式，下面给出对应的代码逻辑：
code:
```python
""" Vanilla Dropout: Not recommended implementation (see notes below) """

p = 0.5 # probability of keeping a unit active. higher = less dropout

def train_step(X):
  """ X contains the data """
  
  # forward pass for example 3-layer neural network
  H1 = np.maximum(0, np.dot(W1, X) + b1)
  U1 = np.random.rand(*H1.shape) < p # first dropout mask
  H1 *= U1 # drop!
  H2 = np.maximum(0, np.dot(W2, H1) + b2)
  U2 = np.random.rand(*H2.shape) < p # second dropout mask
  H2 *= U2 # drop!
  out = np.dot(W3, H2) + b3
  
  # backward pass: compute gradients... (not shown)
  # perform parameter update... (not shown)
  
def predict(X):
  # ensembled forward pass
  H1 = np.maximum(0, np.dot(W1, X) + b1) * p # NOTE: scale the activations
  H2 = np.maximum(0, np.dot(W2, H1) + b2) * p # NOTE: scale the activations
  out = np.dot(W3, H2) + b3
```
**inverted dropout**
code:
```python
""" 
Inverted Dropout: Recommended implementation example.
We drop and scale at train time and don't do anything at test time.
"""

p = 0.5 # probability of keeping a unit active. higher = less dropout

def train_step(X):
  # forward pass for example 3-layer neural network
  H1 = np.maximum(0, np.dot(W1, X) + b1)
  U1 = (np.random.rand(*H1.shape) < p) / p # first dropout mask. Notice /p!
  H1 *= U1 # drop!
  H2 = np.maximum(0, np.dot(W2, H1) + b2)
  U2 = (np.random.rand(*H2.shape) < p) / p # second dropout mask. Notice /p!
  H2 *= U2 # drop!
  out = np.dot(W3, H2) + b3
  
  # backward pass: compute gradients... (not shown)
  # perform parameter update... (not shown)
  
def predict(X):
  # ensembled forward pass
  H1 = np.maximum(0, np.dot(W1, X) + b1) # no scaling necessary
  H2 = np.maximum(0, np.dot(W2, H1) + b2)
  out = np.dot(W3, H2) + b3
```

# Loss functions
1. 分类问题：

    1) SVM : $L\_i = \sum\_{j\neq y\_i} max(0, f\_j - f\_{y\_i} + 1)$

    2)  Softmax : $L\_i = - log\frac{e^{f\_{y\_i}}}{\sum\_j e^{f\_j}}$

    3) 当样本的类别数据巨大时，需要使用[Hierarchical Softmax](http://arxiv.org/pdf/1310.4546.pdf)

    4) 属性分类: 多标签/每个样本可能没有某个属性，属性之间不互相排斥
可以为每个属性创建一个独立的二分类的分类器

2. 回归问题
$L_2$范式： $L_i = \|f - y_i\|^2_2$
$L_1$范式： $L_i = \|f - y_i\|_1 = \sum_j |f_j - (y_i)_j|$

# references
1. [CS231n:Neural Networks Part 2: Setting up the Data and the Loss](http://cs231n.github.io/neural-networks-2/)
2. [CS231n (winter 2016) : Assignment1](http://www.jianshu.com/p/004c99623104)
3. [知乎：CS231n课程笔记翻译：神经网络笔记 2](https://zhuanlan.zhihu.com/p/21560667?refer=intelligentunit)