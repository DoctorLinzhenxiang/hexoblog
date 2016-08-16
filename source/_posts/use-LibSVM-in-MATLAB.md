---
title: LibSVM在MATLAB中应用及一些问题
date: 2016-06-28 23:21:47
categories: MATLAB
tag: matlab
description: LibSVM在MATLAB中运用的一些参数释义以及在运用过程中针对precomputed kernel的问题
---

[LibSVM][1]是台大林智仁教授开发的专门用于求解各种支持向量机问题的工具包，目前已经有Python、MATLAB、Java等平台下的工具包。其实很早就有听说这个工具包，最近要实现一个对比实验需要用到所以终于有机会感受一下她的强大了。

下面简单介绍几个libsvm中常用的命令：
svmtrain
svmpredict
confusionmat

# svmtrain

```
model = libsvmtrain(training_label_vector, training_instance_matrix [, 'libsvm_options']);
```

# svmpredict

```
[predicted_label, accuracy, decision_values/prob_estimates] 
　　　　= libsvmpredict(testing_label_vector, testing_instance_matrix, model [, 'libsvm_options']);
```

# svmtrain参数设置

options:

- -s  svm_type : set type of SVM (default 0)

　　　　0 -- C-SVC

　　　　1 -- nu-SVC

　　　　2 -- one-class SVM

　　　　3 -- epsilon-SVR

　　　　4 -- nu-SVR
- -t kernel_type : set type of kernel function (default 2)

　　　　0 -- linear: $u'*v$

　　　　1 -- polynomial:  $(gamma \* u^T \* v + coef0)^{degree}$

　　　　2 -- radial basis function: $exp(-gamma*|u-v|^2)$

　　　　3 -- sigmoid: $ tanh(gamma \* u^T \* v + coef0)$

　　　　**4 -- precomputed kernel (kernel values in training_instance_matrix)**

- -d degree : set degree in kernel function (default 3)

- -g gamma : set gamma in kernel function (default 1/num_features)

- -r coef0 : set coef0 in kernel function (default 0)

- -c cost : set the parameter C of C-SVC, epsilon-SVR, and nu-SVR (default 1)

- -n nu : set the parameter nu of nu-SVC, one-class SVM, and nu-SVR (default 0.5)

- -p epsilon : set the epsilon in loss function of epsilon-SVR (default 0.1)

- -m cachesize : set cache memory size in MB (default 100)

- -e epsilon : set tolerance of termination criterion (default 0.001)

- -h shrinking: whether to use the shrinking heuristics, 0 or 1 (default 1)

- -b probability_estimates: whether to train a SVC or SVR model for probability estimates, 0 or 1 (default 0)

- -wi weight: set the parameter C of class i to weight*C, for C-SVC (default 1)

对应的中文解释[2]：

- -s svm类型：SVM设置类型（默认0)

　　　　0 — C-SVC； 
		
　　　　1 – v-SVC； 
		
　　　　2 – 一类SVM； 
		
　　　　3 — e-SVR； 
		
　　　　4 — v-SVR
- -t 核函数类型：核函数设置类型（默认2）

　　　　0 – 线性核函数：$u'*v$

　　　　1 – 多项式核函数：$(gamma \* u^T \* v + coef0)^{degree}$

　　　　2 – RBF(径向基)核函数：$exp(-gamma*|u-v|^2)$

　　　　3 – sigmoid核函数：$ tanh(gamma \* u^T \* v + coef0)$

　　　　**4 - 预先计算好的核矩阵 （核矩阵取代training_instance_matrix）**
- -d degree：核函数中的degree设置（针对多项式核函数）（默认3）

- -g r(gamma）：核函数中的gamma函数设置（针对多项式/rbf/sigmoid核函数）（默认1/k，k为总类别数)

- -r coef0：核函数中的coef0设置（针对多项式/sigmoid核函数）（（默认0)

- -c cost：设置C-SVC，e -SVR和v-SVR的参数（损失函数）（默认1）

- -n nu：设置v-SVC，一类SVM和v- SVR的参数（默认0.5）

- -p p：设置e -SVR 中损失函数p的值（默认0.1）

- -m cachesize：设置cache内存大小，以MB为单位（默认40）

- -e eps：设置允许的终止判据（默认0.001）

- -h shrinking：是否使用启发式，0或1（默认1）

- -wi weight：设置第几类的参数C为weight*C (C-SVC中的C) （默认1）

- -v n: n-fold交互检验模式，n为fold的个数，必须大于等于2

# svmtrain输出model的含义
libsvmtrain函数返回训练好的SVM分类器模型，可以用来对未知的样本进行预测。这个模型是一个结构体，包含以下成员[2]：

- -Parameters: 一个5 x 1的矩阵，从上到下依次表示：

　　　　-s SVM类型（默认0）；

　　　　-t 核函数类型（默认2）

　　　　-d 核函数中的degree设置(针对多项式核函数)(默认3)；

　　　　-g 核函数中的r(gamma）函数设置(针对多项式/rbf/sigmoid核函数) (默认类别数目的倒数)；

　　　　-r 核函数中的coef0设置(针对多项式/sigmoid核函数)((默认0)

- -nr_class: 表示数据集中有多少类别，比如二分类时这个值即为2。

- -totalSV: 表示支持向量的总数。

- -rho: 决策函数wx+b中的常数项的相反数（-b）。

- -Label: 表示数据集中类别的标签，比如二分类常见的1和-1。

- -sv_indices: 表示支持向量对应的样本索引

- -ProbA: 使用-b参数时用于概率估计的数值，否则为空。

- -ProbB: 使用-b参数时用于概率估计的数值，否则为空。

- -nSV: 表示每类样本的支持向量的数目，和Label的类别标签对应。如Label=[1; -1],nSV=[63; 67]，则标签为1的样本有63个支持向量，标签为-1的有67个。

- -sv_coef: 表示每个支持向量在决策函数中的系数。

- -SVs: 表示所有的支持向量，如果特征是n维的，支持向量一共有m个，则为m x n的稀疏矩阵。

另外，如果在训练中使用了-v参数进行交叉验证时，返回的不是一个模型，而是交叉验证的分类的正确率或者回归的均方根误差。

# svmpredice输出含义

- -predicted_label：第一个返回值，表示样本的预测类标号。

- -accuracy：第二个返回值，一个3 x 1的数组，表示分类的正确率、回归的均方根误差、回归的平方相关系数。

- -decision_values/prob_estimates：第三个返回值，一个矩阵包含决策的值或者概率估计。对于n个预测样本、k类的问题，如果指定“-b 1”参数，则n x k的矩阵，每一行表示这个样本分别属于每一个类别的概率；如果没有指定“-b 1”参数，则为n x k*(k-1)/2的矩阵，每一行表示k(k-1)/2个二分类SVM的预测结果。

# confusionmat

```
C = confusionmat(testLabel, predLabel)
```

输出的是[混淆矩阵][4]。

# 运用中遇到的一些问题

## precomputed kernel
在svmtrain的参数中，对于核函数类型的选择，参数4比较特殊，是我们自己计算好核矩阵给libsvm来求解，其中有两个注意点：
1. 核矩阵的位置--取代原来训练实例矩阵的位置

2. 核矩阵需要处理一下：

> To use precomputed kernel, you must include sample serial number as the first column of the training and testing data.
    
上面的话用代码来表示：

```matlab
K = getKernel(X, Y, options);
K = [(1:size(K, 1))', K];
model = svmtrain(trainLabel, K, '-t 4');
```

## libsvmpredict的工作原理
在应用中，有时我们只需要libsvmtrain的功能，在得到特征向量的信息之后，根据不同的需求来求解精度，而不再需要使用libsvmpredict的功能。这样我们就需要了解通过libsvmtrain求得得model来实现predict得功能。

model的信息上面已经说明了，我们只需要利用好sv得信息：model.sv_indices, model.sv_coef, model.SVs。另外需要注意的是，model.sv_coef是支持向量的系数，相当于`alpha .* y_train`,所以，我们在求解时就不需要再乘`y_train`了,具体的形式为：

$$predLabel = sign(\sum^n\_{i=1} w\_i K(x\_i, x)) + b$$

- 当输入libsvmtrain的是原始样本信息（而不是kernel matrix）
```matlab
model = libsvmtrain(y_train, x_train, '-s 0 -t 2 -c 1.2 -g 2.8');
[predictLabel, accuracy, prob_estimate] = libsvmpredict(y_test, x_test, model);

b = -model.rho;
sv_mat_sparse = model.SVs;
sv_mat_full = full(sv_mat_sparse)';
gamma = model.Parameters(4);
tmp = @(x, y) (bsxfun(@plus, sum(x.^2, 1).', sum(y.^2, 1)) - 2 * (x' * y));
RBF = exp(-gamma * tmp(sv_mat_full, x_test));
pred_label = model.sv_coef' * RBF + b;
accuracy_self = mean(sign(pred_label') == y_test);
```

- 当输入libsvmtrain的是precomputed kernel matrix
```matlab
train_kernel_mat = exp( -gamma * tmp(x_train, x_train));
train_kernel_tilde = [(1:size(train_kernel_mat, 1))', train_kernel_mat];
model = libsvmtrain(y_train, train_kernel_tilde, '-t 4');

test_kernel_mat = exp(-gamma * tmp(x_train, x_test));
test_kernel_tilde = [(1:size(test_kernel_mat, 1))', test_kernel_mat];
[predictLabel, accuracy, prob_estimate] = libsvmpredict(y_test, test_kernel_tilde, model);

b = -model.rho;
sv_mat = x_train(:, model.sv_indices');
gamma = model.Parameters(4);
RBF = exp(-gamma * tmp(sv_mat, x_test));
pred_label = model.sv_coef' * RBF + b;
accuracy_self = mean(sign(pred_label') == y_test);
```

我们会发现`accuracy_self`和`accuracy`的值是一样的。


# reference
[1] : [林智仁Libsvm主页][1]
[2] : [LIBSVM在Matlab下的使用][2]
[3] : [using precomputed kernels with libsvm][3]




[1]: http://www.csie.ntu.edu.tw/~cjlin/libsvm/
[2]: http://noalgo.info/363.html
[3]: http://stackoverflow.com/questions/7715138/using-precomputed-kernels-with-libsvm
[4]: http://baike.baidu.com/view/2781781.htm



