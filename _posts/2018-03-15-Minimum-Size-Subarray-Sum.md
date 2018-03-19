---
layout:     post
title:      209. Minimum Size Subarray Sum
subtitle:   LeetCode刷题笔记
date:       2018-03-15
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-209. Minimum Size Subarray Sum 题目

# 题目 #

Given an array of n positive integers and a positive integer s, find the minimal length of a contiguous subarray of which the sum ≥ s. If there isn't one, return 0 instead.

For example, given the array [2,3,1,2,4,3] and s = 7,
the subarray [4,3] has the minimal length under the problem constraint.

# 解题思路 #

1.思路一：

(1). 算法描述：

暴力解法，遍历所有的连续子数组[i...j]，计算其和sum，验证sum >= s

- 时间复杂度：O(n^3)
- 空间复杂度：O(1)

(2). 算法实现：

----------


	class Solution {
	public:
	    int minSubArrayLen(int s, vector<int>& nums) {
	        int min_sub_size = nums.size() + 1;
	        for (int i = 0; i < nums.size()-1; ++i) {
	            for (int j = i+1; j < nums.size(); ++j) {
	                int sum = 0;
	                for (int k = i; k <= j; ++k) {
	                    sum += nums[k];
	                }
	                if (sum >= s)
	                    min_sub_size = min(min_sub_size, j - i + 1);
	            }
	        }
	        if (min_sub_size == nums.size() + 1)
	            return 0;
	        return min_sub_size;
	    }
	};

(3). 暴力解优化：

用空间换时间，使用数组sums[i]保存nums[0],..., nums[i-1]的和，然后在遍历过程中通过sums[j+1]-sums[i]来快速获取nums[i],...,nums[j]之间的子数组之和。

- 时间复杂度：O(n^2)
- 空间复杂度：O(n)

----------


	class Solution {
	public:
	    int minSubArrayLen(int s, vector<int>& nums) {
	        //sum[i]存放nums[0]...nums[i-1]的和
	        vector<int> sums(nums.size()+1, 0);
	        for (int i = 1; i <= nums.size(); ++i) {
	            sums[i] = sums[i-1] + nums[i-1];
	        }
	
	        int min_sub_size = nums.size() + 1;
	        for (int i = 0; i < nums.size(); ++i) {
	            for (int j = i; j < nums.size(); ++j) {
	                // 使用sums[j+1] - sums[i] 快速获得nums[i...j]的和
	                if (sums[j+1] - sums[i] >= s)
	                    min_sub_size = min(min_sub_size, j - i + 1);
	            }
	        }
	
	        if (min_sub_size == nums.size() + 1)
	            return 0;
	        return min_sub_size;
	    }
	};

2.思路二：

(1). 算法描述：

由于暴力解法存在大量重复计算，算法的时间复杂度较高。因此可以考虑滑动窗口法来解决此问题。

- 时间复杂度：O(n)
- 空间复杂度：O(1)

(2). 算法实现：


----------


	class Solution {
	public:
	    int minSubArrayLen(int s, vector<int>& nums) {
	        int l = 0, r = -1; //nums[l...r]为滑动窗口
	        int sum = 0;
	        int min_sub_size = nums.size() + 1;
	        
	        while (l < nums.size()) {
	            if (r+1 < nums.size() && sum < s)
	                sum += nums[++r];
	            else
	                sum -= nums[l++];
	            
	            if (sum >= s)
	                min_sub_size = min(min_sub_size, r - l + 1);
	            
	        }
	        
	        if (min_sub_size == nums.size() + 1)
	            return 0;
	        return min_sub_size;
	    }
	};


