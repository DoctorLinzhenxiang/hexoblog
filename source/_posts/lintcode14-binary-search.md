---
title: lintcode[14] 二分查找
date: 2016-04-12 21:24:57
categories: Lintcode
tag: [lintcode, python, algorithm]
description:
---

# 题目描述
给定一个排序的整数数组（升序）和一个要查找的整数target，用O(logn)的时间查找到target第一次出现的下标（从0开始），如果target不存在于数组中，返回-1。

> 在数组 [1, 2, 3, 3, 4, 5, 10] 中二分查找3，返回2。

<!--more-->

# 思路
search_idx = (tail + head) / 2 这里不是减法
然后注意是要找到第一个target

# 代码

```python
class Solution:
    # @param nums: The integer array
    # @param target: Target number to find
    # @return the first position of target in nums, position start from 0 
    def binarySearch(self, nums, target):
        # write your code here
        len_nums = len(nums)
        search_idx = 0
        head = 0
        tail = len_nums - 1
        while head != tail:
            search_idx = (tail + head) / 2
            if nums[search_idx] == target:
                while nums[search_idx] == target:
                    if search_idx > head:
                        search_idx -= 1
                    else:
                        return search_idx
                return search_idx + 1
            elif nums[search_idx] > target:
                tail = search_idx - 1
            else:
                head = search_idx + 1
        if nums[head] == target:
            return head
        else:
            return -1
```

