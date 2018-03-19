---
layout:     post
title:      345. Reverse Vowels of a String
subtitle:   LeetCode刷题笔记
date:       2018-03-12
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-345. Reverse Vowels of a String 题目

# 题目 #

Write a function that takes a string as input and reverse only the vowels of a string.

**Example 1:**

Given s = "hello", return "holle".

**Example 2:**

Given s = "leetcode", return "leotcede".

**Note:**

The vowels does not include the letter "y".

# 解题思路 #

(1). 算法描述：

注意字符串区分大小写，采用双指针法，i,j分别指向字符串首尾，指针相向而行，遇到非元音字符跳过，遇到元音字符互换。

- 时间复杂度：O(n)
- 空间复杂度 :O(1)



(2). 算法实现：

	class Solution {
	public:
	    string reverseVowels(string s) {
	        int i = 0;
	        int j = s.size() - 1;
	
	        if (s.size() == 0 || s.size() == 1)
	            return s;
	
	        while (i < j) {
	            while (s[i] != 'a' && s[i] != 'e' && s[i] != 'i' && s[i] != 'o' && s[i] != 'u' && s[i] != 'A' && s[i] != 'E' && s[i] != 'I' && s[i] != 'O' && s[i] != 'U') {
	                if (i >= j)
	                    break;
	                i++;
	            }
	            while (s[j] != 'a' && s[j] != 'e' && s[j] != 'i' && s[j] != 'o' && s[j] != 'u' && s[j] != 'A' && s[j] != 'E' && s[j] != 'I' && s[j] != 'O' && s[j] != 'U') {
	                if (i >= j)
	                    break;
	                j--;
	            }
	
	            swap(s[i++], s[j--]);
	        }
	        return s;
	    }
	};