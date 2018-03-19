---
layout:     post
title:      728. Self Dividing Numbers
subtitle:   LeetCode刷题笔记
date:       2018-02-03
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
---

>详解LeetCode-728. Self Dividing Numbers 题目

# 题目 #

A self-dividing number is a number that is divisible by every digit it contains.

For example, 128 is a self-dividing number because 128 % 1 == 0, 128 % 2 == 0, and 128 % 8 == 0.

Also, a self-dividing number is not allowed to contain the digit zero.

Given a lower and upper number bound, output a list of every possible self dividing number, including the bounds if possible.

**Note:**

The boundaries of each input argument are:
<a href="http://www.codecogs.com/eqnedit.php?latex=1&space;\leq&space;left&space;\leq&space;right&space;\leq&space;10000" target="_blank"><img src="http://latex.codecogs.com/gif.latex?1&space;\leq&space;left&space;\leq&space;right&space;\leq&space;10000" title="1 \leq left \leq right \leq 10000" /></a>

# 举例 #

	Input: 
	left = 1, right = 22
	Output: [1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 15, 22]

# 解题思路 #

由left至right判断每个数字，依次取数字的低位至高位(若此位为零则此数字不是自分数)，将数字与其各位取余数，若余数都为零，则该数字是自分数。

## C语言实现 ##

	/**
	 * Return an array of size *returnSize.
	 * Note: The returned array must be malloced, assume caller calls free().
	 */
	int isValid(int n){
	    int num = n;
	    while(n != 0){
	        int c = n%10;
	        if(c == 0)
	            return 0;
	        else
	            if(num % c != 0)
	                return 0;
	        n /= 10;
	    }
	    return 1;
	}
	
	int* selfDividingNumbers(int left, int right, int* returnSize) {
	    int *array = (int *)malloc((right-left)*sizeof(int));
	    int *tmp = array;
	    for(int i=left; i<=right; ++i){
	        if(isValid(i) == 1){
	            *array = i;
	            array++;
	        }
	    }
	    *returnSize = array - tmp;
	    return tmp;
	}