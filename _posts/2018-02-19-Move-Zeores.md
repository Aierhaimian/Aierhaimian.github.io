---
layout:     post
title:      283. Move Zeroes
subtitle:   LeetCode刷题笔记
date:       2018-02-20
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-283. Move Zeroes 题目

# 题目 #

Given an array nums, write a function to move all 0's to the end of it while maintaining the relative order of the non-zero elements.

For example, given **nums = [0, 1, 0, 3, 12]**, after calling your function, nums should be **[1, 3, 12, 0, 0]**.

## Note: ##

- You must do this **in-place** without making a copy of the array.
- Minimize the total number of operations.

## Credits: ##

Special thanks to @jianchao.li.fighter for adding this problem and creating all test cases.

# 解题思路 #

##思路一##

将原数组中的非零元素按原始顺序移至临时数组中，然后再将临时数组中的元素逐个移至原数组中，原数组剩余位置赋0即可。

- 时间复杂度：O(n)
- 空间复杂度：O(n)

		class Solution {
		public:
		    void moveZeroes(vector<int>& nums) {
		        vector<int> nonZeroElements;
		        for (int i = 0; i < nums.size(); ++i) {
		            if (nums[i])
		                nonZeroElements.push_back(nums[i]);
		        }
		
		        for (int i = 0; i < nonZeroElements.size(); ++i) {
		            nums[i] = nonZeroElements[i];
		        }
		
		        for (int i = nonZeroElements.size(); i < nums.size(); ++i) {
		            nums[i] = 0;
		        }
		    }
		};

##思路二##

使用哨兵k，初始为零。 从头遍历数组到i时将[0...i]中所有的非零元素都按顺序移动到[0...k)中，直至遍历完所有数组元素后，数组[0...k)中为该数组所有非零元素，再将数组[k...nums.size())中的元素赋零。

- 时间复杂度：O(n)
- 空间复杂度：O(1)

		class Solution {
		public:
		    void moveZeroes(vector<int>& nums) {
		        int k = 0; //在nums中，[0...k)的元素均为非0元素
		        // 遍历到第i个元素后,保证[0...i]中所有非0元素
		        // 都按照顺序排列在[0...k)中
		        for (int i = 0; i < nums.size(); ++i) {
		            if (nums[i]) {
		                nums[k++] = nums[i];
		            }
		        }
		        // 将nums剩余的位置放置为0
		        for (int i = k; i < nums.size(); ++i) {
		            nums[i] = 0;
		        }
		    }
		};

##思路三##

使用哨兵k，初始为零。 遍历数组，当遇到非零元素时，该元素与k所指向的元素交换位置，然后k++，直到遍历完所有数组元素时可得正确结果。

- 时间复杂度：O(n)
- 空间复杂度：O(1)

		class Solution {
		public:
		    void moveZeroes(vector<int>& nums) {
		        int k = 0;
		        for (int i = 0; i < nums.size(); ++i) {
		            if (nums[i])
		                if (i != k)
		                    swap(nums[k++], nums[i]);
		                else
		                    k ++;
		        }
		    }
		};