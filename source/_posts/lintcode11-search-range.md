---
title: lintcode[11] 二叉查找树中搜索区间
date: 2016-04-09 21:10:51
categories: Lintcode
tag: [lintcode, python, algorithm]
description: 如题~
---

# 题目描述
给定两个值 k1 和 k2（k1 < k2）和一个二叉查找树的根节点。找到树中所有值在 k1 到 k2 范围内的节点。即打印所有x (k1 <= x <= k2) 其中 x 是二叉查找树的中的节点值。返回所有升序的节点值。


> 如果有 k1 = 10 和 k2 = 22, 你的程序应该返回 [12, 20, 22].

```
     20
    /  \
   8   22
  / \
 4   12
```

# 思路
本质就是有条件的中序遍历

# 代码

```python
"""
Definition of TreeNode:
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left, self.right = None, None
"""
class Solution:
    """
    @param root: The root of the binary search tree.
    @param k1 and k2: range k1 to k2.
    @return: Return all keys that k1<=key<=k2 in ascending order.
    """     
    def searchRange(self, root, k1, k2):
        # write your code here
        res = self.inorderSearch(root, k1, k2)
        return res
        
    def inorderSearch(self, root, k1, k2):
        inorder = []
        if root == None:
            return inorder
        left = self.inorderSearch(root.left, k1, k2)
        if len(left) != 0:
#            inorder.append(left)
            inorder += left
        if root.val >= k1 and root.val <= k2:
            inorder.append(root.val)
        right = self.inorderSearch(root.right, k1, k2)
        if len(right) != 0:
#            inorder.append(right)
            inorder += right
        return inorder
```