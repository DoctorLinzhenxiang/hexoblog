---
title: lintcode[8] 旋转字符串
date: 2016-04-07 21:01:56
categories: Lintcode
tag: [lintcode, c++, algorithm]
description:
---
# 题目描述
给定一个字符串和一个偏移量，根据偏移量旋转字符串(从左向右旋转)

> 对于字符串 "abcdefg".
> offset=0 => "abcdefg"
> offset=1 => "gabcdef"
> offset=2 => "fgabcde"
> offset=3 => "efgabcd"

<!--more-->

# 思路
这题主要考虑的是字符串拼接，方法有四种
1. +=， 
2. append，
> **string的连接：**
> string &operator+=(const string &amp;s);//把字符串s连接到当前字符串的结尾 
> string &amp;append(const char *s);            //把c类型字符串s连接到当前字符串结尾
> string &amp;append(const char *s,int n);//把c类型字符串s的前n个字符连接到当前字符串结尾
> string &amp;append(const string &amp;s);    //同operator+=()
> string &amp;append(const string &amp;s,int pos,int n);//把字符串s中从pos开始的n个字符连接到当前字符串的结尾
> string &amp;append(int n,char c);        //在当前字符串结尾添加n个字符c
> string &amp;append(const_iterator first,const_iterator last);//把迭代器first和last之间的部分连接到当前字符串的结尾

3.  stringstream s1; s1&lt;&lt;s2;
4.  sprintf

# 代码

```c++
class Solution {
public:
    /**
     * @param str: a string
     * @param offset: an integer
     * @return: nothing
     */
    void rotateString(string &str,int offset){
        //wirte your code here
        int strlen = str.length();
        if(strlen == 0){
            return;
        }
        offset %= strlen;
        string tmp = "";
        while(offset){
            tmp = str[strlen - 1];
            str.erase(strlen - 1);
//          str = tmp + str;
            str = tmp.append(str);
            offset--;
        } 
    }
};

```