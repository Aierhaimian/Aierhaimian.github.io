---
layout:     post
title:      7. Reverse Integer
subtitle:   LeetCode刷题笔记
date:       2018-03-19
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-7. Reverse Integer 题目

# 题目 #

Given a 32-bit signed integer, reverse digits of an integer.

**Example 1:**

	Input: 123
	Output:  321

**Example 2:**

	Input: -123
	Output: -321

**Example 3:**

	Input: 120
	Output: 21

**Note:**

Assume we are dealing with an environment which could only hold integers within the 32-bit signed integer range. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.

# 解题思路 #

(1). 算法描述：

此题思路较为简单，关键之处在于如何处理转换后的数字的溢出问题。

(2). 算法实现：

----------

	class Solution {
	public:
	    int reverse(int x) {
	      if (x == 0)
	      		return x;
	
	      int sign = 1; //sign = 0 为正数 sign = -1 为负数
	      if (x < 0) {
	      		sign = -1;
	         	x = abs(x);
	      }
	       
	      int num = 0;
	      while (x > 0) {
	          if ((num != 0 && (INT_MAX / abs(num) < 10)) || ((unsigned int)abs(num * 10) + (unsigned int)(x % 10) > INT_MAX))
	              return 0;
	          num = num*10 + sign*(x%10);
	          x = x/10;
	      }
	      
	      return num;
	    }
	};