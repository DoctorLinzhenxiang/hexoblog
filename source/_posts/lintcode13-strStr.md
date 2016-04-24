---
title: lintcode[13] 字符串查找
date: 2016-04-11 21:18:03
categories: Lintcode
tag: [lintcode, python, algorithm]
description: 
---

# 题目描述
对于一个给定的 source 字符串和一个 target 字符串，你应该在 source 字符串中找出 target 字符串出现的第一个位置(从0开始)。如果不存在，则返回 -1。

> 如果 source = "source" 和 target = "target"，返回 -1。
> 如果 source = "abcdabcdefg" 和 target = "bcd"，返回 1。

<!--more-->

# 思路
字符串匹配，有著名的KMP算法，这里咩有实现，只是用了暴力搜索
不过有很多小的细节需要注意

# 代码

```python
class Solution:
    def strStr(self, source, target):
        if source == None or target == None:
            return -1
        sourceLen = len(source)
        targetLen = len(target)
        if sourceLen == 0 and targetLen != 0:
            return -1
        if not(sourceLen and targetLen):
            return 0
        sourceIdx = 0
        while sourceIdx < sourceLen:
            targetIdx = 0
            tmpIdx = sourceIdx
            while targetIdx < targetLen and source[tmpIdx] == target[targetIdx]:
                    tmpIdx += 1
                    targetIdx += 1
            if targetIdx == targetLen:
                return sourceIdx
            else:
                sourceIdx += 1
        return -1
```