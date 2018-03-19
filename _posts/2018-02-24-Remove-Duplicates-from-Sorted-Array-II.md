---
layout:     post
title:      80. Remove Duplicates from Sorted Array II
subtitle:   LeetCode刷题笔记
date:       2018-02-24
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-80. Remove Duplicates from Sorted Array II 题目

# 题目 #

Follow up for "Remove Duplicates":
What if duplicates are allowed at most twice?

For example,
Given sorted array nums = [1,1,1,2,2,3],

Your function should return length = 5, with the first five elements of nums being 1, 1, 2, 2 and 3. It doesn't matter what you leave beyond the new length.


# 解题思路 #

同26. Remove Duplicates from Sorted Array 一样，区别在于此题中每个数组元素最多可重复2次，同样适用双指针法。此处区别在于需要记录前两个遍历到的数组元素，用于判断一个数组元素是否出现第三次。

- 时间复杂度：O(n)
- 空间复杂度：O(1)


		class Solution {
		public:
		    int removeDuplicates(vector<int>& nums) {
		        if (nums.size() <= 2)
		            return  nums.size();
		        int count = 2;
		        int curr1 = nums[0];
		        int curr2 = nums[1];
		        for (int i = 2; i < nums.size(); ++i) {
		            if (nums[i] != curr1) {
		                nums[count++] = nums[i];
		                curr1 = curr2;
		                curr2 = nums[i];
		            }
		        }
		        return count;
		    }
		};