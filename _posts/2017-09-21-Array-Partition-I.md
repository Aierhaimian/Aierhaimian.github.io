---
layout:     post
title:      561. Array Partition I
subtitle:   LeetCode刷题笔记
date:       2017-09-21
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-561.Array Partition I 题目

# 题目

Given an array of 2n integers, your task is to group these integers into n pairs of integer, say (a1, b1), (a2, b2), ..., (an, bn) which makes sum of min(ai, bi) for all i from 1 to n as large as possible.

给定一个包含2n个整型元素的数组，你的任务是将这些整数分成n对整数，比如(a1,b1),(a2,b2),...,(an,bn), 这使得min(ai,bi)对于所有i从1到n的和尽可能大。

# 举例

	Input: [1,4,3,2]
	
	Output: 4
	Explanation: n is 2, and the maximum sum of pairs is 4 = min(1, 2) + min(3, 4).


# 说明

1. n is a positive integer, which is in the range of [1, 10000].
2. All the integers in the array will be in the range of [-10000, 10000].

# 问题思路

先将数组按从小到大的顺序排序，然后再将相邻的两个数组元素划分为一组，然后取每组最小值求和即可。

# 算法分析



- 假设对于每一对i，都存在bi >= ai

- 现在定义sum = min(a1,b1) + min(a2,b2) + ... + min(an,bn)，由上可知sum = a1 + a2 + ... + an

- 现定义数组合为S = a1 + b1 + a2 + b2 + ... + an + bn，可知对于每一个特定的输入，S为定值

- 因为bi >= ai，则每组数据之间的距离为故Di = bi - ai，恒等变形为bi = ai + Di

- 记SD = D1 + D2 + ... + Dn

- 综上，S = a1 + (a1 + D1) + a2 + (a2 + D2) + ... + an + (an + Dn)，恒等变形后S = 2sum + SD，推出sum = (S - SD)/2

- 由题所知，要使sum最大，因为对于特定输入S为定值，所以SD需要最小，也即Di = bi - ai最小，就是ai与bi之间的距离最小，所以本问题就是在数组中找到使Di最小的对，显然当数组有小到大排序后相邻元素间的距离是最小的，最小距离为1。

# C代码实现

	int getMin(int a, int b)
	{
	    if(a > b)
	        return b;
	    else
	        return a;
	}
	
	void quickSort(int Arr[], int left, int right)
	{
	    int i,j;
	    int tmp;
	    
	    if(left<right)
	    {
	        i = left;
	        j = right;
	        tmp = Arr[left];
	        
	        while(i != j)
	        {
	            while(i<j && Arr[j]>=tmp)
	                --j;
	            while(i<j && Arr[i]<=tmp)
	                ++i;
	            
	            if(i<j)
	            {
	                int t = Arr[i];
	                Arr[i] = Arr[j];
	                Arr[j] = t;
	            }
	        }
	        Arr[left] = Arr[i];
	        Arr[i] = tmp;
	        
	        quickSort(Arr,left,i-1);
	        quickSort(Arr,i+1,right);
	    }
	}
	
	int arrayPairSum(int* nums, int numsSize) {
	    int maxSum = 0;
	    int k = 0;
	    
	    quickSort(nums, 0, numsSize-1);
	    
	    while(k < numsSize)
	    {
	        maxSum += getMin(nums[k],nums[k+1]);
	        k += 2;
	    } 
	    return maxSum;
	}


# 结果

	81 / 81 test cases passed.
	Status: Accepted
	Runtime: 36 ms

# 我犯的错

	int getMin(int a, int b)
	{
	    if(a > b)
	        return b;
	    else
	        return a;
	}
	
	int arrayPairSum(int* nums, int numsSize) {
	    int maxSum = 0;
	    int i,j;
	    int k = 0;
	    int tmp;
	    int flag;
	    for(i=0;i<numsSize-1;i++)
	    {
	        flag = 0;
	        for(j=0;j<numsSize-i-1;j++)
	        {
	            if(nums[j] > nums[j+1])
	            {
	                tmp = nums[j];
	                nums[j] = nums[j+1];
	                nums[j+1] = tmp;
	                flag = 1;
	            }
	        }
	        if(flag == 0)
	            break;
	    }
	    
	    while(k < numsSize)
	    {
	        maxSum += getMin(nums[k],nums[k+1]);
	        k += 2;
	    } 
	    return maxSum;
	}

大家应该看的出来，上述代码中，我选择了冒泡排序算法来进行数组元素排序，然而我忽视了题目中的说明，数组元素的范围在[1,10000]之间，数组元素的大小在[-10000,10000]之间，对于这么大规模的数据，很显然采用一个平均时间复杂度在O(n^2)的算法会导致超时错误，得到的教训是，以后遇到排序问题尽量选择时间复杂度为O(nlgn)的快速排序吧。

# NOTE
初次编辑技术博客，如有错误请批评指正，我会奉上更精彩，更有内涵的好文章，谢谢大家。