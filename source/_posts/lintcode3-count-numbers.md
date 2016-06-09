---
title: lintcode[3] 统计数字
date: 2016-04-02 20:39:55
categories: Lintcode
tag: [lintcode, c++, algorithm]
description: 如题~
---
# 题目描述
计算数字k在0到n中的出现的次数，k可能是0~9的一个值

> 例如n=12，k=1，在 [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]，我们发现1出现了5次 (1, 10, 11, 12)

# 思路
暴力破解，遍历每个数的每一位，但是要考虑到是从0开始的，所以单独考虑k=0的特殊情况

# 代码

```c++
class Solution {
public:
    /*
     * param k : As description.
     * param n : As description.
     * return: How many k's between 0 and n.
     */
    int digitCounts(int k, int n) {
        // write your code here
        int res = 0;
        if(k == 0){
    	    res += 1;
    	}
        while(n){
            int tmp = n;
            while(tmp){	
                int digital = tmp % 10;
                if(digital == k){
                    res += 1;
                }
                tmp /= 10;
            }
            n--;
        }
        return res;
    } 
};
```