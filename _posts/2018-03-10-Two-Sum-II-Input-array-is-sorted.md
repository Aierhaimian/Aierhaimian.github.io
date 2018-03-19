---
layout:     post
title:      167. Two Sum II - Input array is sorted
subtitle:   LeetCode刷题笔记
date:       2018-03-10
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-167. Two Sum II - Input array is sorted 题目

# 题目 #

Given an array of integers that is already **sorted in ascending order**, find two numbers such that they add up to a specific target number.

The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.

You may assume that each input would have exactly one solution and you may not use the same element twice.

Input: numbers={2, 7, 11, 15}, target=9
Output: index1=1, index2=2

# 解题思路 #

1.思路一：

(1). 算法描述：

暴力解法，直接使用双层循环，分别依次遍历前后两个元素，直至找到解为止。

- 时间复杂度：O(n^2)
- 空间复杂度：O(1)

(2). 算法实现：

	class Solution {
	public:
	    vector<int> twoSum(vector<int>& numbers, int target) {
	
	        assert(numbers.size() >= 2);
	
	        for (int i = 0; i < numbers.size(); ++i) {
	            for (int j = i+1; j < numbers.size(); ++j) {
	                if((numbers[i] + numbers[j]) == target) {
	                    int res[2] = {i+1, j+1};//返回数组下标从1开始
	                    return vector<int> (res, res+2);
	                }
	            }
	        }
	        throw invalid_argument("the input has no solution.");
	    }
	};

(3). LeetCode运行结果：

Time Limit Exceeded！

2.思路二：

(1). 算法描述：

因为暴力解法中未利用数组有序这个点，因此，我们可以考虑使用二分查找去解决这个问题，遍历数组时，每遍历一个元素，就可以从此元素后面的数据中通过二分查找的方法来查找对应的元素，如果找到则返回，否则再遍历下一个元素。

-时间复杂度：O(nlogn)
-空间复杂度：O(1)

(2). 算法实现：

	class Solution {
	public:
	    vector<int> twoSum(vector<int>& numbers, int target) {
	
	        assert(numbers.size() >= 2);
	
	        for (int i = 0; i < numbers.size()-1; ++i) {
	            int pos = binarySearch(numbers, i+1, (numbers.size() - 1), (target - numbers[i]));
	            if (pos != -1) {
	                int res[2] = {i+1, pos+1};//返回数组下标从1开始
	                return vector<int> (res, res+2);
	            }
	        }
	
	        throw invalid_argument("the input has no solution.");
	    }
	
	private:
	    int binarySearch(vector<int>& nums, int l, int r, int target) {
	        assert( l >= 0 && l < nums.size() );
	        assert( r >= 0 && r < nums.size() );
	
	        while (l <= r) {
	            int mid = l + (r - l)/2;
	            if (nums[mid] == target)
	                return mid;
	            else if (target > nums[mid])
	                l = mid + 1;
	            else
	                r = mid - 1;
	        }
	        return -1;
	    }
	};

(3). LeetCode运行结果：

![](https://i.imgur.com/oKSYFgo.jpg)

3.思路三：

(1). 算法描述：

因为数组已经升序排序，因此，使用双指针i,j分别指向数组开头和结尾的元素：

	- 如果nums[i] + nums[j] == target，则返回下标i,j；
	- 如果nums[i] + nums[j] > target，则指针j向前移一个位置；
	- 如果nums[i] + nums[j] < target，则指针i向后移一个位置。

- 时间复杂度：O(n)
- 空间复杂度：O(1)

(2). 算法实现：

	class Solution {
	public:
	    vector<int> twoSum(vector<int>& numbers, int target) {
	        assert(numbers.size() >= 2);
	
	        int i = 0;
	        int j = numbers.size()-1;
	
	        while (i < j) {
	            if (numbers[i] + numbers[j] == target) {
	                int res[2] = {i+1, j+1};//返回数组下标从1开始
	                return vector<int> (res, res+2);
	            } else if (numbers[i] + numbers[j] > target)
	                -- j;
	            else
	                ++i;
	        }
	        throw invalid_argument("the input has no solution.");
	    }
	};

(3). LeetCode运行结果：

![](https://i.imgur.com/zhHKG1g.jpg)