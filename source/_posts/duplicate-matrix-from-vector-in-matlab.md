---
title: matlab中将向量复制成矩阵
date: 2016-02-24 23:29:23
categories: MATLAB
tag: matlab
description: 如题~
---

# 向量复制成矩阵常用方法是repmat

```
>> a = [2 3 4];
>> b = repmat(a, 3, 1)

b =

     2     3     4
     2     3     4
     2     3     4
```
# 今天还发现了一种复制的方法

```
>> c = a(ones(3, 1), :)

c =

     2     3     4
     2     3     4
     2     3     4
>> d = [2 3 4]';
>> e = d(:, ones(3, 1))

e =

     2     2     2
     3     3     3
     4     4     4
```
