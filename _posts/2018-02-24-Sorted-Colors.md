---
layout:     post
title:      75. Sort Colors
subtitle:   LeetCode刷题笔记
date:       2018-02-24
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-75. Sort Colors 题目

# 题目 #

Given an array with n objects colored red, white or blue, sort them so that objects of the same color are adjacent, with the colors in the order red, white and blue.

Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.

Note:
You are not suppose to use the library's sort function for this problem.

# 解题思路 #

## 思路一 ##

直接调用sort()函数，或者使用任意排序算法。

## 思路二 ##

因为数组中只有0,1,2三个元素，因此可以首先统计这三个元素出现的频率，然后再按照0,1,2的出现的频度依次给数组赋值0,1,2。

- 时间复杂度：O(n)
- 空间复杂度：O(k)

		class Solution {
		public:
		    void sortColors(vector<int>& nums) {
		        int count[3] = {0}; //存放0,1,2三个元素的频率
		//        for (int i = 0; i < nums.size(); ++i) {
		//            if (nums[i] == 0)
		//                count[0] += 1;
		//            else if (nums[i] == 1)
		//                count[1] += 1;
		//            else
		//                count[2] += 1;
		//        }

		        for (int i = 0; i < nums.size(); ++i) {
		            assert(nums[i] >= 0 && nums[i] <= 2);   //细节
		            count[nums[i]] ++;
		        }
		
		//        for (int i = 0; i < count[0]; ++i)
		//            nums[i] = 0;
		//        for (int i = count[0]; i < count[0] + count[1]; ++i)
		//            nums[i] = 1;
		//        for (int i = count[0] + count[1]; i < count[0] + count[1] + count[2]; ++i)
		//            nums[i] = 2;

		        int index = 0;
		        for (int i = 0; i < count[0]; ++i)
		            nums[index++] = 0;
		        for (int i = 0; i < count[1]; ++i)
		            nums[index++] = 1;
		        for (int i = 0; i < count[2]; ++i)
		            nums[index++] = 2;
		    }
		};

## 思路三 ##

借助三路快速排序partition的思路：

![](https://i.imgur.com/RQjf45i.jpg)

- 时间复杂度：O(n)
- 空间复杂度：O(1)

		class Solution {
		public:
		    //三路快速排序思路
		    void sortColors(vector<int>& nums) {
		        int zero = -1; //nums[0...zero] == 0
		        int two = nums.size(); //nums[two...n-1] == 2
		        for (int i = 0; i < two; ) {
		            if (nums[i] == 1)
		                i++;
		            else if (nums[i] == 2)
		                swap(nums[i], nums[--two]);
		            else {
		                assert(nums[i] == 0);
		                swap(nums[i++], nums[++zero]);
		            }
		        }
		    }
		};