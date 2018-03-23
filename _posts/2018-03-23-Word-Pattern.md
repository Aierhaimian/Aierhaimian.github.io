---
layout:     post
title:      290. Word Pattern
subtitle:   LeetCode刷题笔记
date:       2018-03-23
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-290. Word Pattern 题目

# 题目 #

Given a pattern and a string str, find if str follows the same pattern.

Here follow means a full match, such that there is a bijection between a letter in pattern and a non-empty word in str.

**Examples:**

	1.pattern = "abba", str = "dog cat cat dog" should return true.
	2.pattern = "abba", str = "dog cat cat fish" should return false.
	3.pattern = "aaaa", str = "dog cat cat dog" should return false.
	4.pattern = "abba", str = "dog dog dog dog" should return false.

**Notes:**

You may assume pattern contains only lowercase letters, and str contains lowercase letters separated by a single space.

# 解题思路 #

(1). 算法实现：

	class Solution {
	public:
	    bool wordPattern(string pattern, string str) {
	        unordered_map<char, int> p_table;
	        unordered_map<string, int> s_table;
	
	        istringstream in(str);
	        int i = 0;
	        for (string word; in >> word; ++i) {
	            auto p_res = p_table.insert({pattern[i], p_table.size()});
	            auto s_res = s_table.insert({word, s_table.size()});
	            if (p_res.first->second != s_res.first->second) {
	                return false;
	            }
	        }
	        return i == pattern.size();
	    }
	};