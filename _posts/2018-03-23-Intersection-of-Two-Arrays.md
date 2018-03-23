---
layout:     post
title:      349. Intersection of Two Arrays
subtitle:   LeetCode刷题笔记
date:       2018-03-23
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-349. Intersection of Two Arrays 题目

# 题目 #

Given two arrays, write a function to compute their intersection.

**Example:**

	Given nums1 = [1, 2, 2, 1], nums2 = [2, 2], return [2].

**Note:**

- Each element in the result must be unique.
- The result can be in any order.

# 解题思路 #

1.使用C++ set容器类(底层实现：平衡的二叉树)

(1). 算法描述：

将nums1数组元素加进set中，然后拿nums2中元素与set中元素比较，如果有则记录在另一个set中（set可保证没有重复）。

- 时间复杂度：O(nlogn)
- 空间复杂度：O(n)

(2). 算法实现：

	class Solution {
	public:
	    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
			//O(nlogn)
	        set<int> record(nums1.begin(), nums1.end());
			
			//O(nlogn)
	        set<int> reslut;
	        for (int i = 0; i < nums2.size(); ++i) {
	            if (record.find(nums2[i]) != record.end())
	                reslut.insert(nums2[i]);//每个元素只记录一次
	        }
			
			O(n)
	        return vector<int>(reslut.begin(), reslut.end());
	    }
	};

2.使用C++ unorder_set容器类(底层实现：哈希表)

(1). 算法描述：

将nums1数组元素加进unorder_set中，然后拿nums2中元素与unorder_set中元素比较，如果有则记录在另一个unorder_set中（unorder_set可保证没有重复）。

- 时间复杂度：O(n)
- 空间复杂度：O(n)

(2). 算法实现：

	class Solution {
	public:
	    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
	        unordered_set<int> record(nums1.begin(), nums1.end());
	
	        unordered_set<int> reslut;
	        for (int i = 0; i < nums2.size(); ++i) {
	            if (record.find(nums2[i]) != record.end())
	                reslut.insert(nums2[i]);//每个元素只记录一次
	        }
	
	        return vector<int>(reslut.begin(), reslut.end());
	    }
	};	