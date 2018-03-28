---
layout:     post
title:      205. Isomorphic Strings
subtitle:   LeetCode刷题笔记
date:       2018-03-28
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-205. Isomorphic Strings 题目

# 题目 #

Given two strings **s** and **t**, determine if they are isomorphic.

Two strings are isomorphic if the characters in **s** can be replaced to get **t**.

All occurrences of a character must be replaced with another character while preserving the order of characters. No two characters may map to the same character but a character may map to itself.

**For example,**

	Given "egg", "add", return true.
	
	Given "foo", "bar", return false.
	
	Given "paper", "title", return true.

**Note:**

You may assume both **s** and **t** have the same length.

# 解题思路 #

(1). 算法实现：

	class Solution {
	public:
	    bool isIsomorphic(string s, string t) {
	        if (s.empty() || t.empty())
	            return true;
	        if (s.size() != t.size())
	            return false;
	
	        unordered_map<char, int> s_table;
	        unordered_map<char, int> t_table;
	
	        int i = 0;
	        for (i = 0; i < s.size(); ++i) {
	            auto s_res = s_table.insert({s[i], s_table.size()});
	            auto t_res = t_table.insert({t[i], t_table.size()});
	            if (s_res.first->second != t_res.first->second)
	                return false;
	        }
	
	        return i == s.size();
	    }
	};