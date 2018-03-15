---
layout:     post
title:      11. Container With Most Water
subtitle:   LeetCode刷题笔记
date:       2018-03-13
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-11. Container With Most Water 题目

# 题目 #

Given **n** non-negative integers **a1, a2, ..., an**, where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.

Note: You may not slant the container and n is at least 2.

# 解题思路 #

1.思路一

(1). 算法描述：

直接使用暴力解法，双重循环来解决问题。

- 时间复杂度：O(n^2)
- 空间复杂度：O(1)

(2). 算法实现：

	class Solution {
	public:
	    int maxArea(vector<int>& height) {
	        if (height.size() == 2)
	            return height[0] > height[1] ? height[1] : height[0];
	        int max_volume = 0;
	        for (int i = 0; i < height.size()-1; ++i) {
	            for (int j = i+1; j < height.size(); ++j) {
	                int volume = (height[i] > height[j] ? height[j] : height[i]) * (j-i);
	                if (max_volume < volume)
	                    max_volume = volume;
	            }
	        }
	        return max_volume;
	    }
	};

(3). LeetCode执行结果：
	
Time Limit Exceeded

2.思路二

(1). 算法描述：

采用双指针法，用i和j分别指向数组首尾，计算盛水容量，如果比当前最大容量大就将此次计算的容量作为最大容量，然后让高度较小的指针运动。

- 时间复杂度：

(2). 算法实现：

	class Solution {
	public:
	    int maxArea(vector<int>& height) {
	        if (height.size() == 2)
	            return height[0] > height[1] ? height[1] : height[0];
	        int i = 0;
	        int j = height.size() - 1;
	        int max_volume = 0;
	        while (i < j) {
	            int volume = (height[i] > height[j] ? height[j] : height[i]) * (j-i);
	            if (max_volume < volume)
	                max_volume = volume;
	            if (height[i] < height[j])
	                i++;
	            else
	                j--;
	        }
	        return max_volume;
	    }
	};

(3). LeetCode执行结果：