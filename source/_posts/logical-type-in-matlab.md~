---
title: logical type in matlab
date: 2016-02-24 23:30:47
categories: MATLAB
tag: MATLAB
description: 
---

logical类型数据并不经常用到，今天在学习Ng课程的collaborative filter时，示例代码中有用到，故做个总结
<!--more-->

**logical**故名思议，逻辑数据类型，值为0或1

# 一些栗子

1. 将一个矩阵变成逻辑型矩阵，logical(A)
     ```
     >> a = logical(eye(3))

     a =

          1     0     0
          0     1     0
          0     0     1
     ```
2. 应用：

     ```
     >> b = magic(3)

     b =

          8     1     6
          3     5     7
          4     9     2
     >> c = b(a)    %获得a中1的对应位置的b的值;取出b中对应位置a为真的值
     c =

          8
          5
          2
     >> d = b(~a)   %反之
     d =

          3
          4
          1
          9
          6
          7
     ```