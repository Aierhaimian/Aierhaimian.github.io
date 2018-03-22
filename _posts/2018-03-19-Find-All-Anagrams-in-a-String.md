---
layout:     post
title:      438. Find All Anagrams in a String
subtitle:   LeetCode刷题笔记
date:       2018-03-19
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-438. Find All Anagrams in a String 题目

# 题目 #

Given a string **s** and a **non-empty** string **p**, find all the start indices of **p**'s anagrams in **s**.

Strings consists of lowercase English letters only and the length of both strings **s** and **p** will not be larger than 20,100.

The order of output does not matter.

**Example 1:**

	Input:
	s: "cbaebabacd" p: "abc"
	
	Output:
	[0, 6]
	
	Explanation:
	The substring with start index = 0 is "cba", which is an anagram of "abc".
	The substring with start index = 6 is "bac", which is an anagram of "abc".

**Example 2:**

	Input:
	s: "abab" p: "ab"
	
	Output:
	[0, 1, 2]
	
	Explanation:
	The substring with start index = 0 is "ab", which is an anagram of "ab".
	The substring with start index = 1 is "ba", which is an anagram of "ab".
	The substring with start index = 2 is "ab", which is an anagram of "ab".

# 解题思路 #

(1). 算法描述：

采用滑动窗口法，设置窗口大小为固定的p.size()，每次左右指针都加1。在指针不越界的情况下，判断s中指针指向的窗口内的字符如果都包含在p中的字符范围内，且窗口内字符之和等于p中字符之和，则符合anagram要求，记录其位置，即为当前l所在位置，然后滑动窗口继续判断。否则直接滑动窗口继续判断。

- 时间复杂度：O(n)
- 空间复杂度：O(1)

(2). 算法实现：

----------

	class Solution {
	public:
	    vector<int> findAnagrams(string s, string p) {
	        vector<int> anagrams(20100);
	        int l = 0， r = p.size() - 1; //滑动窗口
	        int freq[256] = {0};		  //字频
	        int p_num = 0;				  //字符串p中所有字符之和
	        int k = 0;
	        for (int i = 0; i < p.size(); ++i) {
	            freq[p[i]] = 1;
	            p_num += p[i];
	        }
	        while (l < s.size() && r < s.size()) {
	            int flag = 0;
	            int tmp_num = 0;
	            for (int i = l; i <= r; ++i) {
	                tmp_num += s[i];
	                if (freq[s[i]] != 1) {
	                    flag = 1;
	                    break;
	                }
	            }
	            if (flag == 0 && tmp_num == p_num) {
	                anagrams[k++] = l;
	                l++;
	                r++;
	            } else {
	                l++;
	                r++;
	            }
	        }
	        anagrams.resize(k);
	        return anagrams;
	    }
	};