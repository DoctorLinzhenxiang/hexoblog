---
title: lintcode[16] 带重复元素的排列
date: 2016-04-18 21:28:46
categories: Lintcode
tag: [lintcode, python, algorithm]
description:
---

# 题目描述
给出一个具有重复数字的列表，找出列表所有不同的排列。

> 给出列表 [1,2,2]，不同的排列有：
	[
  	[1,2,2],
  	[2,1,2],
  	[2,2,1]
	]

<!--more-->

# 思路
这题与上一题的思路差不多，先给list排序，然后在进行排列

# 代码

```python
class Solution:
    """
    @param nums: A list of integers.
    @return: A list of unique permutations.
    """
    def permuteUnique(self, nums):
        # write your code here
        res = []
        nums.sort()
        self.getPermute([], nums, res)
        return res
        
    def getPermute(self, current, num, res):
        if not num:
            res.append(current + [])
            #print res
            return
        for i, v in enumerate(num):
            if i > 0 and num[i] == num[i - 1]:
                continue
            current.append(num[i])
            self.getPermute(current, num[:i] + num[i+1:], res)
            current.pop()

```