---
title: lintcode[1] a + b
date: 2016-04-01 12:50:21
categories: Lintcode
tag: [lintcode, c++, algorithm]
description:
---

# 题目描述
给出两个整数a和b, 求他们的和, 但不能使用 + 等数学运算符。
<!--more-->

# 思路
这题其实考察了加法的本质：两数相加和进位操作 用异或表示两数相加，用与和左移来表示进位

# 代码

```c++
class Solution {
public:
    /*
     * @param a: The first integer
     * @param b: The second integer
     * @return: The sum of a and b
     */
    int aplusb(int a, int b) {
        // write your code here, try to do it without arithmetic operators.
        if(b == 0){
            return a;
        }
        int sum = a ^ b;
        int carry = (a & b) << 1;
        return aplusb(sum, carry);
    }
};
```
