---
layout:     post
title:      27. Remove Element
subtitle:   LeetCode刷题笔记
date:       2018-02-24
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-27. Remove Element 题目

# 题目 #

Given an array and a value, remove all instances of that value in-place and return the new length.

Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.

The order of elements can be changed. It doesn't matter what you leave beyond the new length.

	Example:
	
	Given nums = [3,2,2,3], val = 3,
	
	Your function should return length = 2, with the first two elements of nums being 2.

# 解题思路 #

## 思路一 ##

使用标记flag，初始为零，当nums中元素与给定val不等时，将该元素赋值到nums[flag]中，并使flag++，最后返回flag即为移除所有nums中元素值等于val的元素后nums的长度。

- 时间复杂度：O(n)
- 空间复杂度：O(1)

		class Solution {
		public:
		    int removeElement(vector<int>& nums, int val) {
		        int flag = 0;
		        for (int i = 0; i < nums.size(); ++i) {
		            if (nums[i] != val) {
		                nums[flag++] = nums[i];
		            }
		        }
		        return flag;
		    }
		};

## 思路二 ##

每次首先判断nums最后一个元素是否等于val，如果相等则将最后一个元素移除，直到最后一个元素不等于val为止。然后从头开始遍历nums中元素，如果nums[i]等于val，则将nums中最后一个元素赋值给nums[i]，因为此时nums最后一个元素必然不等于val。

- 时间复杂度：O(n)
- 空间复杂度：O(1)

		class Solution {
		public:
		    int removeElement(vector<int>& nums, int val) {
		        int sz = nums.size();
		        if (sz == 0)
		            return 0;
		        while (nums[sz-1] == val && sz > 0)
		            sz--;
		        for (int i = 0; i < sz; ++i) {
		            if (nums[i] == val) {
		                nums[i] = nums[sz-1];
		                sz--;
		                while (nums[sz-1] == val)
		                    sz--;
		            }
		        }
		        nums.resize(sz);
		        return sz;
		    }
		};