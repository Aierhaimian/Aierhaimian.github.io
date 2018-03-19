---
layout:     post
title:      3. Longest Substring Without Repeating Characters
subtitle:   LeetCode刷题笔记
date:       2018-03-19
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-3. Longest Substring Without Repeating Characters 题目

# 题目 #

Given a string, find the length of the longest substring without repeating characters.

**Examples:**

Given "abcabcbb", the answer is "abc", which the length is 3.

Given "bbbbb", the answer is "b", with the length of 1.

Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

# 解题思路 #

1.思路一：

(1). 算法描述：

使用暴力解法。

- 时间复杂度：O(n^3)
- 空间复杂度：O(1)

(2). 算法实现：

----------


	class Solution {
	public:
	    int lengthOfLongestSubstring(string s) {
	        if (s.empty())
	            return 0;
	        if (s.size() == 1)
	            return 1;
	
	        int max_sub_len = 0;
	        for (int i = 0; i < s.size()-1; ++i) {
	            int flag = 0;
	            for (int j = i; j < s.size(); ++j) {
	                int k = i;
	                while (s[k] != s[j]){
	                    if (k < j)
	                        k++;
	                    else {
	                        k = i;
	                        j--;
	                    }
	                }
	                if (k != j && s[k] == s[j])
	                    flag = 1;
	                if (flag != 1)
	                    max_sub_len = max(max_sub_len, j - i + 1);
	            }
	        }
	        return max_sub_len;
	    }
	};

2.思路二：

(1). 算法描述：

使用滑动窗口法。

- 时间复杂度：O(n)
- 空间复杂度：O(1)

(2). 算法实现：

----------

	class Solution {
	public:
	    int lengthOfLongestSubstring(string s) {
	        if (s.empty())
	            return 0;
	        if (s.size() == 1)
	            return 1;
	
	        int freq[256] = {0}; //字频
	        int l = 0, r = -1;   //滑动窗口s[l...r]
	        int max_sub_len = 0;
	        while (l < s.size()) {
	            if (r+1 < s.size() && freq[s[r+1]] == 0) //右边界的下个字符是否重复
	                freq[s[++r]] ++;
	            else
	                freq[s[l++]] --;
	            max_sub_len = max(max_sub_len, r - l + 1);
	        }
	        return max_sub_len;
	    }
	};