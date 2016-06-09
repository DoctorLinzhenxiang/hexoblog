---
title: matlab并行编程
date: 2016-03-10 23:23:59
categories: MATLAB
tag: matlab
description: MATLAB并行编程的方法总结
---

前几天研究了MATLAB的代码优化，今天来研究一下MATLAB的并行编程（Parallel Programming）。

# 一些概念

1. 概念：**client** & **workers**
Start up MATLAB in the regular way. This copy of MATLAB that you start with is called the "client" copy; the copies of MATLAB that will be created to assist in the computation are known as "workers".


# 一些函数
 
1. **matlabpool**

	```matlab
	>>matlabpool open local 4
	>>your_code.m
	>>matlabpool close
	```
	上面的4表示机器的核数，而不是线程数；这里的数字可以<=Num(内核)

2. **pctRunOnAll**

	```matlab
	%Clear all loaded functions on all labs
	>> pctRunOnAll clear functions

	% Change the directory on all workers to the project directory:
	>> pctRunOnAll cd /opt/projects/new_scheme

	%Add some directories to the paths of all the labs:
	>> pctRunOnAll (‘addpath /usr/share/path1’);
	```

3. **parfor**
这是对for-loop的一种并行化改进，有一些限制
	- ***Nested spmd Statements***
	The body of a parfor-loop cannot contain an spmd statement, and an spmd statement cannot contain a parfor-loop.

	- ***Break and Return Statements***
	The body of a parfor-loop cannot contain break or return statements.

	- ***Global and Persistent Variables***
	The body of a parfor-loop cannot contain global or persistent variable declarations.

	- ***Handle Classes***
	Changes made to handle classes on the workers during loop iterations are not automatically propagated to the client.

	- ***P-Code Scripts***
	You can call P-code script files from within a parfor-loop, but P-code script cannot contain a parfor-loop.

	举个栗子
	
	``` matlab
	clear all;

	matlabpool open local 2;
	ms.UseParallel = 'always';

	N = 100;
	tic
	parfor i = 1 : N
		fprintf('============================\n');
		fprintf('============================\n');
	    fprintf('Iteration Order no = %2d\n', i);
	    fprintf('============================\n');
	    fprintf('============================\n');
	end
	t_parallel = toc;
	matlabpool close
	```
	但是，这里可能由于for循环中操作比较简单，并没有比直接用for循环节约时间

4. spmd：single program multiple data
	- The ***"single program"*** aspect of spmd means that the identical code runs on multiple labs. When the spmd block is complete, your program continues running in the client.

	- The ***"multiple data"*** aspect means that even though the spmd statement runs identical code on all labs, each lab can have different, unique data for that code. So multiple data sets can be accommodated by multiple labs.


# Reference

1. [Parallel Programming Toolbox User's Guide](http://www.lsta.upmc.fr/documentation/distcomp.pdf)

2. [parallel computing with matlab](http://hpc.kfupm.edu.sa/Documentation/MATLAB%20PARALLEL.pdf)
