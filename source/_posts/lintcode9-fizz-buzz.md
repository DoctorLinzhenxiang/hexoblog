---
title: lintcode[9] fizz buzz问题
date: 2016-04-08 21:06:03
categories: Lintcode
tag: [lintcode, c++, algorithm]
description: 如题~
---

# 题目描述
给你一个整数n. 从 1 到 n 按照下面的规则打印每个数：

- 如果这个数被3整除，打印fizz.
- 如果这个数被5整除，打印buzz.
- 如果这个数能同时被3和5整除，打印fizz buzz.


> 比如 n = 15, 返回一个字符串数组：

```
[
  "1", "2", "fizz",
  "4", "buzz", "fizz",
  "7", "8", "fizz",
  "buzz", "11", "fizz",
  "13", "14", "fizz buzz"
]
```

# 思路
主要判断这个数是不是能被3或5整除

# 代码

```c++
class Solution {
public:
    /**
     * param n: As description.
     * return: A list of strings.
     */
    vector<string> fizzBuzz(int n) {
        vector<string> results;
        for (int i = 1; i <= n; i++) {
            if (i % 15 == 0) {
                results.push_back("fizz buzz");
            } else if (i % 5 == 0) {
                results.push_back("buzz");
            } else if (i % 3 == 0) {
                results.push_back("fizz");
            } else {
                results.push_back(to_string(i));
            }
        }
        return results;
    }
};
```
