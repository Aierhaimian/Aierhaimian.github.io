---
layout:     post
title:      202. Happy Number
subtitle:   LeetCode刷题笔记
date:       2018-03-23
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-202. Happy Number 题目

# 题目 #

Write an algorithm to determine if a number is "happy".

A happy number is a number defined by the following process: Starting with any positive integer, replace the number by the sum of the squares of its digits, and repeat the process until the number equals 1 (where it will stay), or it loops endlessly in a cycle which does not include 1. Those numbers for which this process ends in 1 are happy numbers.

**Example:** 19 is a happy number

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

$$1^2 + 9^2 = 82$$

$$8^2 + 2^2 = 68$$

$$6^2 + 8^2 = 100$$

$$1^2 + 0^2 + 0^2 = 1$$

# 解题思路 #

(1). 算法思路：

首先，将n作为键对对应的值加1，然后看运算后的结果是否为1，如果为1则返回true；否则该结果作为键去查找unorder_map中是否有对应的键，如果有，则返回false，如果没有用最新结果继续计算。

- 时间复杂度：O(n)
- 空间复杂度：O(n)

(2). 算法实现：

	class Solution {
	public:
	    bool isHappy(int n) {
	        unordered_map<int, int> record;
	        ++ record[n];
	        int tmp = n;
	        while (tmp != 1) {
	            int tmp2 = 0;
	            while (tmp > 0) {
	                tmp2 += pow(tmp%10, 2);
	                tmp /= 10;
	            }
	            if(record[tmp2] > 0)
	                return false;
	            else
	                ++ record[tmp2];
	            tmp = tmp2;
	        }
	        return true;
	    }
	};

