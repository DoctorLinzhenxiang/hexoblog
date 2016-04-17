---
title: matlab代码优化
date: 2016-04-08 23:27:02
categories: MATLAB
tag: [MATLAB, 代码优化]
description:
---

最近做实验发现跑的时间实在太漫长了，其实主要原因是迭代次数比较大的原因，需要进一步读些paper，找到好的解决方法，但是也感觉对于MATLAB的代码优化还是需要好好掌握一下的，因为之前在做图像处理实验时就发现for循环和向量化之后的性能差异简直一天一地，所以今天google一下，感觉发现新大陆一样[FreeMind][2]的[Recipes for Faster Matlab Code](http://freemind.pluskid.org/programming/recipes-for-faster-matlab-code/)的对k-medoids的一段代码的解释，收获颇丰，要好好的掌握下。原文就不贴了（主要是他搞的太精致了，实在不忍心粘成txt格式的......），自己来总结一下吧。

-------------------------------------------------------------------------
## 一个栗子
``` matlab
function [label, energy, index] = kmedoids(X,k)
% X: d x n data matrix
% k: number of cluster
% Written by Mo Chen (sth4nth@gamil.com)
v = dot(X,X,1);
D = bsxfun(@plus,v,v')-2*(X'*X);
n = size(X,2);
[~, label] = min(D(randsample(n,k),:));
last = 0;
while any(label ~= last)
    [~, index] = min(D*sparse(1:n,label,1,n,k,n));
    last = label;
    [val, label] = min(D(index,:),[],1);
end
energy = sum(val);
```

1. dot(A, B, dim)函数，实际完成的是对两个矩阵对应元素相乘，然后在dim维上求和。主要可以在求两个点之间的距离时用得上。

2. bsxfun(FUNC,A,B)函数，实际上就是要对两个矩阵进行+，-，*，/等等的操作，只是有一个较强大的功能，当输入的A或B中某个dim是1，则会自动在这一维度上进行复制，形成两个大小一样的矩阵，再进行操作，这个功能类似于repmat，不过在freemind的blog中已经实验证明bsxfun 的性能略优于repmat。感觉还是很好用的！
还有个类似的meshgrid，help里面举得栗子

	``` MATLAB
	[X,Y] = meshgrid(-2:.2:2, -4:.4:4);
	```
得到的X和Y分别对-2:.2:2和-4:.4:4在行和列上进行了扩展。

3. randsample(n, k)函数，故名思议，从1：n个数中随机采用k个数（还有几个功能，遇到再整理吧）

4. S = sparse(i,j,s,m,n,nzmax)函数，产生一个稀疏矩阵，使用i,j,s来产生一个m X n的稀疏矩阵，其中S(i(k),j(k)) = s(k),共有nzmax个非零值。

- 建议使用A = logical(sparse(m,n))，不建议使用 A = sparse(false(m,n))，两者结果一样，但是后者生成m×n的临时矩阵，浪费空间，且当m、n很大时，后者不一定能申请成功

- 由于matlab按照“先行后列”的方式读取数据（即先把第一列所有行读取完以后再读取第二列的各行），因此定义稀疏矩阵时，最好“行数>列数”， 这样有利于寻址和空间的节省

5. 最后freemind那篇blog还提到了[profile](http://cn.mathworks.com/help/matlab/ref/profile.html)，主要是用来监测代码性能的工具，需要好好研究研究

------------------------------------------------------------------------
## 综述（总结）

### 编码技巧

1. **程序语法分析，消除warning项**

- 没有分号结束符，输出会耗时

- 未初始化变量

2. **内存预分配**(循环内大数组预先定义)

3. **减少临时变量**的使用

4. 在迫不得已要使用较多循环而每一步都很耗时的时候，可以**将每一步的结果保存成mat文件**，然后**在循环的开头用if语句与exist函数进行判断**， 如果该文件存在的话，就直接进入下一次循环。这样做的好处是不会因为断电、电脑死机或误操作等原因而重头从第一个循环再次运行。（这个没试过）

5. 有多层循环时，将**循环次数多的放到内层**，且有**LOAD函数**时尽量**往外层放**

13. 在for循环中，清零操作用赋值语句 A = B，其中B是在for循环外的一个同A大小一样的全0阵，不要使用A(:) = 0；但这样会大大影响后面的逐点运算速度。这个问题要请教高手，那就是***“个别语句的改动会引起其他语句的执行速度”***。

17. 在必须使用循环时，可以考虑**转换为C-MEX**
当必须使用耗时的循环时，可以考虑将循环体中的语句转换为C-MEX。C-MEX是将M文件通过MATLAB的编译器转换为可执行文件，是按照 MEX 技术要求的格式编写相应的程序，通过编译连接，生成扩展名为.dll的动态链接库文件，可以在MATLAB环境下直接执行。这样，循环体中的语句在执行时不必每次都解释(interpret)。一般来说，C-MEX 文件的执行速度是相同功能的M文件执行速率的20~40倍。编写C-MEX不同于M文件，需要了解MATLAB C-MEX规范。幸运的是MATLAB提供了将M文件转换为C-MEX的工具。

23. 在可能的情况下，**使用numeric array或者struct array**，它们的效率大幅度高于cell array（几十倍甚至更多）。对于struct，尽可能使用普通的域（字段，field）访问方式，在非效率关键，执行次数较少，而灵活性要求较高的代码中，可以考虑使用动态名称的域访问。

### 函数：内建函数，.m函数，匿名函数，句柄函数
9. **尽量使用MATLAB的内部函数**

10. **利用内置函数自动记录运行结果**
matlab生成的结果变量只贮存在内存空间中，一旦matlab关闭则会丢失，很多时候需要手工将这些结果存储到我们需要的文件中。事实上，matlab提供 了很多函数可以自动记录结果，如fprintf可以将数据写入txt文件，xlswrite可以将数据写入excel文件等等。这样自动化的命令可以缩减大量时间， 从而提高效率。

11. **用函数代替脚本文件**
调用多次的计算代码写成函数形式，而不是写在脚本程序中，因为Matlab中，函数是被翻译成微码的，执行效率更高。

14. **二维矩阵转置**操作可以用以下三种方法进行，三者的效率基本一样（时空），如果遇到三维以上的矩阵要转置，用permute命令较为方便：
a) A = A’;
b) A = permute(A,[2,1]);
c) A = shiftdim(A,1);

15. “使用**eval方式**动态存储多个一维数组”比“使用二维数组动态存储多个一维数组”要快，这个没有看太懂，help eval，看到如下代码：
	```
expression = input('Enter the name of a matrix: ','s');
if (exist(expression,'var'))
	plot(eval(expression))
end
	```
	大概能理解就是判断用户输入的变量是否存在于当前变量空间中。
	
16. **corrcoef函数**，求整个矩阵所有列的相关系数，因为我只需要求出某一列跟其他各列的相关 系数，所以参照corrcoef函数自己写了一个

18.  **内存管理函数和命令**
（1）Clear variablename：从内存中删除名称为variablename的变量。
（2）Clear all：从内存中删除所有的变量。
（3）Save：将指令的变量存入磁盘。
（4）Load：将save命令存入的变量载入内存。
（5）Quit：退出MATLAB，并释放所有分配的内存。
（6）Pack：把内存中的变量存入磁盘，再用内存中的连续空间载回这些变量。考虑到执行效率问题，不能在循环中使用。

19. 最常用的使用**vectorizing技术**的函数有：All、diff、ipermute、permute、reshape、ueeze、y、find、logical、prod、shiftdim、sub2ind、cumsum、ind2sub、ndgrid、repmat、sort、sum 等。

20. 用一个循环建立一个向量，其元素依赖于前一个元素使用的工具：**FILTER, CUMSUM, CUMPROD**
 - filter 
 优化前： 

	```
A = 1; 
L = 1000; 
for i = 1:L 
  A(i+1) = 2*A(i)+1; 
end 
	```
	优化后： 
	```
L = 1000; 
A = filter([1],[1 -2],ones(1,L+1)); 
	```

- cumsum或cumprod函数：如果向量构造只使用加法或乘法
优化前： 

	``` MATLAB 
n=10000; 
V_B=100*ones(1,n); 
V_B2=100*ones(1,n); 
ScaleFactor=rand(1,n-1); 
for i = 2:n 
	   V_B(i) = V_B(i-1)*(1+ScaleFactor(i-1)); 
end 
for i=2:n 
	   V_B2(i) = V_B2(i-1)+3; 
end 
	```

	优化后： 
	``` matlab
n=10000; 
V_A=100*ones(1,n); 
V_A2 = 100*ones(1,n); 
ScaleFactor=rand(1,n-1); 
V_A=cumprod([100 1+ScaleFactor]); 
V_A2=cumsum([100 3*ones(1,n-1)]); 
	```
21. MATLAB的函数调用过程（非built-in function）有显著开销，因此，在效率要求较高的代码中，应该尽可能采用**扁平的调用结构**，也就是在保持代码清晰和可维护的情况下，**尽量直接写表达式和利用built-in function**，避免不必要的自定义函数调用过程。在次数很多的循环体内（包括在cellfun, arrayfun等实际上蕴含循环的函数）形成长调用链，会带来很大的开销。
> 注：感觉这里好像和上面第三条矛盾了，不过我想应该尽量使用内建函数来实现需求吧，这样既可以减少使用自己建的函数，也可以提升脚本代码的效率。

22. 在调用函数时，**首选built-in function，然后是普通的m-file函数，然后才是function handle或者anonymous function**。在使用function handle或者anonymous function作为参数传递时，如果该函数被调用多次，最好先用一个变量接住，再传入该变量。这样，可以有效避免重复的解析过程。（其实这条在[matlab代码优化][1]中有举了一个栗子来验证得到的结果）

------------------------------------------
##Reference：

- [FreeMind][2]

- [ Matlab高性能编程——代码优化和并行计算](http://blog.csdn.net/linj_m/article/details/9730717)

- [Matlab代码优化方法几则](http://ibillxia.github.io/blog/2012/04/25/matlab-code-optimization/)

- [matlab程序性能优化与混合编程技术介绍](http://www.cnblogs.com/emanlee/archive/2012/02/27/2370504.html)

- [优化Matlab语言，提高运行速度](http://www.zhizhihu.com/html/y2010/1670.html)，这篇blog里面讲了很多很细致的点，需要好好消化

- [matlab代码优化][1]，这篇被转了多次好像找不到源主了，有点杂


-----------------------------------------------------
[1]: http://m.blog.csdn.net/article/details?id=29408981
[2]: http://freemind.pluskid.org