---
title: lintcode[6] 合并排序数组
date: 2016-04-05 20:52:49
categories: Lintcode
tag: [lintcode, c++, algorithm]
description: 如题~
---

# 题目描述
合并两个排序的整数数组A和B变成一个新的数组。

> 给出A=[1,2,3,4]，B=[2,4,5,6]，返回 [1,2,2,3,4,4,5,6]


# 思路
这题可以考虑不另外开辟空间，直接先比较两个数组的大小，将小的复制到大的里面，然后返回大的就好，不过这里，vector用的不熟。

# 代码

```c++
class Solution {
public:
    /**
     * @param A and B: sorted integer array A and B.
     * @return: A new sorted integer array
     */
    vector<int> mergeSortedArray(vector<int> &A, vector<int> &B) {
        // write your code here
        int A_len = A.size();
        int B_len = B.size();
        
        vector<int> C;
        int i = 0, j = 0;
        while(i < A_len && j < B_len){
            if(A[i] <= B[j]){
                C.push_back(A[i++]);
            }else{
                C.push_back(B[j++]);
            }
        }
        while(i < A_len){
            C.push_back(A[i++]);
        }
        while(j < B_len){
            C.push_back(B[j++]);
        }
        return C;
    }
};
```