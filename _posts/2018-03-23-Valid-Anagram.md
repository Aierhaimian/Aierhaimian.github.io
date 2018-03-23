---
layout:     post
title:      242. Valid Anagram
subtitle:   LeetCode刷题笔记
date:       2018-03-23
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-242. Valid Anagram 题目

# 题目 #

Given two strings s and t, write a function to determine if t is an anagram of s.

**For example,**

	s = "anagram", t = "nagaram", return true.
	s = "rat", t = "car", return false.

**Note:**

You may assume the string contains only lowercase alphabets.

**Follow up:**

What if the inputs contain unicode characters? How would you adapt your solution to such case?

# 解题思路 #

(1). 算法描述：

首先，如果两个字符串元素个数不相等，则返回false；否则，将s的元素作为键对对应的值加1，然后用t中的元素作为键去查找unorder_map中是否有对应的键，如果有，将unorder_map中该键的值减1，如果没有则返回false。

- 时间复杂度：O(n)
- 空间复杂度：O(n)

(2). 算法实现：

	class Solution {
	public:
	    bool isAnagram(string s, string t) {
	        if (s.length() != t.length())
	            return false;
	        unordered_map<int, int> record;
	        for (int i = 0; i < s.length(); ++i) {
	            ++ record[s[i]];
	        }
	
	        for (int i = 0; i < t.length(); ++i) {
	            if (record[t[i]] > 0) {
	                -- record[t[i]];
	            } else {
	                return false;
	            }
	        }
	        return true;
	    }
	};