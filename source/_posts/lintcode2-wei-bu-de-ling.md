---
title: lintcode[2] 尾部的零
date: 2016-04-01 12:55:32
categories: Lintcode
tag: [lintcode, c++, algorithm]
description:
---

# 题目描述
设计一个算法，计算出n阶乘中尾部零的个数

> 11! = 39916800，因此应该返回 2

<!--more-->

# 思路
主要思路就是看10的组成2和5，然后含有5的数远远小于含有2的数的数量，于是我们只要统计出含有5的数的数量，同时，对于不同的数，对于5的贡献不一样，5,15,20均只贡献1个5，而25会贡献2个5，所以要计算两次，递归找下去

# 代码

```c++
class Solution {
 public:
    // param n : description of n
    // return: description of return 
    long long trailingZeros(long long n) {
        long long num = 0;
        while(n){
            num += n / 5;
            n /= 5;  
        }
        return num;
    }
};
```	