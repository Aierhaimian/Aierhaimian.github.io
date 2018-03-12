---
layout:     post
title:      125. Valid Palindrome
subtitle:   LeetCode刷题笔记
date:       2018-03-10
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-125. Valid Palindrome 题目

# 题目 #

Given a string, determine if it is a palindrome, considering only alphanumeric characters and ignoring cases.

For example,

	"A man, a plan, a canal: Panama" is a palindrome.
	"race a car" is not a palindrome.

**Note:**

Have you consider that the string might be empty? This is a good question to ask during an interview.

For the purpose of this problem, we define empty string as valid palindrome.

# 解题思路 #

(1). 算法描述：

题目中已说明只判断字母和数字字符，且不区分大小写。空字符串属于回文。

首先，针对输入空字符串的特殊情况进行单独处理，然后使用双指针i,j，分别指向字符串首尾，在进行判断前先将字符串中所有的大写字母全部转换成小写；

然后进行判断：

	- 如果指针指向的字符非字母或数字，并且i<j时对i,j分别加一、减一，否则break；
	- 如果两个指针指向字符不同则返回false，否则对i,j分别加一、减一。

- 时间复杂度：O(n)
- 空间复杂度：O(1)

(2). 算法实现：

	class Solution {
	public:
	    bool isPalindrome(string s) {
		if (s == "" || s.size() == 0 || s.size() == 1)
		    return true;
		
		int i = 0;
		int j = s.size() - 1;

		for (int k = i; k <= j; ++k) {
		    if (s[k] >= 'A' && s[k] <= 'Z')
		        s[k] = tolower(s[k]);
		}
		
		while (i < j) {
		    while ((s[i] <'0' || s[i] > '9') && (s[i] <'a' || s[i] > 'z')) {
		        if (i >= j)
		            break;
		        ++i;
		    }
		    while ((s[j] <'0' || s[j] > '9') && (s[j] < 'a'|| s[j] > 'z')) {
		        if (i >= j)
		            break;
		        --j;
		    }
		    
		    if (s[i] != s[j])
		        return false;
		    
		    ++i;
		    --j;
		}
		return true;
	    }
	};

