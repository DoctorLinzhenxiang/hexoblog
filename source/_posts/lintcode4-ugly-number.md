---
title: lintcode[4] 丑数
date: 2016-04-03 20:43:35
categories: Lintcode
tag: [lintcode, c++, algorithm]
description:
---

# 题目描述

设计一个算法，找出只含素因子2，3，5 的第 n 大的数。符合条件的数如：

> 1, 2, 3, 4, 5, 6, 8, 9, 10, 12... 
> 如果n = 9， 返回 10

<!--more-->

# 思路
这题最开始的想法就是遍历所有的数，判断该数书不是丑数，但是为超时，所以就要考虑其他方法。
这题在**[剑指offer](http://zhedahht.blog.163.com/blog/static/2541117420094245366965/)**上有，参考了他的解法
思路：只考虑丑数，而丑数的形成是由2,3,5的倍数组合而成，那就是由这三个数交替生成，产生一个丑数数组，典型的空间换时间

# 代码

```c++
class Solution {
private:
    int min(int a, int b, int c){
        int minNum = (a < b) ? a : b;
        minNum = (minNum < c) ? minNum : c;
        return minNum;
    }
public:
    /*
     * @param n an integer
     * @return the nth prime number as description.
     */
    int nthUglyNumber(int n) {
        // write your code here
        if(n < 1){
            return 0;
        }
        int *uglyNumbers = new int[n];
        uglyNumbers[0] = 1;
        int nextUglyNumIdx = 1;
		
        int *multiply2 = uglyNumbers;
        int *multiply3 = uglyNumbers;
        int *multiply5 = uglyNumbers;
		
        while(nextUglyNumIdx < n){
            int minUgly = min(*multiply2 * 2, *multiply3 * 3, *multiply5 * 5);
            uglyNumbers[nextUglyNumIdx] = minUgly;
			
            if(minUgly >= *multiply2 * 2) multiply2++;
            if(minUgly >= *multiply3 * 3) multiply3++;
            if(minUgly >= *multiply5 * 5) multiply5++;
			
            nextUglyNumIdx ++;	
        }
        int ugly = uglyNumbers[n - 1];
        delete[] uglyNumbers;
		
        return ugly; 
    }
};
```