---
layout:     post
title:      88. Merge Sorted Array
subtitle:   LeetCode刷题笔记
date:       2018-02-24
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-88. Merge Sorted Array 题目

# 题目 #

Given two sorted integer arrays nums1 and nums2, merge nums2 into nums1 as one sorted array.

**Note:**

You may assume that nums1 has enough space (size that is greater or equal to m + n) to hold additional elements from nums2. The number of elements initialized in nums1 and nums2 are m and n respectively.

# 解题思路 #

由题干已知：

- 给定两个数组nums1和nums2中的元素都是已经排好序的；
- 数组nums1中的位数是大于等于m+n的。

因此，我们可以从两个数组的最后一个元素开始比较，将较大的那一个元素放到nums1的m+n-1位置，那个较小的元素与另外一个数组中的倒数第二个元素比较，把较大的元素放到nums1的m+n-2的位置上。按照以上步骤依次往前比较，最终我们就可以得到两个给定数组的合并nums1，并且数组中各元素是有序的。算法实现如下：

- 时间复杂度：O(n)
- 空间复杂度：O(1)

	class Solution {
	public:
	    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
	        int arr_end = m + n -1;
	        int arr1 = m - 1,
	            arr2 = n - 1;
	
	        while (arr_end >= 0) {
	            if (arr1 < 0)
	                nums1[arr_end--] = nums2[arr2--];
	            else if (arr2 < 0)
	                nums1[arr_end--] = nums1[arr1--];
	            else
	                nums1[arr_end--] = (nums1[arr1] > nums2[arr2]) ? nums1[arr1--] : nums2[arr2--];
	        }
	    }
	};