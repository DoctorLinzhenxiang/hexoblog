---
title: lintcode[17] 子集
date: 2016-04-20 21:34:34
categories: Lintcode
tag: [lintcode, python, algorithm]
description:
---

# 题目描述
给定一个含不同整数的集合，返回其所有的子集

<!--more-->

> 如果 S = [1,2,3]，有如下的解：
	[
  		[3],
  		[1],
  		[2],
  		[1,2,3],
  		[1,3],
  		[2,3],
  		[1,2],
  		[]
	]

# 思路
这题与全排列的题目有点类似，不过子集需要把所有排列的情况都列出来
这里参考了[lintcode 中等题：subSets 子集](http://www.cnblogs.com/theskulls/p/5099745.html)的解法，感谢博主！

# 代码

```python
class Solution:
    """
    @param S: The set of numbers.
    @return: A list of lists. See example.
    """
    def subsets(self, S):
        S.sort()
        res = []
        self.mydfs(res, S, 0, 0, [])
        return res
        
        
    def mydfs(self, res, S, depth, start, valuelist):
        res.append(valuelist)
        print res
        if depth == len(S): return
        for i in range(start, len(S)):
            self.mydfs(res, S, depth+1, i+1, valuelist+[S[i]])
```
