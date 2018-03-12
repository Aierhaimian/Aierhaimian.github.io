---
layout:     post
title:      344. Reverse String
subtitle:   LeetCode刷题笔记
date:       2018-03-12
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-344. Reverse String 题目

# 题目 #

Write a function that takes a string as input and returns the string reversed.

**Example:**

Given s = "hello", return "olleh".


# 解题思路 #

(1). 算法描述：

首先处理空字符串和只有一个字符的字符串这种特殊情况，然后采用双指针法逐个交换字符即可。

- 时间复杂度：O(n)
- 空间复杂度 :O(1)

(2). 算法实现：

	class Solution {
	public:
	    string reverseString(string s) {
	        int i = 0;
	        int j = s.size() - 1;
	        if (s.size() == 0 || s.size() == 1)
	            return s;
	
	        while (i < j) {
	            swap(s[i++], s[j--]);
	        }
	
	        return s;
	    }
	};