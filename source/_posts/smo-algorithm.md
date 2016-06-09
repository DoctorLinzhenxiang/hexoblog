---
title: SMO算法实现
date: 2016-05-01 23:31:56
categories: [Machine Learning]
tag: [machine learning, python]
description: 最近机器学习课上到了支持向量机，这两天把SMO算法实现了一下。
---

# 一些理解
svm的理论部分在很多资料里都有详细的推导过程，这里就不总结了，对于SMO算法，《统计学习方法》中是这样描述的：
> smo算法是一种启发式算法，其基本思路是：如果所有变量的解都满足此最优化问题的KKT条件，那么这个最优化问题的解就得到了。**因为KKT条件是该最优化问题的充分必要条件**

所以，我们的目标就是找到一组解使得他们都满足最优化问题的KKT条件。那么smo算法的手段就是：
> 否则，选择两个变量，固定其他变量，针对这两个变量构建一个二次规划问题。这个二次规划问题关于这两个变量的解应该更接近于原始二次规划问题，。。。

对于两个变量的选择，要求其中至少一个变量是违反KKT条件的。第一个变量$\alpha_1$选取违反KKT条件最严重的样本点，在实现过程中，感觉并不好判断违反的严重程度，所以选择全部遍历；第二个变量$\alpha_2$选取希望使其能够有足够大的变化。

# Python实现
代码有参考博文[1]中的实现，主要改进：
1. 对于第二个变量的选择，并没有遍历所有的样本点，而是选择能够使得$\alpha_2$有足够大的变化的点，实验表明，这样能够提高迭代效率；
2. 给出了高斯核函数的实现
3. 重构了一下代码~~ 当然也还有改进的地方，欢迎下方评论哦~
4. [源码及样本数据下载](https://github.com/HeimingX/MLAlgorithms/tree/master/smv_smo)如果有帮助记得点赞哦~~哈哈

```python
# -*- coding: utf-8 -*-
'''
Created on 2016年5月1日

@author: heiming
'''
import numpy as np
from scipy import io
import matplotlib.pyplot as plt

class Params:
    def __init__(self):
        ##alpha:lagrange multiplyer
        self.a = []
        ##b bias
        self.b = 0.0
        ## punish parameter
        self.c = 10
        ## decision error
        self.epsilon = 0.001
        ## stop tolerance
        self.stopTol = 0.0001
        ## smo difference of predict accuracy
        self.e = []
        ## maximum iteration
        self.maxIter = 100
        ## kernel type
        self.kernelType = 'gaussian'
        ## gaussian parameter
        self.sigma = 0.5
    
    def setC(self, c):
        self.c = c
    
    def setEpsilon(self, epsilon):
        self.epsilon = epsilon
        
    def setKernel(self, kernelType, *args):
        self.kernelType = kernelType
        if self.kernelType == 'gaussian':
            self.sigma = args[0]
    
    def setMaxIter(self, Maxiter):
        self.maxIter = Maxiter

def loadData(filePath, fileType):
    """
    load样本
    """
    if fileType == 'txt':
        data = np.loadtxt(filePath)
        trainData = data[:, :-1]
        trainLabel = data[:, -1]
        testData = []
        testLabel = []
    if fileType == 'mat':
        data = io.loadmat(filePath)
        trainData = data['xapp']
        trainLabel = np.reshape(data['yapp'], newshape = (1, data['yapp'].size))[0]
        testData = data['xtest']
        testLabel = np.reshape(data['ytest'], newshape = (1, data['ytest'].size))[0]
    
    return (trainData, trainLabel, testData, testLabel)

def kernel(data_j, data_i, params):
    """
    计算样本j和样本i的内积
    """
    if params.kernelType == 'linear':
        k = np.dot(data_j, data_i)
    if params.kernelType == 'gaussian':
        diff = data_j - data_i
        diff = np.dot(diff, diff)
        k = np.exp(- diff / (2.0 * np.square(params.sigma)))
    
    return k

def gOfx(data, label, data_i, params):
    """
    计算g(x)=sum(ai * yi * k(xi, x)) + b
    """
    if params.kernelType == 'linear':
        k = np.dot(data, data_i) 
    if params.kernelType == 'gaussian':
        diff = np.subtract(data, data_i)
        diff = np.sum(np.multiply(diff, diff), axis = 1)
        k = np.exp(- diff / (2.0 * np.square(params.sigma)))
    g = np.dot(np.multiply(params.a ,label), k) + params.b
    return g

def init(data, label, params):
    """
    初始化E, a
    """
    params.a = np.zeros((1, len(label)))[0]
    for idx in range(len(params.a)):
        diff = gOfx(data, label, data[idx], params) - label[idx]
        params.e.append(diff)

def updateE(data, label, params):
    """
    更新E
    """
    for idx in range(len(params.a)):
        diff = gOfx(data, label, data[idx], params) - label[idx]
        params.e[idx] = diff

def getMaxE(idx, params):
    """
    获取除去第idx个样本后最大的E
    """
    tmp = params.e[1 : idx] + [0] + params.e[idx+1 :]
    return tmp.index(max(tmp))

def getMinE(idx, params):
    """
    获取除去第idx个样本后最小的E
    """
    tmp = params.e[1:idx] + [params.c * 10] + params.e[idx+1 :]
    return tmp.index(min(tmp))

def judgeConvergence(data, label, params):
    """
    判断是否都已满足KKT条件
    """
    alphay = 0.0    
    for idx in range(len(label)):
        yg = label[idx] * gOfx(data, label, data[idx], params)
        if (params.a[idx] == 0) and (yg < 1 - params.epsilon):
            return False
        elif (params.a[idx] > 0) and (params.a[idx] < params.c) and \
            (yg > 1 + params.epsilon) or (yg < 1 - params.epsilon):
            return False
        elif (params.a[idx] == params.c) and (yg > 1 + params.epsilon):
            return False
        if (params.a[idx] < 0) or (params.a[idx] > params.c):
            return False
        alphay += params.a[idx] * label[idx]
    if abs(alphay) > params.epsilon:
        return False
    return True

def train(data, label, params):
    """
    svm train, use smo solve the quadratic convex problem
    """
    iters = 0
    init(data, label, params)
    updated = True
    while iters < params.maxIter and updated:
        updated = False
        iters += 1
        ##这里直接遍历所有的样本，只要违反了KKT条件就进入迭代
        for i in range(len(params.a)):
            ai = params.a[i]
            ei = params.e[i]
            
            if (label[i] * ei < params.stopTol and ai < params.c) or \
                (label[i] * ei > params.stopTol and ai > 0):
                if ei > 0:
                    j = getMinE(i, params)
                else:
                    j = getMaxE(i, params)
                
                eta = kernel(data[i], data[i], params) + \
                    kernel(data[j], data[j], params) - \
                    2 * kernel(data[i], data[j], params)
                if eta <= 0:
                    continue
                new_aj = params.a[j] + label[j] * \
                    (params.e[i] - params.e[j]) / eta
                L = 0.0
                H = 0.0
                if label[i] == label[j]:
                    L = max(0, params.a[j] + params.a[i] - params.c)
                    H = min(params.c, params.a[j] + params.a[i])
                else:
                    L = max(0, params.a[j] - params.a[i])
                    H = min(params.c, params.c + params.a[j] - params.a[i])
                if new_aj > H:
                    new_aj = H
                if new_aj < L:
                    new_aj = L
                
                ##判断alpha j是否有较大的变化
                if abs(params.a[j] - new_aj) < 0.001:
                    print "j= %d, is not moving enough" % j
                    continue
                new_ai = params.a[i] + label[i] * label[j] * (params.a[j] - new_aj)
                new_b1 = params.b - params.e[i] - label[i] * kernel(data[i], data[i], params) * (new_ai - params.a[i]) \
                    - label[j] * kernel(data[j], data[i], params) * (new_aj - params.a[j])
                new_b2 = params.b - params.e[j] - label[i] * kernel(data[i], data[j], params) * (new_ai - params.a[i]) \
                    - label[j] * kernel(data[j], data[j], params) * (new_aj - params.a[j])
                if new_ai > 0 and new_ai < params.c:
                    new_b = new_b1
                elif new_aj > 0 and new_aj < params.c:
                    new_b = new_b2
                else:
                    new_b = (new_ai + new_aj)/2.0
                    
                params.a[i] = new_ai
                params.a[j] = new_aj
                params.b = new_b
                updateE(data, label, params)
                updated = True
                print "iterate: %d, change pair: i: %d, j: %d" % (iters, i, j)
        res = judgeConvergence(data, label, params)
        if res:
            break

def predictAccuracy(trainData, trainLabel, testData, testLabel, params):
    """
    计算测试集的精度
    """
    pred = np.zeros((1, len(testLabel)))
    for idx in range(len(testLabel)):
        pred[0, idx] = gOfx(trainData, trainLabel, testData[idx], params)
    accuracy = np.mean(np.sign(pred) == testLabel)
    return accuracy

def getSupportVector(params):
    """
    获取支持向量的index
    """
    supportVector = []
    for idx in range(len(params.a)):
        if params.a[idx] > 0 and params.a[idx] < params.c:
            supportVector.append(idx)
    return supportVector
            
def draw(trainData, trainLabel, params):
    """
    特征数是两维时，可以画图来展示
    """
    plt.xlabel(u'x1')
    plt.xlim(0, 100)
    plt.ylabel(u'x2')
    plt.ylim(0, 100)
    plt.title('svm - %s, tolerance %f, c %f' % ('trainData', params.stopTol, params.c))
    for idx in range(len(trainLabel)):
        data = trainData[idx]
        if int(trainLabel[idx]) > 0:
            plt.plot(data[0], data[1], 'or')
        else:
            plt.plot(data[0], data[1], 'xg')
    
    w1 = 0.0
    w2 = 0.0
    for i in range(len(trainLabel)):
        w1 += params.a[i] * trainLabel[i] * trainData[i][0]
        w2 += params.a[i] * trainLabel[i] * trainData[i][1]
    w = - w1 / w2
    
    b = - params.b / w2
    r = 1 / w2
    lp_x1 = [10, 90]
    lp_x2 = []
    lp_x2up = []
    lp_x2down = []
    for x1 in lp_x1:
        lp_x2.append(w * x1 + b)
        lp_x2up.append(w * x1 + b + r)
        lp_x2down.append(w * x1 + b - r)
    plt.plot(lp_x1, lp_x2, 'b')
    plt.plot(lp_x1, lp_x2up, 'b--')
    plt.plot(lp_x1, lp_x2down, 'b--')
    plt.show()
    
    
def main():
####------------------------------------------
#     filePath = '../smo/train.txt'
#     fileType = 'txt'
#     trainData, trainLabel, testData, testLabel = loadData(filePath, fileType)
#     params = Params()
#     params.setKernel('linear')
    
####------------------------------------------
    filePath = '../smo/ionosphere_1.mat'
    fileType = 'mat'
    trainData, trainLabel, testData, testLabel = loadData(filePath, fileType)
    params = Params()
#     params.setKernel("gaussian", 0.5) #acc=0.902857  100步截断
    params.setKernel("gaussian", 1) # acc = 0.942857  18步收敛
    params.setMaxIter(500) ##在230步左右收敛，精度为0.88,sv 太多
    params.setEpsilon(0.01)

####------------------------------------------

    train(trainData, trainLabel, params)
    print 'a= ', params.a
    print 'b= ', params.b
         
    supportVector = getSupportVector(params)
    print 'support vector = ', supportVector
    
#### 两维特征可以在样本空间中画出图来展示划分效果
#     draw(trainData, trainLabel, params)

#### 有测试集可以计算出预测精度
    print('test set accuracy: %f' % predictAccuracy(trainData, trainLabel, testData, testLabel, params))

if __name__ == '__main__':
    main()
```

# 实验结果分析
- 线性核函数的分类效果图
![linear kernel](/img/blog/smo/smo.png)

这里可能跟博文[1]中的结果不太一样了，主要是由于改变了对于第二变量的选择

- 高斯核函数
> **dataset: ionosphere(trainingSet : testingSet = 1:1)**
sigma = 5 accuracy = 0.46 iteration <=100
sigma = 1 accuracy = 0.942857 iteration = 18
sigma = 0.5 accuracy = 0.902857 iteration = 100+
sigma = 0.05 accuracy = 0.645714 iteration <=100

    可以看出sigma的选择对于精度的影响还是很大的。这里只列出了sigma的影响，还有其他的参数对于精度和训练的迭代步数都有影响。


# 一些想法
1. 刚开始看的时候有这样一个疑问：
> 如果有多组解都满足kkt条件，那么smo算法不一定会找到使得目标函数取最小值的解

    然而有：KKT条件是最优化问题的充分必要条件，不禁感叹这样的理论是多么的强大！！！于是可以放心大胆的用smo了

2. 对于核函数的选择，高斯核是很常用的，但是对于sigma的选取却很有讲究，在上面的实验结果也能够看出，不同的sigma的选择，对预测精度和迭代步数都有影响，于是搜索了一下，高斯核参数sigma对svm的影响，发现有不少关于这方面的讨论，不过还有一些暂时还消化不了，先总结下来。
    
    > 这里面大家需要注意的就是gamma的物理意义，大家提到很多的RBF的幅宽，它会影响每个支持向量对应的高斯的作用范围，从而影响泛化性能。我的理解：如果设的**太大**，则会造成**只会作用于支持向量样本附近，对于未知样本分类效果很差**，存在训练准确率可以很高，而测试准确率不高的可能，就是通常说的过训练；而如果设的**过小**，则会**造成平滑效应太大，无法在训练集上得到特别高的准确率，也会影响测试集的准确率**。

    此外大家注意RBF公式里面的$\sigma$和$\gamma$的关系如下：

    The rbf kernel is typically defined as

    $$k(x,z) = e^{\frac{-d(x,z)^2}{2*\sigma^2}}$$ 
    
    which can be re-defined in terms of gamma as 

    $$k(x,z) = e^{-\gamma *d(x,z)^2}$$
    
    where $\gamma = \frac{1}{2*\sigma^2}$.

    > $\gamma$越大，支持向量越少，$\gamma$值越小，支持向量越多。    
    > RBF核应该可以得到与线性核相近的效果（按照理论，RBF核可以模拟线性核），可能好于线性核，也可能差于，但是，不应该相差太多。

    当然，很多问题中，比如维度过高，或者样本海量的情况下，大家更倾向于用线性核，因为效果相当，但是在速度和模型大小方面，线性核会有更好的表现。

3. smo名字的由来，未知~求解答~~

# reference
1. [SMO序列最小最优化算法](http://liuhongjiang.github.io/tech/blog/2012/12/28/svm-smo/) 这篇博文写的很赞，里面对KKT条件的推导规约很好
2. [SVM的两个参数 C 和 gamma](http://blog.csdn.net/lujiandong1/article/details/46386201) 
3. [使用svm的一个常见错误](http://blog.sina.com.cn/s/blog_6ae183910101cxbv.html) 及博主的[微博](http://weibo.com/facerecog?source=blog&is_all=1&noscale_head=1&is_search=1&key_word=rbf#_rnd1462087315345) 啊，里面还是有一些暂时不能看懂的东西，核方法很强大，需要进一步学习！
4. [Python Numpy Tutorial](http://cs231n.github.io/python-numpy-tutorial/) 这个是stanford的一个教程，很赞！





