---
title: lintcode[12] min stack
date: 2016-04-10 21:15:33
categories: Lintcode
tag: [lintcode, python, algorithm]
description:
---

# 题目描述
实现一个带有取最小值min方法的栈，min方法将返回当前栈中的最小值。

你实现的栈将支持push，pop 和 min 操作，所有操作要求都在O(1)时间内完成。

> 如下操作：push(1)，pop()，push(2)，push(3)，min()， push(1)，min() 返回1，2，1

<!--more-->

# 思路
搞两个list，一个放原序数，另一个存前i个数的最小值
这边一开始多想了，在minstack中并不需要把所有的数都放进去，要真正理解存前i个数的最小值得意义

对了今天发现lintcode一个非常恶心的bug，居然不能有中文注释！！！！！搞了好久wtf

# 代码

```python
class MinStack(object):

    def __init__(self):
        # do some intialize if necessary
        self.stack = []
        # the ith number is the min of the numbers哈哈
        self.minstack = []

    def push(self, number):
        # write yout code here
        self.stack.append(number)
        if len(self.minstack) == 0 or self.minstack[-1] >= number:
            self.minstack.append(number)

    def pop(self):
        # pop and return the top item in stack
        if self.stack[-1] == self.minstack[-1]:
            self.minstack.pop()
        return self.stack.pop()
        
    def min(self):
        # return the minimum number in stack
        return self.minstack[-1]
```
