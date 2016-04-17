---
title: lintcode[15] 全排列
date: 2016-04-17 18:39:20
categories: Lintcode
tag: [lintcode, algorithm]
description:
---

# 题目描述
For nums = [1,2,3], the permutations are:
> [[1,2,3],
   [1,3,2],
   [2,1,3],
   [2,3,1],
   [3,1,2],
   [3,2,1]]

<!--more-->

# 思路
这题比较简单，不过lintcode平台这题不能用len()函数,所以，需要用enumerate来实现(这里参考了这位[博主](http://blog.csdn.net/u013291394/article/details/50476076#comments)的方法)

# 代码
```python
class Solution:
    """
    @param nums: A list of Integers.
    @return: A list of permutations.
    """
    def permute(self, nums):
        # write your code here
        #if len(num) <= 1: return [num]
        result = []
        if nums == None:
            return []
        else:
            self.get_permute([], nums, result)
            return result

    def get_permute(self, current, num, result):
        if not num:
            result.append(current + [])
            return
        for i, v in enumerate(num):
            current.append(num[i])
            self.get_permute(current, num[:i] + num[i + 1:], result)
            current.pop()
```

    

