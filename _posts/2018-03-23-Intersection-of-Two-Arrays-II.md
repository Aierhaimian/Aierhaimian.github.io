---
layout:     post
title:      350. Intersection of Two Arrays II
subtitle:   LeetCode刷题笔记
date:       2018-03-23
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-350. Intersection of Two Arrays II 题目

# 题目 #

Given two arrays, write a function to compute their intersection.

**Example:**

	Given nums1 = [1, 2, 2, 1], nums2 = [2, 2], return [2, 2].

**Note:**

- Each element in the result should appear as many times as it shows in both arrays.
- The result can be in any order.
- 
**Follow up:**

- What if the given array is already sorted? How would you optimize your algorithm?
- What if nums1's size is small compared to nums2's size? Which algorithm is better?
- What if elements of nums2 are stored on disk, and the memory is limited such that you cannot load all elements into the memory at once?

# 解题思路 #

1.使用C++ map容器类(底层实现：平衡的二叉树)

(1). 算法描述：

将nums1的元素作为键对对应的值加1，然后用nums2中的元素作为键去查找map中是否有对应的键，如果有，看其值是否为零，如果不为零，则保存此元素到resultVec中，并将map中该键的值减1。

- 时间复杂度：O(nlogn)
- 空间复杂度：O(n)

(2). 算法实现：

	class Solution {
	public:
	    vector<int> intersect(vector<int>& nums1, vector<int>& nums2) {
			O(nlogn)
	        map<int, int> record;
	        for (int i = 0; i < nums1.size(); ++i) {
				if (record.find(nums1[i]) == record.end())
					record.insert(make_pair(nums1[i], 1));
				else	            
					++ record[nums1[i]];
	        }
			
			//O(nlogn)
	        vector<int> resultVec;
	        for (int i = 0; i < nums2.size(); ++i) {
	            if (record.find(nums2[i]) != record.end() && record[nums2[i]] > 0) {
	                resultVec.push_back(nums2[i]);
	                -- record[nums2[i]];
					if(record[nums2[i]] == 0)
						record.erase(nums2[i]);
	            }
	        }
	        return resultVec;
	    }
	};


2.使用C++ unorder_map容器类(底层实现：哈希表)

(1). 算法描述：


将nums1的元素作为键对对应的值加1，然后用nums2中的元素作为键去查找unorder_map中是否有对应的键，如果有，看其值是否为零，如果不为零，则保存此元素到resultVec中，并将unorder_map中该键的值减1。

- 时间复杂度：O(n)
- 空间复杂度：O(n)

(2). 算法实现：

	class Solution {
	public:
	    vector<int> intersect(vector<int>& nums1, vector<int>& nums2) {
	        unordered_map<int, int> record;
	        for (int i = 0; i < nums1.size(); ++i) {
	            if (record.find(nums1[i]) == record.end())
	                record.insert(make_pair(nums1[i], 1));
	            else
	                ++ record[nums1[i]];
	        }
	        vector<int> resultVec;
	        for (int i = 0; i < nums2.size(); ++i) {
	            if (record.find(nums2[i]) != record.end() && record[nums2[i]] > 0) {
	                resultVec.push_back(nums2[i]);
	                -- record[nums2[i]];
	                if(record[nums2[i]] == 0)
	                    record.erase(nums2[i]);
	            }
	        }
	        return resultVec;
	    }
	};