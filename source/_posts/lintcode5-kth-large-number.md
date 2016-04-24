---
title: lintcode[5] 第k大元素
date: 2016-04-04 20:47:46
categories: Lintcode
tag: [lintcode, c++, algorithm]
description:
---
# 题目描述
在数组中找到第k大的元素

> 给出数组 [9,3,2,4,8]，第三大的元素是 4
> 给出数组 [1,2,3,4,5]，第一大的元素是 5，第二大的元素是 4，第三大的元素是3，以此类推

<!--more-->

# 思路
这题直接用sort函数排个序就解决了

# 代码

```c++
class Solution {
public:
    /*
     * param k : description of k
     * param nums : description of array and index 0 ~ n-1
     * return: description of return
     */
    int kthLargestElement(int k, vector<int> nums) {
        // write your code here
        sort(nums.begin(), nums.end(),greater<int>());
        return nums[k-1];
    }
};
```