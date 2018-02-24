---
layout:     post
title:      26. Remove Duplicates from Sorted Array
subtitle:   LeetCode刷题笔记
date:       2018-02-24
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-26. Remove Duplicates from Sorted Array 题目

# 题目 #

Given a sorted array, remove the duplicates in-place such that each element appear only once and return the new length.

Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.

	Example:
	
	Given nums = [1,1,2],
	
	Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively.
	It doesn't matter what you leave beyond the new length.

# 解题思路 #

## 思路一 ##

因为数组已排序，记录数组中每一组重复数的第一个数的下标和最后一个数的下标，然后从该重复数的最后一个数复制到该重复数的第一个数，依次向后复制。

- 时间复杂度：O(n^2)
- 空间复杂度：O(1)

		class Solution {
		public:
		    int removeDuplicates(vector<int>& nums) {
		        int k = 0, p = 0;
		        if (nums.size() == 0)
		            return 0;
		        while (p < nums.size()) {
		            for (int i = p; i < nums.size() - 1; ++i) {
		                if (nums[i] == nums[i + 1])
		                    p = i + 1;
		                else
		                    break;
		            }
		            int interval = p - k;
		            if (interval != 0) {
		                int tmp = k;
		                for (int i = p; i < nums.size(); ++i)
		                    nums[tmp++] = nums[i];
		                nums.resize(nums.size() - interval);
		            }
		            k ++;
		            p = k;
		        }
		        return nums.size();
		    }
		};


## 思路二 ##

我们可以将不重复的序列存到数列前面，因为不重复序列的长度一定小于等于总序列，所以不用担心覆盖的问题。具体来说，用两个指针，快指针指向当前数组遍历到的位置，慢指针指向不重复序列下一个存放的位置，这样我们一旦遍历到一个不重复的元素，就把它复制到不重复序列的末尾。判断重复的方法是记录上一个遍历到的数字，看快慢指针指向的数组元素是否一样。

- 时间复杂度：O(n)
- 空间复杂度：O(1)

		class Solution {
		public:
		    int removeDuplicates(vector<int>& nums) {
		        if(nums.size()==0)
		            return 0;
		        int count=1;
		        int curr=nums[0];
		        for(int i=1;i<nums.size();++i)
		            if(curr != nums[i]) {
		                nums[count++] = nums[i];
		                curr = nums[i];
		            }
		        return count;
		    }
		};