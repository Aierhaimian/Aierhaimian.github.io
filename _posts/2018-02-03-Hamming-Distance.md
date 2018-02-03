---
layout:     post
title:      461. Hamming Distance
subtitle:   LeetCode刷题笔记
date:       2018-02-03
author:     Earl Du
header-img: img/post-bg-2.jpg
catalog: true
tags:
    - LeetCode
    - Algorithm
---

>详解LeetCode-461. Hamming Distance 题目

# 题目

The Hamming distance between two integers is the number of positions at which the corresponding bits are different.

Given two integers x and y, calculate the Hamming distance.

**Note:**

<a href="http://www.codecogs.com/eqnedit.php?latex=0&space;\leq&space;x" target="_blank"><img src="http://latex.codecogs.com/gif.latex?0&space;\leq&space;x" title="0 \leq x" /></a>

<a href="http://www.codecogs.com/eqnedit.php?latex=y&space;<&space;2^{31}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?y&space;<&space;2^{31}" title="y < 2^{31}" /></a>

#举例

	Input: x = 1, y = 4

	output: 2

	Explanation:
	1	(0 0 0 1)
	4	(0 1 0 0)
	       *   *
	The above arrows point to positions where the corresponding bits are different.

#问题思路1

将两个十进制数分别转换成二进制数，然后后逐位比较各二进制位，若不同其汉明距离加1。

时间复杂度：O(n)

##C语言实现
	int hammingDistance(int x, int y) {
	    char str1[32] = {0};
	    char str2[32] = {0};
	    for (int i = 0; i < 32; ++i) {
	        str1[i] = 0 + '0';
	        str2[i] = 0 + '0';
	    }
	    int i = 0, j = 0;
	    int flag = 0;
	    while(x)
	    {
	        str1[i]=x%2+'0';
	        x=x/2;
	        i++;
	    }
	    while(y)
	    {
	        str2[j]=y%2+'0';
	        y=y/2;
	        j++;
	    }
	    for(int k = 0; k < 32; k++){
	//        printf("%c, %c\n", str1[k], str2[k]);
	        if(str1[k] != str2[k]){
	            flag += 1;
	        }
	    }
	    return flag;
	}

#问题思路2

首先，将两数异或，即计算出两个数字对应二进制数各位中数字不同的位，在异或结果中为1；

然后将异或结果逐位右移，并与1，如果结果为1则汉明距离加1，最终即可计算出两数的汉明距离。

时间复杂度;O(n)

##C语言实现

	int hammingDistance(int x, int y) {
	    int xor = x^y;
	    int hamming_distance = 0;
	    for (int i = 0; i < 32; ++i) {
	        hamming_distance += (xor >> i) & 1;
	        if (xor == 0)
	            break;
	    }
	    return hamming_distance;
	}