---
layout:     post
title:      Calculating the greatest common divisor GCD
subtitle:   《计算机算法设计与分析》
date:       2017-12-01
author:     Earl Du
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Algorithm
---

> 介绍《计算机算法设计与分析》课程中的相关算法。

# Definition

**The greatest common divisor of two integers a and b, when at least one of them is not zero, is the largest positive integer that divides the numbers without a remainder.**

Example
	
- The divisors of 54 are: 1,2,3,6,9,18,27,54
- The divisors of 24 are: 1,2,3,4,6,8,12,24
- Thus,gcd(54,24) = 6

# Problem description

	INPUT: two n-bits numbers a. and b (a >= b)
	OUTPUT: gcd(a,b)


# How to do

- Observation: the problem size can be measured by using n; 

- Let’s start from the "smallest" instance: gcd(1, 0) = 1;

- But how to efficiently solve a "larger" instance, say
gcd(1949, 101)?

- Observation: a large problem can reduce to a smaller
subproblem:

- gcd(1949, 101) = gcd(101, 1949 mod 101) = gcd(101, 30)

- gcd(101, 30) = gcd(30, 101 mod 30) = gcd(30, 11)

- gcd(30, 11) = gcd(11, 30 mod 11) = gcd(11, 8)

- gcd(30, 11) = gcd(11, 8) = gcd(8, 3) = gcd(3, 2) =
gcd(2, 1) = gcd(1, 0) = 1

# Sub-instance relationship graph

![](https://i.imgur.com/9xyODfK.jpg)

- Node: subproblems

- Edge: reduction relationship

# Euclid algorithm

	1: function Euclid(a,b)
	2: if b = 0 then
	3: 		return a;
	4: end if
	5: return Eulid(b,a mod b);

# Time complexity analysis

**Theorem: Suppose a is a n-bit integer. Euclid(a, b) ends in O(n^3) time.**

proof:

- There are at most 2n recursive calling
	- Note that a mod b < a/2
	- After two rounds of recursive calling, both a and b shrink at least a half size.

- At each recursive calling, the mod operation costs O(n^2)
time.

# The Python language implementation

    #!usr/bin/env python
	# -*- coding: utf-8 -*-
	
	'The GCD Algorithm'
	
	__author__ = 'Earl Du'
	
	def Euclid(a,b):
    	if b == 0:
        	return a
    	return Euclid(b, a%b)
	
	if __name__=="__main__":
	    a = input('Please input a:')
	    b = input('Please input b:')
	    a = int(a)
	    b = int(b)
	    gcd = Euclid(a, b)
	    print("The GCD of the number %d and %d is %d :" % (a, b, gcd))